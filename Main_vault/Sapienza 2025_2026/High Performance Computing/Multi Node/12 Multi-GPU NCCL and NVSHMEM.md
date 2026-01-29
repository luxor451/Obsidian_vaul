**Tags:** #HPC #NCCL #NVSHMEM #MultiGPU #CUDA #MPI #PGAS

---

## 1. Assignment Analysis (Jacobi with Streams)

The lecture begins by analyzing the previous assignment ([[12 MultiGPU-MPI#3. Case Study: Jacobi Solver (2D Heat Equation)|Jacobi Solver]] with CUDA Streams and Priority) to illustrate how synchronization works before moving to advanced libraries.

### The Problem: Halo Exchange & Norm Calculation
In a multi-GPU Jacobi solver, we have two dependencies:
1.  **Halo Exchange:** Updating the boundary rows requires data from neighbors.
2.  **Termination Condition:** Calculating the L2 Norm (global error) requires summing errors from all GPUs.

### Implementation Strategy
We use **CUDA Streams** to overlap computation with communication.
* **Compute Stream (Low Priority):** Calculates the "Inner" part of the grid (independent of neighbors).
* **Halo Streams (High Priority):** Calculates the "Boundary" rows (Top/Bottom).
* **Atomic Updates:** All kernels atomically update a shared variable `l2_norm_d` on the GPU.

### Critical Code Analysis
~~~cpp
// 1. Reset Norm
cudaMemsetAsync(l2_norm_d, 0, sizeof(real), compute_stream);
cudaEventRecord(reset_l2norm_done, compute_stream);

// 2. Launch Inner Kernel (Bulk of work)
launch_jacobi_kernel(..., compute_stream);

// 3. Launch Boundary Kernels (High Priority)
// Wait for reset to finish
cudaStreamWaitEvent(push_top_stream, reset_l2norm_done, 0); 
launch_jacobi_kernel(..., push_top_stream);
cudaEventRecord(push_top_done, push_top_stream);

// ... same for bottom ...

// 4. Async Data Transfers (Halo Exchange)
cudaStreamWaitEvent(p2p_stream, push_top_done, 0);
cudaMemcpyPeerAsync(..., p2p_stream); 

// 5. Global Reduction (Norm)
// Wait for all compute to finish
cudaStreamWaitEvent(reset_l2norm_stream, compute_done, 0); 
cudaStreamWaitEvent(reset_l2norm_stream, push_top_done, 0);
cudaStreamWaitEvent(reset_l2norm_stream, push_bottom_done, 0);

// Copy partial norm to host
cudaMemcpyAsync(l2_norm_h, l2_norm_d, ..., reset_l2norm_stream);
cudaStreamSynchronize(reset_l2norm_stream);

// MPI AllReduce (CPU blocks here!)
MPI_Allreduce(l2_norm_h, &l2_norm, 1, ..., MPI_COMM_WORLD);
~~~

**The Bottleneck:**
`cudaStreamSynchronize` followed by `MPI_Allreduce` forces the CPU to wait. While the GPU is computing the next iteration's boundaries, the CPU is stuck doing the reduction for the previous iteration. We need a way to do **Communication on the GPU** without stalling the CPU.

---

## 2. NCCL (NVIDIA Collective Communication Library)

NCCL (pronounced "Nickel") provides inter-GPU communication primitives that are **CUDA Stream aware**.

### Key Features
* **Topology Awareness:** Automatically detects PCIe, NVLink, and InfiniBand connections to build optimal rings/trees.
* **GPU Centric:** Communication is triggered from the CPU but executed by the GPU (DMA engines).
* **Stream Semantics:** Operations are enqueued into a stream and return immediately.

### NCCL Setup
1.  **Id Generation:** Rank 0 generates a unique ID and broadcasts it via MPI.
2.  **Initialization:** Each rank initializes its communicator.

~~~cpp
ncclUniqueId id;
if (myRank == 0) ncclGetUniqueId(&id);
MPI_Bcast(&id, sizeof(id), MPI_BYTE, 0, MPI_COMM_WORLD);
ncclCommInitRank(&comm, nRanks, id, myRank);
~~~

### Sending/Receiving (P2P) in NCCL
NCCL introduced `ncclSend` and `ncclRecv` in v2.7. They behave like `MPI_Isend/Irecv` but take a **Stream** argument.

~~~cpp
// Queue the send operation into p2p_stream
ncclSend(send_ptr, count, ncclFloat, peer_rank, comm, p2p_stream);

// Queue the recv operation
ncclRecv(recv_ptr, count, ncclFloat, peer_rank, comm, p2p_stream);
~~~

### Grouping (ncclGroupStart/End)
To avoid deadlocks or suboptimal scheduling when issuing multiple P2P operations, wrap them in a group. NCCL will treat them as a single aggregate operation.

~~~cpp
ncclGroupStart();
    ncclSend(..., neighbor_up, ...);
    ncclRecv(..., neighbor_down, ...);
ncclGroupEnd(); // Operations are submitted here
~~~

### Solving the Norm Reduction Problem
Instead of `cudaMemcpy` $\to$ `MPI_Allreduce`, we use `ncclAllReduce` directly on the GPU pointer.

~~~cpp
// Enqueue AllReduce on the stream. CPU does NOT wait.
ncclAllReduce(l2_norm_d, l2_norm_d, 1, ncclDouble, ncclSum, comm, reset_l2norm_stream);

// Copy the FINAL result to host (only if we need to print or check convergence)
cudaMemcpyAsync(l2_norm_h, l2_norm_d, ..., reset_l2norm_stream);
~~~
* **Benefit:** The reduction happens on the GPU/Network. The CPU immediately continues to launch kernels for the next iteration.

---

## 3. NVSHMEM (NVIDIA Shared Memory)

While NCCL follows the MPI "Message Passing" model, NVSHMEM implements the **PGAS (Partitioned Global Address Space)** model (similar to OpenSHMEM).



### Concept
* **Symmetric Heap:** A special area of GPU memory is allocated such that every GPU can address it.
* **One-Sided Communication:** GPU A can directly write (`Put`) to or read (`Get`) from GPU B's symmetric heap.
* **Initiator:** Can be the **Host** (CPU) or the **Device** (CUDA Kernel).

### Setup and Allocation
~~~cpp
#include <nvshmem.h>
#include <nvshmemx.h>

// Initialization (Handles MPI interop automatically)
nvshmem_init(); 

// Allocation (Must use nvshmem_malloc, NOT cudaMalloc)
double *data = (double*) nvshmem_malloc(N * sizeof(double));
~~~
* `data` is a pointer in the **Symmetric Heap**.

### Host-Initiated Communication
The CPU calls functions to trigger transfers between GPUs.

~~~cpp
// Put (Write): Write local 'source' to 'dest' on PE (Processing Element) 1
nvshmem_double_put(dest, source, count, 1);

// Get (Read): Read 'source' from PE 1 to local 'dest'
nvshmem_double_get(dest, source, count, 1);
~~~
* These are blocking by default. Use `_nbi` (Non-Blocking Implicit) versions for async behavior (similar to `MPI_Isend`).

### Device-Initiated Communication (The Killer Feature)
You can call NVSHMEM functions **inside a CUDA Kernel**. This allows fine-grained, data-dependent communication without CPU involvement.

~~~cpp
__global__ void myKernel(int *data, int neighbor_pe) {
    int i = threadIdx.x;
    
    // Write directly to neighbor's memory
    nvshmem_int_p(&data[i], value, neighbor_pe);
    
    // Global barrier across all GPUs
    nvshmem_barrier_all();
}
~~~

### Example: Jacobi with NVSHMEM (Host Side)
We can replace MPI Halo Exchange with NVSHMEM Puts.

~~~cpp
// Top Halo Exchange
// Put my top boundary (a[1]) into neighbor's bottom halo (a[0])
nvshmemx_double_put_on_stream(
    a + 0,              // Dest: neighbor's ghost cell
    a + nx,             // Source: my first real row
    nx,                 // Count
    top_neighbor,       // Target PE
    stream              // CUDA Stream
);
~~~
* `nvshmemx_..._on_stream` integrates with CUDA streams, allowing overlap just like NCCL.

---

## 4. NVSHMEM Memory Consistency

Since we are doing one-sided writes, we need to ensure the receiver sees the data before it uses it.

### The Signal/Wait Mechanism
NVSHMEM provides `signal` (flag update) and `wait_until` primitives.

**Sender (Rank 0):**
~~~cpp
// 1. Write the data
nvshmem_put_mem_nbi(dest_buf, src_buf, size, 1);

// 2. Update a flag on the receiver to say "Data is ready"
// signaling '1' to address 'flag_addr' on rank 1
nvshmemx_signal_op(flag_addr, 1, NVSHMEM_SIGNAL_SET, 1, stream);
~~~

**Receiver (Rank 1):**
~~~cpp
// 1. Wait until the flag becomes '1'
nvshmemx_wait_until(flag_addr, NVSHMEM_CMP_EQ, 1, stream);

// 2. Consume data
process_data<<<..., stream>>>(dest_buf);
~~~

### Point-to-Point Synchronization
This effectively rebuilds `MPI_Send/Recv` semantics but entirely offloaded to the GPU hardware/interconnect.

---

## 5. Comparison: MPI vs NCCL vs NVSHMEM

| Feature | MPI (CUDA-Aware) | NCCL | NVSHMEM |
| :--- | :--- | :--- | :--- |
| **Model** | Message Passing (Two-sided) | Collective / P2P (Two-sided) | PGAS (One-sided) |
| **Initiator** | Host (CPU) | Host (CPU) | Host or **Device (Kernel)** |
| **Stream Aware** | No (Blocks CPU thread) | **Yes** | **Yes** |
| **Topology** | Basic | **Optimized Ring/Tree** | Optimized Mapping |
| **Best For** | Control logic, Legacy code | **Dense Collectives (AllReduce)** | **Fine-grained / Irregular** |

### Summary Recommendation
1.  Use **NCCL** for bulk transfers and global reductions (Deep Learning training, standard Halo Exchange).
2.  Use **NVSHMEM** if you have irregular communication patterns or need to trigger communication from inside a kernel.
3.  Use **MPI** for cluster management and bootstrapping.

---

## 6. Compilation

Compiling NVSHMEM requires linking against both the CUDA runtime and the NVSHMEM library (and MPI).

~~~bash
# 1. Compile Device Code (Relocatable Device Code required for NVSHMEM)
nvcc -rdc=true -ccbin g++ -I $NVSHMEM_HOME/include \
     -c jacobi_kernels.cu -o jacobi_kernels.o

# 2. Link Host Code
mpicxx -I $NVSHMEM_HOME/include \
       jacobi.cpp jacobi_kernels.o \
       -L $NVSHMEM_HOME/lib -lnvshmem -lcuda -o jacobi
~~~