**Tags:** #HPC #MPI #CUDA #MultiGPU #GPUDirect #Jacobi #UVA

---

## 1. Introduction to Hybrid MPI+CUDA

In modern HPC clusters, scaling requires combining distributed memory parallelism (across nodes) with shared memory parallelism (accelerators within a node). The standard model combines **MPI** (Message Passing Interface) for inter-node communication and **CUDA** for intra-node computation.

### The Software Stack
To enable efficient communication, the stack typically looks like this:
1.  **Application:** (e.g., Jacobi Solver).
2.  **GPUCCL / MPI:** Communication libraries.
3.  **UCC / UCX:** Unified Communication X (Transport layer).
4.  **Hardware Drivers:** Libfabric, ibverbs (InfiniBand), shared memory.

---

## 2. CUDA-Aware MPI vs. Legacy MPI

There are two main ways to exchange data between GPUs located on different MPI ranks (nodes).

### A. Legacy Approach (Standard MPI)
Standard MPI implementations expect pointers to reside in **CPU (Host)** memory. To send data from GPU A to GPU B, you must manually stage the data through the host.

**Workflow:**
1.  **Device-to-Host (D2H):** Copy buffer from GPU A to CPU A RAM (`cudaMemcpy`).
2.  **MPI Send:** Send buffer from CPU A to CPU B (`MPI_Send`).
3.  **Host-to-Device (H2D):** Copy buffer from CPU B RAM to GPU B (`cudaMemcpy`).

**Code Example:**
~~~cpp
// Rank 0: Sender
cudaMemcpy(host_send_buff, dev_send_buff, size, cudaMemcpyDeviceToHost);
MPI_Send(host_send_buff, size, MPI_CHAR, 1, tag, MPI_COMM_WORLD);

// Rank 1: Receiver
MPI_Recv(host_recv_buff, size, MPI_CHAR, 0, tag, MPI_COMM_WORLD, &stat);
cudaMemcpy(dev_recv_buff, host_recv_buff, size, cudaMemcpyHostToDevice);
~~~
* **Disadvantages:** High latency, low bandwidth, high code complexity, prevents hardware offloading.

### B. CUDA-Aware MPI
Modern MPI libraries (OpenMPI, MVAPICH2, Cray MPI) support passing **Device Pointers** directly to MPI functions.

**Workflow:**
1.  **MPI Send:** Pass the GPU pointer directly. The MPI library handles the movement.

**Code Example:**
~~~cpp
// Rank 0: Sender (dev_send_buff is in GPU memory)
MPI_Send(dev_send_buff, size, MPI_CHAR, 1, tag, MPI_COMM_WORLD);

// Rank 1: Receiver (dev_recv_buff is in GPU memory)
MPI_Recv(dev_recv_buff, size, MPI_CHAR, 0, tag, MPI_COMM_WORLD, &stat);
~~~
* **Advantages:** Cleaner code, allows the library to use optimized paths (like [[11 MultiGPU#C. GPUDirect|GPUDirect]]), avoids explicit host staging buffers.

---

## 3. Case Study: Jacobi Solver (2D Heat Equation)

The lecture demonstrates these concepts using the Jacobi iterative method to solve the Laplace equation ($\nabla^2 u = 0$) on a 2D grid. (See also [[12 Multi-GPU NCCL and NVSHMEM#1. Assignment Analysis (Jacobi with Streams)|Jacobi with Streams]]).

### The Algorithm
The value of a cell is updated iteratively as the average of its four neighbors:
$$A_{new}[i, j] = \frac{A[i-1, j] + A[i+1, j] + A[i, j-1] + A[i, j+1]}{4}$$

### Domain Decomposition & Halo Exchange
To parallelize this across multiple GPUs, we split the 2D grid into smaller blocks (one per GPU).
* **Halo Regions:** To update the cells on the edge of a GPU's local grid, we need the values from the neighboring GPU's boundary.
* **Halo Exchange:** Before every iteration, GPUs must exchange their boundary rows/columns (Ghost Cells).



### Implementation Evolution

#### Version 1: Naive MPI (Explicit Staging)
We manually copy the boundaries (Halo) to the CPU before sending.
* **Top/Bottom Boundaries:** These are contiguous in memory (Row-major order). Easy to copy.
* **Left/Right Boundaries:** These are strided (Column-major). We must execute a CUDA kernel to "pack" the column into a contiguous buffer before copying to Host.

~~~cpp
// Step 1: Pack boundaries on Device
PackKernel<<<...>>>(d_grid, d_send_buffer_left, ...);

// Step 2: Copy to Host
cudaMemcpy(h_send_left, d_send_buffer_left, ...);

// Step 3: MPI Exchange
MPI_Isend(h_send_left, ...);
MPI_Irecv(h_recv_right, ...);
MPI_Waitall(...);

// Step 4: Copy back to Device & Unpack
cudaMemcpy(d_recv_right, h_recv_right, ...);
UnpackKernel<<<...>>>(d_grid, d_recv_right, ...);
~~~

#### Version 2: CUDA-Aware MPI
We remove the host copies, but we still need to pack the non-contiguous columns because `MPI_Send` expects a contiguous buffer (by default).

~~~cpp
// Step 1: Pack boundaries on Device
PackKernel<<<...>>>(d_grid, d_send_buffer_left, ...);

// Step 2: Direct MPI Exchange
MPI_Isend(d_send_buffer_left, ...); // Pointer is on GPU
MPI_Irecv(d_recv_buffer_right, ...);
MPI_Waitall(...);

// Step 3: Unpack
UnpackKernel<<<...>>>(d_grid, d_recv_buffer_right, ...);
~~~
* **Performance:** Already significantly faster because we skip the PCIe hop to system RAM if hardware supports it.

#### Version 3: MPI Derived Datatypes
MPI allows defining custom data structures to handle strided memory (like columns) without manual packing.

**Defining a Vector Type (Column):**
~~~cpp
MPI_Datatype column_type;
// Count, Blocklength, Stride
MPI_Type_vector(height, 1, width, MPI_DOUBLE, &column_type);
MPI_Type_commit(&column_type);
~~~

**Using it:**
~~~cpp
// No PackKernel needed!
MPI_Isend(&d_grid[index], 1, column_type, neighbor, ...);
~~~
* **Note:** Performance varies. Sometimes a manually written packing kernel is faster than the MPI implementation's internal packing engine.

---

## 4. Optimization: Overlapping Communication and Computation

To hide the latency of MPI, we can split the computation into two parts:
1.  **Inner Grid:** Cells that do not depend on Halo data.
2.  **Boundary Grid:** Cells that depend on Halo data.

**Strategy:**
1.  Initiate non-blocking MPI Receive (`MPI_Irecv`).
2.  Compute **Inner Grid** (This takes time, but doesn't need network data).
3.  Wait for MPI Communication to finish (`MPI_Waitall`).
4.  Compute **Boundary Grid** (Now that data has arrived).

**Implementation:**
This requires separating the CUDA Kernel into `ComputeInner` and `ComputeBoundary`, and potentially using CUDA Streams to run kernels while the network is busy.

~~~cpp
// Start Receive
MPI_Irecv(halo_buffer, ...);

// Update internal part of the matrix (independent of halo)
JacobiKernel_Inner<<<...>>>(...);

// Send my boundaries
MPI_Isend(boundary_buffer, ...);

// Wait for neighbors
MPI_Waitall(...);

// Update boundaries
JacobiKernel_Boundary<<<...>>>(...);
~~~

---

## 5. Under the Hood: Enabling Technologies

How does MPI know that a pointer `0x4000...` is on the GPU?

### A. UVA (Unified Virtual Addressing)
Introduced in CUDA 4.0 (Fermi architecture).
* The CPU and GPU share a single virtual address space.
* **Mechanism:** MPI can call `cudaPointerGetAttributes(ptr)`.
    * If it returns `memoryType = cudaMemoryTypeDevice`, MPI knows it's a GPU pointer and calls the appropriate CUDA transfer functions.
    * If `cudaMemoryTypeHost`, it uses standard memcpy.

### B. GPUDirect Family
A set of technologies to minimize CPU involvement.

1.  **GPUDirect P2P (Peer-to-Peer):**
    * **Intra-node** optimization.
    * Allows GPU A to read/write directly to GPU B's memory over PCIe or NVLink.
    * Bypasses host memory copies.
    * *Requirement:* Same PCIe root complex (usually).

2.  **GPUDirect RDMA (Remote Direct Memory Access):**
    * **Inter-node** optimization.
    * Allows the Network Interface Card (NIC, e.g., InfiniBand) to DMA directly into GPU memory.
    * **Path:** GPU Memory $\to$ PCIe $\to$ NIC $\to$ Network.
    * **Benefit:** Completely bypasses the CPU and System RAM.
    * **Latency:** Reduces latency from ~18$\mu$s (Legacy) to ~1-2$\mu$s (GPUDirect RDMA).



---

## 6. Performance Analysis

The slides compare the performance of the Jacobi solver implementations (Scaling from 1 to 4 GPUs).

* **Version 0 (Single GPU):** Baseline. 6.02s.
* **Version 1 (Naive MPI):** 16384x16384 grid.
* **Version 2 (CUDA-Aware):** Shows significant speedup due to removal of `cudaMemcpy`.
* **Version 3 (Derived Types):** Slightly slower/comparable to Version 2 depending on the "Pack" kernel efficiency.

**Key Metric: Efficiency**
$$Efficiency = \frac{T_{1GPU}}{N \times T_{NGPUs}}$$
* The Jacobi example achieved **97.40% efficiency** on 4 GPUs using optimized CUDA-Aware MPI.

**Profiling (Nsight Systems):**
Visual profiling confirms the behavior:
* **Naive:** Large gaps between kernels where the CPU is busy copying data.
* **CUDA-Aware:** `MPI_Send` calls overlap with GPU activity or are much shorter.
* **Overlap:** In the optimized version, the `Stream 1` (Compute) bar runs in parallel with `Stream 2` (Communication).