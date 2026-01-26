![[HPC-1a-intro.pdf]]
![[HPC-1b-intro.pptx.pdf]]
# Overview
### Key Application of HPC

- **Scientific Research**: Climate modelling, astrophysics, material science.
- **Engineering**: Computational fluid dynamics, structural analysis.
- **Healthcare & Medicine**: Genomic sequencing, drug discovery.
- **Artificial Intelligence & Machine Learning**: Training large-scale models.
### Moore's Law

**Moore's law** is the observation that the number of transistors in an integrated circuit (IC) doubles about every two years

But it seems to be failing in recent years, the reason $\implies$ The Etching size is becoming too small (only a few atoms of silicium)

### Dennard Scaling Over
Voltage can't be further reduced because of transistors threshold voltage and noise $\implies$ *End of increasing clock frequency* Since 2003, Most CPU are still running around 3 GHz
### Chiplets
Rather than fabricating a monolithic [system-on-a-chip](https://en.wikipedia.org/wiki/System_on_a_chip), chiplet technology combines multiple chips, each representing a portion of the desired functionality, possibly fabricated using different processes by different vendors and perhaps including IP from multiple sources
![[2025-10-07-142011_hyprshot.png]]

### HPC : the Top500 list

- TOP500 list began in 1993
	- 65 systems used Intel’s i860 architecture
	- Remainder had specialized architectures, mainly vector based
- Most recent TOP500 list
	- 78% of systems used Intel processors
	- Another 19% used AMD processors
- 97% of the systems use x86-64 architecture 

##### They were 4 phases :
1. **Specialized architectures** : 
	- Based on monolithic supercomputers. These were extremely expensive, custom-built machines, often from companies like Cray and Control Data Corporation (CDC). These early supercomputers relied on a small number of highly powerful, specialized vector processors. Due to the limited number of processors, they typically utilized a shared memory architecture.
2. **Microprocessor revolution** : 
	- Commodity microprocessors to eventually surpass the performance of their specialized counterparts for many HPC workloads. In 1994 the  Beowulf cluster shown how to cluster multiple, inexpensive personal computers running on the Linux operating system to work in parallel as a single, powerful machine.
3. **Accelerators**
	- Now we moved to heterogeneous architectures, where traditional general-purpose CPUs are augmented by specialized accelerators. The most dominant and transformative of these accelerators are Graphics Processing Units (GPUs), which have fundamentally reshaped the architecture and capabilities of modern supercomputers
4. **The future**
	- - Come back to specialized architectures
	- Even more heterogeneous nodes: neuromorphic, quantum, optical, AI/ML
	- Composable Disaggregated Infrastructure

In the Domain of Accelerators *NVIDIA* Dominate (426 of the TOP500)
Linux is standard everywhere

##### Today HPC 
- Highly parallel
	- Distributed memory
	- MPI +[Open-MP](https://en.wikipedia.org/wiki/OpenMP) programming model
	
- Heterogeneous, Commodity processors + GPU accelerators  
- The communication between parts is very expensive compared to FP ops
- Floating point hardware at 64, 32, 16 and 8 bit level

##### November 2022: The TOP 10 Systems (53% of the Total Performance of Top500)
![[2025-10-07-143541_hyprshot.png]]

# Component of an HPC system
- **Processors** : CPU/GPU...
- **Memory** : RAM
- **Interconnects** :  High-speed networking for data transfer.
- **Storage** : Large-scale data storage solutions
- **Software** : Operating systems, schedulers, parallel programming tools.
### Today/Future HPC Systems
- - Heterogeneous computing architectures
	- Multi-core CPUs 
	- General Purpose GPU
	- In-network computation
	- Tensor Processing Units/Neural Processing Units
	- Field Programmable Gate Arrays
### Nodes
- Heterogeneous nodes
	- General Purpose nodes (e.g. CPU based)
	- High-throughput computational nodes (e.g. GPU based)
	- Control/frontend nodes
	- Storage nodes
		- Fast storage
		- Capacity storage
### Interconnects
- Heterogeneous interconnections:
	- intra-node: connects CPU to GPU/memory/Accelerator/local storage
	- intra-rack (node to node) communication 
	- inter-rack communication

**Inter-node Communication** :
- Scope: Inside a compute node, between CPUs, GPUs, memory, and accelerators.
- Technologies:
	- QPI / UPI (Intel), Infinity Fabric (AMD) → CPU–CPU interconnect.
	- NVLink, NVSwitch (NVIDIA), UALink → high-bandwidth GPU–GPU links.
	- CXL, PCIe Gen5/Gen6 → CPU ↔ GPU, CPU ↔ accelerator, CPU ↔ memory expanders.
- Characteristics: Ultra-low latency (<100 ns), very high bandwidth (hundreds of GB/s), coherence support (e.g., CXL.mem).

**Intra-rack interconnection** : 
- Scope: Connects compute nodes within the same rack.
- Technologies:
	- InfiniBand HDR/NDR (200–400 Gbps) → dominant in HPC clusters.
	- Slingshot (HPE/Cray) → used in exascale systems (e.g., Frontier).
	- Omni-Path (Intel, legacy).
	- Ethernet (100/200/400 Gbps with RoCE or iWARP) → in cost-sensitive HPC.
	- UltraEthernet (UEC) → Designed for next-gen HPC AI workload
Characteristics: Optimized for low latency (~1–2 µs), high throughput, collective operations, and RDMA support

**Inter-rack interconnection** : 
- Scope: Connects racks together into large supercomputers.
- Technologies:
	- Same as intra-rack (InfiniBand, Slingshot, high-speed Ethernet), but arranged in scalable topologies:
	- Dragonfly, Fat-Tree, Torus, HyperX.
	- Optical interconnects increasingly important for longer distances (fiber links, silicon photonics).
- Characteristics: Scalability (tens to hundreds of thousands of nodes), fault tolerance, congestion control.

# Today and Tomorrow HPC

HPC  : Mainly solving linear equations and thus Matrix Multiplication

Dense Matrix : Matrix with no '$0$' in them
# AI/ML takeoff
Research as been around for a long time
Now there is :
- Lots of available Data
- Increase in computational power

Deep learning need small matrix operations

GEMM : GEneral Matrix Multiplication
Can get by with 16 bits floating point computional

### Future HPC system will be customized 
Now :
- CPU
- GPU 
Future :
- FPGA
- ML/AI
- ASI
- Neuromorphic
- Quantum
- Neural engine 
- Tensor
- Optical

### What is performance

xflops/s : Floating point(double precision) operation (addition or multiplication) per second
Theorical peak perfomance : $$ N_{Calculator} \cdot N_{cycle/second} \cdot N_{operation/cycle} $$
For exemple : An Intel Skylake at 2.1 GHz can do 32 Flop per cycle per core and as as 24 cores thus it's theoretical peak performance is :
$$2.1\times10^{9} \cdot 32 \cdot 24 = 1.61 \  \text{Tflops/s} $$

Python is Reference for DGEMM Performance
(Program that perform a multiplication of two $4000 \times 4000$ Matrix of floats) : 

| **Implementation** | **Speedup** |
| ------------------ | ----------- |
| Python             | 1           |
| Java               | 11          |
| C                  | 47          |
| Parallelization    | 366         |
| Divide & Conquer   | 6727        |
| Vectorization      | 23224       |
| AVX intrinsics     | 62806       |
Software does not have "good enough" performance by default

# Single-Node HPC

How to improve performance of an application running on a single node

We need to : 
1. Measure actual application performance
2. Understand the hardware architecture executing the application
3. Identify bottlenecks
4. Solve Bottlenecks
5. GOTO **1. 

### Optimize Algorithm
For exemple using [counting sort](https://en.wikipedia.org/wiki/Counting_sort) which is 2 times faster than quicksort

### Optimize Memory 
Data mouvement has a big impact
