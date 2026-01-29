**Tags:** #HPC #MultiGPU #NCCL #MPI #Interconnects #NVLink #PCIe #ScaleUp #ScaleOut

---

## 1. HPC Systems Anatomy: Scaling Strategies

When building High-Performance Computing systems, there are two primary directions for scaling:

1.  **Scale-Up (Intra-node):** Increasing the performance of a single node.
    * Adding more sockets/cores.
    * Adding accelerators (GPUs).
    * Improving the connection between components inside the node (e.g., NVLink).
2.  **Scale-Out (Inter-node):** Increasing the number of nodes.
    * Connecting many nodes via a network fabric (InfiniBand, Ethernet).
    * Requires efficient distributed algorithms.

### Non-Von Neumann Architectures
Traditional CPU/GPU clusters follow the Von Neumann bottleneck (moving data between memory and compute). Several novel architectures attempt to break this.

#### A. Cerebras (Wafer Scale Engine)
Instead of cutting a silicon wafer into small individual chips, Cerebras uses the **entire wafer** as a single processor.
* **WSE-3 Specs:** 46,225 $mm^2$ silicon, 4 Trillion transistors, 900,000 AI cores.
* **Memory:** 44 GB of **on-chip SRAM**.
* **Bandwidth:** 21 PB/s memory bandwidth (PetaBytes!).
* **Concept:** Eliminates the slow off-chip memory access. Data stays on the wafer.



#### B. Tesla Dojo
Custom architecture for training self-driving AI.
* **D1 Chip:** 354 nodes, 362 TFLOPs (BF16).
* **Training Tile:** 25 D1 chips integrated into a single module.
* **System Hierarchy:** Tile $\to$ Tray $\to$ Cabinet $\to$ ExaPOD.

#### C. Google TPU (Tensor Processing Unit)
Specialized ASICs for Matrix Multiplication (Systolic Arrays).
* **TPU v4/v5p:** Uses **Optical Circuit Switching (OCS)**.
    * The topology is not fixed. They can reconfigure the 3D Torus connections dynamically to route around failed nodes or optimize for specific traffic patterns.
    * 4096 chips per pod.

---

## 2. Scale-Up Networks (Intra-Node)

How do GPUs inside a single server talk to each other?

### 1. PCIe (Peripheral Component Interconnect Express)
The standard bus for connecting peripherals.
* **Topology:** Tree structure.
    * **Root Complex:** The CPU.
    * **Switch:** Intermediate nodes.
    * **Endpoints:** GPUs/NICs.
* **Bottleneck:** Communication between two GPUs on different switches often has to go up to the Root Complex (CPU) and back down.
* **Bandwidth:** PCIe Gen5 x16 is $\approx$ 64 GB/s (Bidirectional).

### 2. NVLink (NVIDIA)
A proprietary high-speed interconnect designed specifically for GPU-to-GPU communication.
* **Generations:**
    * NVLink 1 (Pascal): 160 GB/s.
    * NVLink 4 (Hopper): **900 GB/s** (Bidirectional).
* **NVSwitch:** A physical switch chip inside the node (e.g., in DGX systems) that connects all GPUs with all-to-all full bandwidth. It enables a "Shared Memory" abstraction across GPUs.



---

## 3. Scale-Out Networks (Inter-Node)

How do we connect thousands of nodes?

### Network Topologies
1.  **Fat-Tree (Clos):**
    * Tree structure where links get "fatter" (more bandwidth) as you go up towards the root.
    * **Pros:** Non-blocking (full bisection bandwidth).
    * **Cons:** Expensive (lots of cables and switches).
2.  **Torus (2D/3D):**
    * Grid structure where nodes connect to neighbors. Wraps around at edges.
    * **Pros:** Cheap, good for nearest-neighbor communication (stencil codes).
    * **Cons:** High diameter (latency), bad for global traffic (FFT).
    * *Used by:* Google TPU (3D Torus).
3.  **Dragonfly:**
    * Hierarchical: High-radix routers grouped together.
    * **Intra-group:** All-to-all (clique).
    * **Inter-group:** Sparse connections.
    * **Pros:** Very low diameter (max 3 hops), cheap cabling.
    * **Cons:** Prone to congestion (requires adaptive routing).
    * *Used by:* Cray Slingshot (Leonardo Supercomputer).

---

## 4. Multi-GPU Programming Models

### A. The "Old" Way: MPI + CUDA
Before specialized libraries, mixing MPI (for multi-node) and CUDA (for GPU) was inefficient.

**Workflow for sending data from GPU A (Node 1) to GPU B (Node 2):**
1.  `cudaMemcpy` (GPU A $\to$ Host A RAM).
2.  `MPI_Send` (Host A RAM $\to$ Host B RAM).
3.  `cudaMemcpy` (Host B RAM $\to$ GPU B).

**Downsides:**
* 3 copies of data.
* High latency.
* Underutilizes PCIe/NVLink bandwidth.

### B. [[12 MultiGPU-MPI#B. CUDA-Aware MPI|CUDA-Aware MPI]]
MPI implementations (like OpenMPI) were updated to accept GPU pointers directly.
~~~cpp
// The pointer 'buf' is in GPU memory
MPI_Send(buf, size, MPI_CHAR, target, tag, MPI_COMM_WORLD);
~~~
* **Mechanism:** MPI queries the pointer (using **UVA** - Unified Virtual Addressing). If it is a device pointer, it handles the copy or uses **GPUDirect**.

### C. GPUDirect
Technologies to bypass the CPU.
1.  **GPUDirect P2P:** GPU-to-GPU memory access within the same node (via PCIe or NVLink).
2.  **GPUDirect RDMA:** GPU-to-NIC access. The Network Card (NIC) reads data directly from GPU memory and sends it over the network. Zero CPU involvement.

---

## 5. NCCL (NVIDIA Collective Communication Library)

NCCL (pronounced "Nickel") is the standard library for Multi-GPU communication on NVIDIA hardware. It is topology-aware (it knows if GPUs are connected via NVLink, PCIe, or QPI).

### Primitives
NCCL implements standard MPI-like collectives:
* `Broadcast`
* `Reduce`
* `AllGather`
* `ReduceScatter`
* `AllReduce` (Most important for Deep Learning training).

### Algorithms: How AllReduce works
For large vectors (Deep Learning gradients), NCCL typically uses a **Ring Algorithm**.

**The Ring AllReduce:**
Assume $N$ GPUs and a vector of size $M$.
The vector is split into $N$ chunks. The operation takes $2(N-1)$ steps.

1.  **Scatter-Reduce Phase ($N-1$ steps):**
    * GPUs are arranged in a ring.
    * Each GPU sends a chunk to its neighbor and receives a chunk.
    * It adds the received data to its local buffer.
    * *Result:* Each GPU holds a fully reduced (summed) portion of the vector (1/$N$th of the total).

2.  **All-Gather Phase ($N-1$ steps):**
    * GPUs circulate the fully reduced chunks around the ring.
    * *Result:* All GPUs have the full reduced vector.

**Performance:**
* **Bandwidth:** Optimized for throughput.
* **Latency:** Increases linearly with $N$ (bad for small messages).
* For small messages/low latency, NCCL uses **Tree Algorithms** (Double Binary Tree).



---

## 6. NCCL Coding Example

NCCL code looks very similar to MPI code but operates on CUDA streams.

### 1. Initialization
Each process (usually 1 MPI rank = 1 GPU) needs a unique ID.

~~~cpp
ncclUniqueId id;
ncclComm_t comm;
float* sendbuff, *recvbuff;
cudaStream_t s;

// Rank 0 generates the ID
if (myRank == 0) ncclGetUniqueId(&id);

// Broadcast ID to all other ranks via MPI
MPI_Bcast(&id, sizeof(id), MPI_BYTE, 0, MPI_COMM_WORLD);

// Pick the GPU for this rank
cudaSetDevice(myRank);

// Initialize NCCL communicator
// This builds the ring/tree topology map
ncclCommInitRank(&comm, nRanks, id, myRank);
~~~

### 2. The Communication (AllReduce)
Operations are asynchronous and queued into a CUDA stream.

~~~cpp
// ... Allocate memory on GPU (cudaMalloc) ...

// Perform AllReduce
// Input: sendbuff, Output: recvbuff
// Operation: Sum
ncclAllReduce(sendbuff, recvbuff, size, ncclFloat, 
              ncclSum, comm, s);

// Synchronize stream to wait for completion
cudaStreamSynchronize(s);

// ... Cleanup ...
ncclCommDestroy(comm);
~~~

---

## 7. Inter-Process Communication with NVSHMEM

While NCCL is great for Collectives, **NVSHMEM** (NVIDIA Shared Memory) provides a **PGAS** (Partitioned Global Address Space) model for GPUs.

* **Concept:** Every GPU can directly access memory on every other GPU (intra-node or inter-node) using a global pointer.
* **Operations:** `put`, `get`, `atomic_add` (One-sided communication).
* **Use Case:** Fine-grained, irregular communication patterns where `AllReduce` is too heavy.

### Software Stack Hierarchy
1.  **Application** (PyTorch, TensorFlow).
2.  **Communication Libraries:** NCCL, NVSHMEM, MPI.
3.  **Transport Layer:** UCX (Unified Communication X), Libfabric.
4.  **Hardware Driver:** IB Verbs (InfiniBand), TCP/IP.