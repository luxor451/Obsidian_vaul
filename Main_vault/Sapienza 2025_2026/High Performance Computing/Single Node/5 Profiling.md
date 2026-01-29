**Tags:** #HPC #Profiling #Optimization #PerformanceLimits #Gprof #Perf #VTune #Roofline #FlameGraphs #TopDownAnalysis

---

## 1. Potential Performance Limits: Why is my software slow?

When software does not perform as expected, it is often due to hardware resource limitations. While computational scientists historically focused on **FLOPS** (floating-point operations per second) as the primary limit, in modern architectures, FLOPS seldom limit performance on their own.

### Key Hardware Limits
1.  **FLOPS (Floating-point Operations):** The theoretical peak calculation speed. Rarely the bottleneck in modern complex codes.
2.  **OPS (Integer Operations):** Often a more frequent limit than assumed. High-dimensional arrays require complex integer index calculations ($i \times N \times M + j \times M + k$), which can saturate the integer ALUs even in scientific code.
3.  **Memory Bandwidth:** The rate at which data can be moved from memory to the CPU. If bandwidth is the limit, the CPU is starved for data. (See [[6 Top-down Microarchitecture Analysis#Drilling into Back-End Bound (Most Common in HPC)|Back-End Bound]]).
4.  **Memory Latency:** The time it takes to fetch a single piece of data. If the CPU stalls waiting for data (e.g., pointer chasing), bandwidth is irrelevant.
5.  **Instruction Queue (Front-End):** If the CPU cannot fetch and decode instructions fast enough to feed the execution units (e.g., due to cache misses in the instruction cache), the core idles. (See [[6 Top-down Microarchitecture Analysis#3. Front-End Bound (The Starvation)|Front-End Bound]]).
6.  **Networks & Disk:** I/O operations are orders of magnitude slower than CPU/RAM operations.

### The Roofline Model
![[Pasted image 20260126173615.png]]
The Roofline model visually represents these limits by plotting **Performance (GFLOPS)** against **Arithmetic Intensity (Ops/Byte)**.
* **Sloped Region:** Performance is limited by **Memory Bandwidth**.
* **Flat Region:** Performance is limited by **Peak Compute (FLOPS)**.
* **Goal:** Increase arithmetic intensity to move the kernel from the memory-bound region to the compute-bound region.

---

## 2. Profiling Concepts

Profiling is the analysis of a program's behavior using information gathered during execution.

### Goals of Profiling
1.  **Identify Hotspots:** Determine which parts of the program consume the most time or memory.
2.  **Understand Behavior:** Analyze the frequency and duration of function calls.
3.  **Hardware Analysis:** Measure cache misses, branch mispredictions, and other microarchitectural events.

### The Pareto Principle (80/20 Rule)
* **Rule:** 90% of execution time is spent in 10% of the code.
* **Strategy:** There is no point in optimizing code that rarely runs. Profiling helps locate that critical 10% (the "hotspots").

---

## 3. Profiling Methodologies

There are two primary approaches to profiling: **Instrumentation** and **Sampling**.

### A. Instrumentation (Tracing)
The compiler or a tool inserts special code ("probes") at the beginning and end of every function.
* **Mechanism:** Measures the exact time taken between entry and exit of functions.
* **Pros:**
    * Precise iteration counts.
    * Exact call graph (who called whom).
* **Cons:**
    * **High Overhead:** The probes themselves consume CPU time, slowing down the program.
    * **Skewed Results:** Short, frequently called functions appear slower than they are because the probe overhead dominates.
    * **Intrusive:** Requires recompilation.

### B. Sampling (Statistical Profiling)
The Operating System interrupts the CPU at regular intervals (e.g., based on time or hardware events) and records the current state (Instruction Pointer).
* **Mechanism:** "Where is the CPU right now?" repeated thousands of times.
* **Pros:**
    * **Low Overhead:** Typically negligible impact on performance.
    * **Non-Intrusive:** No recompilation required (works on standard binaries).
    * **System-Wide:** Can measure time spent in kernel mode or libraries.
* **Cons:**
    * Statistical approximation (not exact).
    * Small functions running between samples might be missed.

---

## 4. Profiling Tools

### 1. Gprof (GNU Profiler)
A classic **instrumentation-based** profiler.
* **Usage:**
    1.  Compile with `-pg`: `gcc -pg -O3 -o myprog myprog.c`
    2.  Run the code: `./myprog` (produces `gmon.out`).
    3.  Analyze: `gprof myprog gmon.out > analysis.txt`.
* **Output:**
    * **Flat Profile:** Shows total time spent in each function (excluding children). Best for finding hotspots.
    * **Call Graph:** Shows the structure of function calls and how time propagates up the stack.

### 2. Perf (Linux Profiling Tool)
A powerful **sampling-based** profiler built into the Linux kernel. It uses **Performance Monitoring Counters (PMCs)**.

**Key Commands:**
* **`perf stat`:** Runs a command and gathers overall statistics (Cycles, Instructions, Cache Misses).
    * *Metric:* **[[6 Top-down Microarchitecture Analysis#1. The Limitation of Traditional Profiling|IPC (Instructions Per Cycle)]]**. If IPC < 1, the CPU is likely stalling (Memory bound).
* **`perf record`:** Samples the execution and saves data to `perf.data`.
    * *Flag:* `-g` enables call-graph recording (stack traces).
    * *Flag:* `-F 99` sets sampling frequency to 99Hz (avoids lockstep with program loops).
* **`perf report`:** Analysis tool.
    * Displays overhead per function.
    * Allows **Annotation**: Shows assembly code side-by-side with source code, highlighting the exact instruction causing the bottleneck.

**Hardware Events:**
Perf can track specific hardware events via PMCs:
* `L1-dcache-load-misses`
* `branch-misses`
* `instructions`
* `cycles`

### 3. Intel VTune Profiler
A professional GUI tool for x86 processors.
* **Top-Down Analysis:** Categorizes where CPU slots are wasted.
    * **Front-End Bound:** Fetch/Decode latency.
    * **Back-End Bound:** Execution stalls (Core or Memory).
    * **Retiring:** Useful work.
    * **Bad Speculation:** Wasted work (branch misprediction).
* **Features:**
    * **Hotspots Analysis:** Similar to `perf` but with a GUI.
    * **Threading Analysis:** Visualizes concurrency and lock contention.
    * **Memory Access:** Analyzes bandwidth and latency issues.

---

## 5. Visualization: Flame Graphs

Flame Graphs are a visualization for profiled software, allowing the most frequent code-paths to be identified quickly and accurately.

* **X-Axis (Population):** The width of the bar represents the frequency of the function in the samples (CPU Time). *Note: X-axis is alphabetical, not temporal.*
* **Y-Axis (Stack Depth):** Shows the call stack, growing upwards.
* **Colors:** Random (usually warm colors), used only to differentiate frames.
* **Interpretation:**
    * **Plateaus:** Wide bars at the top indicate hotspots (CPU-intensive functions).
    * **Towers:** Tall stacks indicate deep recursion or complex frameworks.

---

## 6. Case Study: Sieve of Eratosthenes

The course includes a practical exercise on profiling and optimizing the Sieve of Eratosthenes (prime number finder).

**Optimization Steps:**
1.  **Baseline:** Profile the naive implementation.
2.  **Algorithmic:** Change the outer loop limit to $\sqrt{N}$ instead of $N$.
3.  **Compiler:** Enable optimization flags (`-O3`).
4.  **Vectorization:** The inner loop (`is_prime[j] = 0`) is a memory set operation.
    * Using **SIMD** (e.g., `memset` or AVX intrinsics) allows clearing 32 bytes (256 bits) in a single cycle, rather than 1 byte per cycle.

---

## 7. Performance Monitoring Units (PMUs)

PMUs are specialized hardware logic inside the CPU that count events.
* **Fixed Counters:** Always count core cycles, reference cycles, and instructions retired.
* **Programmable Counters:** Can be configured to count specific events (e.g., LLC Misses).
* **Sampling:** When the counter overflows, an interrupt is generated, recording the Instruction Pointer (IP).
    * *Problem:* **Skid**. The IP recorded might be a few instructions *after* the one that caused the event (due to pipeline latency).
    * *Solution:* **PEBS (Processor Event-Based Sampling)** provides precise IP recording.