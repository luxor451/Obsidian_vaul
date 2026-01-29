**Tags:** #HPC #Profiling #TMA #Microarchitecture #Optimization #Intel #Perf #VTune #Toplev

---

## 1. The Limitation of Traditional Profiling

In modern High Performance Computing, traditional profiling metrics can be deceiving. A developer might look at **IPC (Instructions Per Cycle)** or **Cache Misses** and draw incorrect conclusions.

* **The IPC Trap:** A program might have a high IPC (e.g., 2.0) but be performing useless work, such as spinning in a loop waiting for a lock. Conversely, a program with low IPC might be stalling due to unavoidable memory latency that cannot be fixed by code optimization.
* **The Cache Miss Trap:** Not all cache misses cause stalls. Modern Out-of-Order (OoO) CPUs can "hide" latency by executing other independent instructions while waiting for memory. A profiler might report 10 million cache misses, but if they are fully overlapped with computation, fixing them yields 0% speedup.

**The Solution:** The **Top-down Microarchitecture Analysis (TMA)** method.
Instead of counting events (like misses), TMA characterizes **execution slots**. It asks: *How is the CPU utilizing its pipeline width in every clock cycle?*



---

## 2. Core Concept: Pipeline Slots

To understand TMA, we must quantify the capacity of the CPU.
Modern CPUs are **[[2 CPU Microarchitecture#Superscalar Execution|Superscalar]]**, meaning they can issue multiple micro-operations ($\mu$Ops) in a single clock cycle.

* **Pipeline Width ($W$):** The maximum number of $\mu$Ops the hardware can issue per cycle (e.g., $W=4$ for Skylake, $W=5$ for Ice Lake).
* **Cycles ($C$):** The duration of execution.

The total capacity of the CPU is defined as the total number of available **Slots**:
$$\text{Total Slots} = \text{Core Cycles} \times \text{Pipeline Width}$$

TMA classifies every single slot into one of four categories based on a decision tree logic.

### The Decision Logic
For every slot available in a cycle, the hardware asks:
1.  **Was a $\mu$Op allocated?**
    * **No:** Was it because the Front-End couldn't fetch it? -> **Front-End Bound**.
    * **No:** Was it because the Back-End was full? -> **Back-End Bound**.
2.  **Yes (Allocated):** Did it eventually retire (finish successfully)?
    * **Yes:** -> **Retiring**.
    * **No:** -> **Bad Speculation**.

---

## 3. Level 1 Hierarchy: The Four Buckets

The top level of analysis sums to 100% of all pipeline slots.

### 1. Retiring (The Good)
The slot contained a $\mu$Op that successfully executed and updated the architectural state. This is the only category that represents useful work.
* **Formula:**
    $$\text{Retiring} = \frac{\text{UOPS\_RETIRED.RETIRE\_SLOTS}}{\text{SLOTS}}$$
* **Nuance:** A high retiring rate ($>40\%$) is generally good, but it does not guarantee optimal performance. If the code uses scalar instructions (`ADD`) instead of vector instructions (`VADD`), the CPU is "Retiring" efficiently, but the algorithm is slow. **Retiring code should always be checked for [[3 Vector instructions SMID|Vectorization]] efficiency.**

### 2. Bad Speculation (The Waste)
The slot contained a $\mu$Op that was processed but eventually discarded. The CPU did work, but it was on the wrong path.
* **Primary Cause:** **[[2 CPU Microarchitecture#C. Control Hazards (Branch Hazards)|Branch Misprediction]]**. The CPU guessed an `if` condition was true, executed the block, realized it was false, and flushed the pipeline.
* **Formula:**
    $$\text{Bad Speculation} = \frac{\text{UOPS\_ISSUED.ANY} - \text{UOPS\_RETIRED.RETIRE\_SLOTS} + 4 \times \text{INT\_MISC.RECOVERY\_CYCLES}}{\text{SLOTS}}$$

### 3. Front-End Bound (The Starvation)
The Back-End (Execution Units) was ready to accept work, but the Front-End (Fetch/Decode) failed to deliver instructions. The CPU is starving.
* **Primary Causes:** I-Cache misses, ITLB misses, or slow instruction decoding.
* **Formula:**
    $$\text{Front End Bound} = \frac{\text{IDQ\_UOPS\_NOT\_DELIVERED.CORE}}{\text{SLOTS}}$$

### 4. Back-End Bound (The Stall)
The Front-End had instructions ready, but the Back-End could not accept them. The CPU is clogged.
* **Primary Causes:** Waiting for Memory (DRAM/Cache) or waiting for ALUs (Divider/Multiplier).
* **Formula:**
    $$\text{Back End Bound} = 1 - (\text{Front End} + \text{Bad Spec} + \text{Retiring})$$

---

## 4. Level 2 & 3: Drilling Down

To fix a bottleneck, we must drill down from Level 1 to find the root cause.

### Drilling into Front-End Bound
* **Latency Bound:** The fetch unit is stalled.
    * *I-Cache Miss:* Code is too large or scattered.
    * *ITLB Miss:* Paging issues for instruction memory.
* **Bandwidth Bound:** The fetch unit is working, but inefficiently.
    * *MITE:* Using the legacy hardware decoder (limited to ~4 $\mu$Ops/cycle).
    * *DSB:* Using the "Decoded Stream Buffer" ($\mu$Op Cache), which is much faster. Optimization goal: fit hot loops in the DSB.

### Drilling into Back-End Bound (Most Common in HPC)
This splits into **Memory Bound** and **Core Bound**.

#### A. Memory Bound
The Execution Units are waiting for data.
* **L1 Bound:** Stalls due to L1 latency or DTLB misses.
* **L3 Bound:** Contention or latency at the Last Level Cache.
* **DRAM Bound:**
    * *Bandwidth:* Saturation of the memory bus.
    * *Latency:* Random access patterns (pointer chasing) fetching from RAM.
* **Store Bound:** The Store Buffer is full. The CPU produces writes faster than memory can accept them.

#### B. Core Bound
The Execution Units are busy performing calculations, or there is contention for specific hardware ports.
* **Divider Busy:** Long-latency operations like Division (`DIV`) or Square Root (`SQRT`) are clogging the arithmetic units.
* **Port Utilization:** Too many instructions differ for the same execution port (e.g., only Port 0 and Port 1 can handle Vector FMAs).

---

## 5. Execution: Commands and Tools

We can perform this analysis using `perf` (standard Linux) or `toplev` (Intel's specialized wrapper).

### Method A: Using `perf stat`
Modern kernels (Linux 4.14+) include built-in Top-down metrics.

**Command:**
~~~bash
# Profile the entire system (-a) or a specific command
perf stat --top-down -a -- ./my_application
~~~

**Output Example:**
~~~text
       retiring      bad speculation   frontend bound    backend bound
S0-C0  25.4%         6.2%              12.1%             56.3%
~~~
*Analysis:* The output shows **56.3% Back-End Bound**. This is the bottleneck. We must investigate further.

### Method B: Using `toplev`
`toplev` is a tool built on top of `perf` that automates the drill-down process (Level 2, Level 3, etc.). It is essential because standard `perf` often stops at Level 1.

**Setup:**
~~~bash
git clone https://github.com/andikleen/pmu-tools
cd pmu-tools
export PATH=$PATH:$(pwd)
~~~

**Command (Level 2 Analysis):**
~~~bash
# -l2 tells toplev to drill down 2 levels deep
# --core S0-C0 binds profiling to Socket 0, Core 0 to reduce noise
./toplev.py --core S0-C0 -l2 -- ./my_application
~~~

**Output Example (Drilling down into Back-End Bound):**
~~~text
BE             Backend_Bound:           56.3 %
BE/Mem         Memory_Bound:            12.1 %
BE/Core        Core_Bound:              44.2 %  <-- NEW HOTSPOT
~~~
*Analysis:* We previously saw Back-End Bound. `toplev` reveals that inside Back-End, we are **Core Bound** (44.2%), not Memory Bound. This suggests an issue with ALUs or Ports, not RAM.

**Command (Level 3 Analysis):**
~~~bash
./toplev.py --core S0-C0 -l3 -- ./my_application
~~~

**Output Example (Drilling down into Core Bound):**
~~~text
BE/Core        Divider:                 40.5 %  <-- ROOT CAUSE
BE/Core        Ports_Utilization:        3.7 %
~~~
*Analysis:* The root cause is the **Divider** unit. The code is doing too many divisions or square roots.

---

## 6. The Code Behind the Metrics

To truly understand TMA, we must look at the C/C++ code that generates these specific bottlenecks.

### Scenario 1: Core Bound (Divider)
If TMA reports **Back-End Bound -> Core Bound -> Divider**, the code likely looks like this:

~~~cpp
// Causes "Divider" bottleneck
// Division is very slow (20-80 cycles) and typically un-pipelined.
// It clogs the execution port, preventing other instructions from issuing.
void bottleneck_divider(int n, int* input, int* output) {
    for (int i = 0; i < n; i++) {
        // High latency integer division
        output[i] = input[i] / 71; 
        
        // Or modulo operations
        // output[i] = input[i] % 13; 
    }
}
~~~
**Optimization:** Replace division with multiplication by reciprocal (for floats) or bitwise shifts (for powers of 2).

### Scenario 2: Memory Bound (Latency)
If TMA reports **Back-End Bound -> Memory Bound -> DRAM Bound -> Latency**, the code likely looks like this:

~~~cpp
// Causes "DRAM Latency" bottleneck
// Pointer chasing (Linked List)
struct Node {
    int value;
    struct Node* next;
};

void bottleneck_latency(struct Node* head) {
    struct Node* current = head;
    while (current != NULL) {
        // The CPU cannot know the address of the next node
        // until the current node is loaded from RAM.
        // It forces a full stall (100ns) for every hop.
        current = current->next; 
    }
}
~~~
**Optimization:** Convert the Linked List to an Array (vector). This restores spatial locality and allows the hardware prefetcher to predict the next address.

### Scenario 3: Bad Speculation
If TMA reports **Bad Speculation -> Branch Misprediction**, the code involves unpredictable logic:

~~~cpp
// Causes "Bad Speculation" bottleneck
// Unsorted data makes the "if" condition random (50% T, 50% F).
// The Branch Predictor fails, causing pipeline flushes.
void bottleneck_branch(int n, int* data) {
    for (int i = 0; i < n; i++) {
        if (data[i] > 128) {
            data[i] = 0;
        }
    }
}
~~~
**Optimization:** Sort the data first (making the branch predictable: TTTTT...FFFFF) or use branchless programming (e.g., CMOV instructions).

---

## 7. Intel VTune Profiler

While `perf` and `toplev` are command-line tools, **Intel VTune** visualizes this entire hierarchy graphically.

1.  **Analysis Type:** Select "Microarchitecture Exploration".
2.  **Visualization:** It displays the TMA tree.
3.  **Thresholding:** Nodes are colored **Red** if they exceed the statistical threshold for "normal" behavior (e.g., if Bad Speculation > 10%).



VTune allows you to double-click the Red node (e.g., "Divider") and jump directly to the line of source code responsible (e.g., `output[i] = input[i] / 71;`), completing the loop from high-level metric to code fix.