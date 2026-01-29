**Tags:** #HPC #Architecture #Microarchitecture #ISA #Pipeline #Cache #VirtualMemory #TLB #DRAM #HBM #PMU

---

## 1. Instruction Set Architecture (ISA)

The **Instruction Set Architecture (ISA)** serves as the fundamental interface—the "contract"—between the hardware (processor) and the software (compiler/OS). It abstracts the physical hardware complexity, providing a consistent model for programming.

### Categories of ISA
Architectures are generally categorized by how they access memory:
1.  **Register-Memory Architectures:**
    * Instructions can perform operations directly on memory addresses (e.g., `ADD R1, [Address]`).
    * **Example:** x86 (Intel/AMD).
    * **Pros:** Compact code density; fewer instructions needed.
    * **Cons:** Variable instruction lengths and complex decoding make hardware pipelining difficult.
2.  **Load-Store Architectures:**
    * Arithmetic operations *only* work on registers. Memory is accessed exclusively via specific `LOAD` and `STORE` instructions.
    * **Example:** ARM, RISC-V, MIPS.
    * **Pros:** Fixed instruction length and simpler decoding facilitate high-speed, efficient pipelining.
    * **Cons:** Requires more instructions to perform the same task compared to x86.

### Modern ISA Extensions for HPC
Standard ISAs are augmented with domain-specific extensions to accelerate heavy workloads:
* **Vector Extensions ([[3 Vector instructions SMID|SIMD]]):** Instructions that operate on wide registers (128, 256, 512 bits) to process multiple data points simultaneously.
    * *Examples:* Intel AVX-512, ARM SVE (Scalable Vector Extension), RISC-V "V".
* **Matrix Extensions:** Hardware support for matrix multiplication, crucial for AI and Linear Algebra.
    * *Examples:* Intel AMX (Advanced Matrix Extensions), ARM SME (Scalable Matrix Extension).

---

## 2. Pipelining: The Engine of Throughput

Pipelining is the technique of overlapping the execution of consecutive instructions. Instead of waiting for one instruction to finish completely before starting the next, the CPU breaks the execution process into discrete stages.

### Standard 5-Stage Pipeline (RISC Model)
1.  **IF (Instruction Fetch):** The CPU reads the instruction from the L1 Instruction Cache using the Program Counter (PC).
2.  **ID (Instruction Decode):** The instruction is decoded to determine the operation. Registers are read from the Register File.
3.  **EX (Execute):** The ALU (Arithmetic Logic Unit) calculates the result (for arithmetic ops) or the memory address (for load/stores).
4.  **MEM (Memory Access):** If the instruction is a Load/Store, memory is accessed. Otherwise, this stage is idle.
5.  **WB (Write Back):** The result is written back to the destination register.

**Theoretical Speedup:** In an ideal world, a $k$-stage pipeline improves throughput by $k$ times. However, reality is limited by **Hazards**.

### Pipeline Hazards
Hazards are situations that prevent the next instruction from executing in the following clock cycle.

#### A. Structural Hazards
Occur when two instructions attempt to use the same hardware resource simultaneously.
* *Example:* An instruction trying to write to the register file while another tries to read, if the port count is insufficient.
* *Resolution:* The pipeline must **Stall** (insert a "bubble"), delaying execution. Modern CPUs avoid this by duplicating resources (e.g., separate L1 caches for Data and Instructions to allow simultaneous fetch and memory access).

#### B. Data Hazards
Occur when an instruction depends on the result of a previous instruction that has not yet completed (Read-After-Write or RAW).
* *Example:*
    ~~~assembly
    ADD R1, R2, R3   ; R1 = R2 + R3 (Result ready at end of EX/WB)
    SUB R4, R1, R5   ; R4 = R1 - R5 (Needs R1 at start of ID/EX)
    ~~~
* *Resolution 1: Stalling:* The hardware detects the dependency and pauses the `SUB` until `ADD` finishes. This kills performance.
* *Resolution 2: Data Forwarding (Bypassing):* Special hardware wires route the result from the ALU output of the `ADD` instruction directly to the ALU input of the `SUB` instruction, skipping the WB and ID stages. This eliminates most stalls.

#### C. Control Hazards (Branch Hazards)
Occur when the pipeline encounters a branch (e.g., `if`, `loop`). The PC for the next instruction is not known until the branch condition is evaluated (deep in the pipeline), but the Fetch stage needs to grab the next instruction *now*.
* *Resolution: Branch Prediction.* The CPU guesses the outcome.
    * **Static Prediction:** Always guess "Not Taken" (or "Taken" for backward loops).
    * **Dynamic Prediction:** Uses a **Branch History Table (BHT)** or **Branch Target Buffer (BTB)** to record past outcomes.
        * *2-Bit Predictor:* A state machine that changes its prediction only after two consecutive mispredictions (Strongly Taken $\leftrightarrow$ Weakly Taken $\leftrightarrow$ Weakly Not Taken $\leftrightarrow$ Strongly Not Taken).
    * *Cost:* If prediction is wrong, the entire pipeline must be **flushed** (all speculative work discarded), wasting many cycles.

---

## 3. Advanced Parallelism Techniques

### Superscalar Execution
A scalar pipeline completes at most 1 instruction per cycle (IPC = 1). **Superscalar** processors fetch, decode, and issue multiple instructions (e.g., 4 or 8) in a single cycle.
* Requires duplicating execution units (multiple ALUs, FPUs).
* Logic checks for dependencies between all instructions in the "issue bundle".

### Out-of-Order (OoO) Execution
In-order processors stall whenever a hazard occurs. OoO processors decouple the "Front End" (Fetch/Decode) from the "Back End" (Execute).
1.  **Instruction Window:** Instructions are placed in a pool (Reservation Station).
2.  **Dynamic Scheduling:** An instruction executes as soon as its operands are ready, regardless of program order.
3.  **Reorder Buffer (ROB):** Results are collected here and "retired" (committed) in original program order to ensure correct exception handling and state updates.

### VLIW (Very Long Instruction Word)
An alternative to complex hardware scheduling (OoO). The **Compiler** decides which instructions run in parallel and packs them into one giant instruction word.
* *Pros:* Simple hardware (no dependency check logic).
* *Cons:* The compiler must know the exact latency of everything. If a cache miss occurs, the whole machine stalls. History (Intel Itanium) showed this is hard to support across different CPU generations.

---

## 4. SIMD: Data Level Parallelism

**Single Instruction Multiple Data (SIMD)** exploits the fact that in HPC, we often perform the same operation on large arrays (vectors).

* **Scalar:** `C[i] = A[i] + B[i]` (Requires 1 loop iteration per add).
* **Vector:** `C[i:i+7] = A[i:i+7] + B[i:i+7]` (Calculates 8 additions in 1 cycle).

**Hardware Implementation:**
* Requires wide registers (XMM=128-bit, YMM=256-bit, ZMM=512-bit).
* **Predication (Masking):** Handles conditional logic (`if-else`) inside a vector loop. A "mask" register disables writing for specific lanes where the condition is false, allowing the vector instruction to proceed without branching.

---

## 5. Multithreading: Thread Level Parallelism

How do we utilize pipeline slots when a thread is stalled (e.g., waiting for RAM)?

**Simultaneous Multithreading (SMT) / Hyper-Threading:**
* The physical core maintains two (or more) sets of architectural state (PC, Registers).
* The execution resources (ALUs, Caches) are shared.
* In every cycle, the hardware fetches instructions from *both* threads. If Thread A is stalled on a cache miss, Thread B can use the ALUs.
* *Performance:* Typically yields ~20-30% gain, but enables better resource utilization.

---

## 6. The Memory Hierarchy: Bridging the Gap

The **Memory Wall** is the primary bottleneck in modern computing. CPU speeds have increased exponentially, while memory latency has stagnated.
* **CPU Cycle:** ~0.3 ns
* **RAM Access:** ~100 ns
* *Implication:* A single RAM access costs ~300 CPU cycles. Without caches, the CPU would be idle 99% of the time.

### The Hierarchy Pyramid
1.  **Registers:** Instant access, compiler-managed.
2.  **L1 Cache:** Split into Instruction (L1i) and Data (L1d). Private to core. ~1ns latency.
3.  **L2 Cache:** Usually private. ~4ns latency.
4.  **L3 Cache (LLC):** Shared across all cores. ~10-20ns latency. Inclusive or Exclusive.
5.  **Main Memory (DRAM):** Off-chip. ~100ns latency.
6.  **Local Storage (SSD/NVMe):** Non-volatile. ~10-100 $\mu$s latency.

### The Principle of Locality
Caches work because computer programs are predictable.
1.  **Temporal Locality:** If you access data $X$, you will likely access $X$ again soon (loops, counters).
2.  **Spatial Locality:** If you access data $X$, you will likely access data near $X$ (arrays, matrices, instructions). *Solution:* Fetch data in "Cache Lines" (typically 64 bytes).

---

## 7. Cache Microarchitecture

### Cache Placement Strategies
When a block of memory is loaded into cache, where does it go?
1.  **Direct Mapped:**
    * Formula: `(Block Address) % (Number of Cache Blocks)`.
    * Each memory address maps to exactly *one* location.
    * *Pros:* Simple, fast lookup.
    * *Cons:* High conflict rate. If two active variables map to the same line, they thrash (constantly evict each other).
2.  **Fully Associative:**
    * A block can be placed in *any* cache line.
    * *Pros:* Zero conflict misses.
    * *Cons:* Requires comparing the address tag against *every* line simultaneously. Hardware is huge and slow.
3.  **Set Associative (N-Way):**
    * The cache is divided into $S$ sets.
    * A block maps to a specific *set*, but can go into any of the $N$ "ways" within that set.
    * *Example:* 8-Way Associative is common in L3.
    * *Trade-off:* Balances flexibility (like associative) with lookup speed (like direct mapped).

### Addressing the Cache
An address is split into three parts:
* **Tag:** Used to verify if the line holds the correct data.
* **Index:** Selects the Set.
* **Offset:** Selects the specific byte within the 64-byte Cache Line.

### Replacement Policies
When a set is full, who gets evicted?
* **LRU (Least Recently Used):** Good for temporal locality. Complex to track perfectly for high associativity.
* **Random:** Surprisingly effective and cheap to implement.
* **Pseudo-LRU:** Approximation using few bits.

### Write Policies
1.  **Write-Through:** Data is written to Cache and RAM simultaneously. Safe, but consumes massive bandwidth.
2.  **Write-Back:** Data is written only to Cache. The "Dirty Bit" tracks modification. RAM is updated only when the dirty line is evicted. Used in almost all modern CPUs.
3.  **Write Allocate vs No-Allocate:** On a write miss, do we load the block into cache? (Usually Yes: Write Allocate).

### The Three C's of Cache Misses
1.  **Compulsory (Cold):** The first time you access data, it's never in cache. Unavoidable (unless prefetching).
2.  **Capacity:** The cache isn't big enough to hold the program's working set.
3.  **Conflict:** The cache has space, but the mapping function forced multiple blocks into the same set.

---

## 8. Virtual Memory

Programs use **Virtual Addresses** (0 to $2^{64}$). Hardware uses **Physical Addresses** (RAM limits). The OS and MMU (Memory Management Unit) map between them.

### Paging
Memory is divided into fixed-size **Pages** (standard 4KB).
* **Page Table:** A data structure in RAM storing the mapping (Virtual Page Number $\to$ Physical Frame Number).
* **Page Fault:** Accessing a virtual page not currently mapped to physical RAM (OS must fetch from disk or allocate).

### TLB (Translation Lookaside Buffer)
Translating an address requires reading the Page Table (in RAM). This would double memory latency (1 read for table, 1 read for data).
* **The Solution:** The **TLB** is a specialized, ultra-fast cache that stores recent translations.
* **TLB Hit:** Translation takes ~0 cycles.
* **TLB Miss:** The hardware "Page Walker" must traverse the Page Table structure in RAM (very slow).

### Huge Pages
In HPC, accessing TBs of data with 4KB pages requires billions of mappings, overflowing the TLB.
* **Solution:** Use **Huge Pages** (2MB or 1GB).
* *Benefit:* One TLB entry covers a massive memory range, drastically reducing TLB misses and Page Walks.

---

## 9. Main Memory Technology (DRAM)

**Dynamic RAM (DRAM)** uses a capacitor to store 1 bit. It leaks charge and must be **Refreshed** every ~64ms.

### Organization
* **DIMM:** The physical stick.
* **Rank:** A set of chips on the DIMM accessed simultaneously (64-bit width).
* **Bank:** Internal logical subdivision. Banks can be pre-charged independently.
* **Row Buffer:** When a row is accessed, it's copied to the "Row Buffer". Subsequent accesses to the same row (Row Hits) are much faster than opening a new row.

### High Bandwidth Memory (HBM)
HBM is critical for modern HPC and AI GPUs.
* **3D Stacking:** Memory dies are stacked vertically using **TSV (Through-Silicon Vias)**.
* **Interposer:** Connects the memory stack directly to the GPU/CPU silicon.
* **Performance:**
    * DDR5: ~64-bit wide bus, ~100 GB/s.
    * HBM3: ~1024-bit wide bus, >800 GB/s per stack (Total >3 TB/s on high-end GPUs).

---

## 10. Performance Analysis (PMU)

How do we optimize if we can't see what the CPU is doing?

**Performance Monitoring Unit (PMU):**
Dedicated hardware logic that counts microarchitectural events without slowing down the CPU.
* **Fixed Counters:** Always count core cycles, instructions retired, and reference cycles.
* **Programmable Counters:** Can be configured to track specific events (e.g., L3 Cache Misses, Branch Mispredicts, AVX usage).

### Sampling vs Counting
* **Counting:** "How many cache misses total?" (Good for overview).
* **Sampling (Profiling):** "Interrupt the CPU every 1M cache misses and record the Instruction Pointer." (Tells you exactly *which line of code* causes the misses).
* **Hardware Assist:** Intel PEBS / AMD IBS record the exact machine state at the moment of the event to handle superscalar complexity (skid).

### Key Metrics
* **CPI (Cycles Per Instruction):** The inverse of IPC.
    * CPI < 1: Good (Superscalar utilization).
    * CPI > 1: Bad (Stalls).
    * *High CPI Analysis:* Is it memory bound (check cache misses)? Is it core bound (check functional unit usage)?