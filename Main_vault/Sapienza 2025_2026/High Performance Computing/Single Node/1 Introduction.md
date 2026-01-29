**Tags:** #HPC #Supercomputing #Parallelism #Optimization #Hardware #Leonardo

---

## 1. Logistics

**Contacts:**
* `desensi@di.uniroma1.it`
* `salvatore.pontarelli@uniroma1.it`

**Schedule:**
* Monday: 16:00 - 19:00
* Thursday: 17:00 - 19:00

**Resources:**
* **Google Classroom Code:** `zx5soohr`

**Prerequisites:**
* Good knowledge of C/C++.
* Basic understanding of embedded/multicore systems.

---

## 2. What is HPC?

High Performance Computing involves using supercomputers and parallel processing techniques to solve complex computational problems.

**Key Applications:**
* **Scientific Research:** Climate modeling, astrophysics (supernovae), material science.
* **Engineering:** Computational Fluid Dynamics (CFD), structural analysis.
* **Energy:** Nuclear fusion reactors, electric grid simulation.
* **Healthcare:** Genomic sequencing, drug discovery.
* **AI/ML:** Training large-scale models (LLMs, Digital Twins).

---

## 3. Hardware Evolution & Trends

### Historical Laws
* **Moore's Law:** Observation that the number of transistors on a chip doubles approximately every 2 years. This is now slowing down.
* **Dennard Scaling:** Observation that as transistors get smaller, their power density stays constant. This **ended around 2005**.
* **The Power Wall:** We can no longer increase CPU frequency (clock speed) because the chips would generate too much heat.

### The Shift to Multicore
Since single-core performance can no longer grow exponentially via frequency scaling, the industry shifted to **Parallelism**:
* Increasing the number of cores per chip.
* Using specialized hardware accelerators (GPUs).

---

## 4. Performance Laws

When parallelizing code, we face theoretical limits on the maximum possible speedup ($S$).

### Amdahl's Law (Strong Scaling)
Focuses on a **fixed problem size**. Speedup is limited by the sequential portion ($P_{serial}$) of the code. Even with infinite processors, you cannot exceed $1/P_{serial}$.
$$S(N) = \frac{1}{(1-P) + \frac{P}{N}}$$

### Gustafson's Law (Weak Scaling)
Focuses on **scaled problem sizes**. If the problem size grows with the number of processors, the serial part becomes less significant percentage-wise.
$$S(N) = N - (1-P)(N-1)$$

---

## 5. Performance Optimization

Modern HPC is about overcoming bottlenecks, primarily the **Memory Wall** (CPU is much faster than memory).

### The Roofline Model
A visual model to understand performance limits:
* **Compute Bound:** Performance limited by the CPU's processing power (FLOPS).
* **Memory Bound:** Performance limited by how fast data can move from RAM to CPU (Bandwidth).
* **Arithmetic Intensity:** Ratio of Floating Point Operations (FLOPS) to Bytes accessed.

### Optimization Strategies (Example: Matrix Multiplication - DGEMM)
Optimizing a standard $O(N^3)$ algorithm involves several layers of hardware exploitation:

1.  **Language Choice:** Moving from interpreted (Python) to compiled (C) reduces overhead.
2.  **Parallel Loops (Thread Parallelism):** Using OpenMP to utilize multiple cores.
3.  **Memory Optimization (Blocking/Tiling):** Dividing matrices into small blocks that fit in the CPU **Cache**. This improves data locality and avoids fetching data from slow RAM repeatedly.
4.  **Vectorization (Data Parallelism):** Using **SIMD** (Single Instruction Multiple Data) instructions (e.g., AVX) to perform operations on multiple numbers simultaneously within a single core.
5.  **Hardware Accelerators:** Offloading heavy computation to GPUs.



---

## 6. Supercomputers: The Leonardo System

The course references the **Leonardo** supercomputer (hosted by CINECA), one of the top systems in the world.

**Architecture Modules:**
1.  **Booster Module (GPU-centric):**
    * 3456 Nodes.
    * 4x NVIDIA A100 GPUs per node.
    * Focus: High throughput computation.
2.  **Data Centric Module (CPU-centric):**
    * 1536 Nodes.
    * Intel Sapphire Rapids CPUs.
3.  **LISA (Leonardo Improved Supercomputing Architecture):**
    * Expansion dedicated to Generative AI.
    * Equipped with NVIDIA H100 GPUs.

---

## 7. Course Syllabus

### Part 1: Single-Node HPC (S. Pontarelli)
Focuses on maximizing performance on a single machine.
* Modern CPU Architectures (Pipelines, Cache hierarchy).
* Vector Processors & SIMD.
* High-Performance Programming techniques.
* Application Profiling.
* Hardware Accelerators.

### Part 2: Large-Scale HPC (D. De Sensi)
Focuses on scaling applications across many machines.
* Scale-up vs Scale-out networks.
* High-Performance Network Protocols (RDMA).
* Advanced GPU Programming.
* Collective Communications (MPI).
* Use cases: Distributed Training, Linear Algebra.