**Tags:** #HPC #Vectorization #SIMD #AVX #Intrinsics #CompilerOptimization #AmdahlLaw

---

## 1. Introduction to SIMD (Single Instruction Multiple Data)

**SIMD** stands for *Single Instruction Multiple Data*. It is a parallel processing technique where a single machine instruction operates on multiple data points simultaneously.

### The Concept
In a standard **[[2 CPU Microarchitecture#Superscalar Execution|Scalar]]** operation, adding two arrays requires a loop that processes one pair of elements at a time:
$$C[i] = A[i] + B[i]$$
This takes $N$ instructions and $N$ cycles (assuming 1 cycle per op).

In a **Vector (SIMD)** operation, the CPU loads a "chunk" of data (a vector) into a wide register and processes it all at once:
$$C[i:i+k] = A[i:i+k] + B[i:i+k]$$
If the vector width is 512 bits and we are processing 64-bit doubles, we can perform **8 additions in a single cycle**.



### Hardware Implementation
* **Vector Registers:** Specialized registers that are much wider than standard 64-bit registers (e.g., 128, 256, or 512 bits).
* **Vector ALUs:** Arithmetic units designed to split the wide register into "lanes" and execute the operation on each lane independently.

---

## 2. Evolution of SIMD Architectures

Intel and other vendors have steadily increased the width of vector registers over the last two decades.

* **MMX (1997):** 64-bit registers. Integer only.
* **SSE (Streaming SIMD Extensions, 1999):** 128-bit registers (`xmm`). Introduced single-precision floats (4 floats).
* **SSE2 (2001):** Added double-precision floats (2 doubles).
* **AVX (Advanced Vector Extensions, 2011):** 256-bit registers (`ymm`). Supports 8 floats or 4 doubles.
* **AVX2 (2013):** Extended AVX to support integer vector operations and Gather/Scatter.
* **AVX-512 (2016):** 512-bit registers (`zmm`). Supports 16 floats or 8 doubles. Introduced Masking.

**Naming Convention:**
* `xmm0` ... `xmm15`: 128-bit registers.
* `ymm0` ... `ymm15`: 256-bit registers (The lower 128 bits are aliased to `xmm`).
* `zmm0` ... `zmm31`: 512-bit registers (The lower 256 bits are aliased to `ymm`).



---

## 3. How to Use SIMD?

There are three main ways to utilize vector units, ranging from easy (but less control) to hard (but maximum control).

### Level 1: Auto-Vectorization (The Compiler)
The compiler analyzes your loops and, if safe, automatically generates SIMD instructions.
* **Pros:** Zero effort, portable code.
* **Cons:** Fragile. Small changes in code can stop vectorization. The compiler is conservative (won't vectorize if there's a risk of [[9 Optimizing Computations#2. Data Dependencies & Dependency Chains|data dependency]]).
* **Flags:** `-O3`, `-march=native`, `-fopt-info-vec` (to see report).

### Level 2: Compiler Hints (Pragmas)
You give explicit instructions to the compiler to ignore potential dependencies or unroll loops.
* **OpenMP SIMD:** `#pragma omp simd`
* **IVDEP:** `#pragma ivdep` (Ignore Vector DEPendencies).
* **Vectorize:** `#pragma vector always`

### Level 3: Intrinsics (Assembly Wrapper)
You write C functions that map directly to assembly instructions.
* **Pros:** Full control, maximum performance.
* **Cons:** Verbose, hard to read, not portable (AVX code won't run on ARM).
* **Headers:** `<immintrin.h>`

---

## 4. Intel Intrinsics in Detail

Intrinsics use a specific naming convention: `_mm<width>_<op>_<suffix>`.

* **Width:**
    * (none): 128-bit (SSE)
    * `256`: 256-bit (AVX)
    * `512`: 512-bit (AVX-512)
* **Op:** The operation (e.g., `add`, `sub`, `mul`, `load`, `store`).
* **Suffix:** Data type.
    * `ps`: Packed Single (float32)
    * `pd`: Packed Double (float64)
    * `epi32`: Extended Packed Integer 32-bit.

### Example: Vector Addition (AVX2)
~~~cpp
#include <immintrin.h>

void add_vectors(float* a, float* b, float* c, int n) {
    // Process 8 floats at a time (256 bits / 32 bits = 8)
    for (int i = 0; i < n; i += 8) {
        // 1. Load data into registers
        __m256 va = _mm256_loadu_ps(&a[i]); // load unaligned
        __m256 vb = _mm256_loadu_ps(&b[i]);
        
        // 2. Perform addition
        __m256 vc = _mm256_add_ps(va, vb);
        
        // 3. Store result back to memory
        _mm256_storeu_ps(&c[i], vc);
    }
}
~~~
*Note: This simple loop assumes `n` is divisible by 8. Real code must handle the "remainder" (peeling) with a standard scalar loop.*

---

## 5. Data Alignment

Vector units are sensitive to memory alignment.
* **Aligned Load (`_mm256_load_ps`):** Requires the address to be a multiple of 32 bytes (for AVX). Very fast. Crashes if address is wrong.
* **Unaligned Load (`_mm256_loadu_ps`):** Works with any address. Slightly slower on older hardware, but mostly free on modern CPUs.

**Best Practice:** Always align your data when allocating memory.
~~~cpp
// Allocate aligned memory (32-byte alignment for AVX)
float* data = (float*) _mm_malloc(n * sizeof(float), 32);
...
_mm_free(data);
~~~

---

## 6. Challenges in Vectorization

Not all loops can be vectorized. The compiler (or you) must overcome several hurdles.

### A. Data Dependencies
If an iteration depends on the result of a previous iteration, vectorization is impossible (or requires complex shuffling).
* **Read-After-Write (RAW):** `A[i] = A[i-1] + B[i]`
    * This is a "recurrence". You cannot calculate `A[i]` until `A[i-1]` is finished.
* **Write-After-Read (WAR):** `A[i] = A[i+1]`
    * Safe to vectorize if the vector width is small enough, but generally tricky.

### B. Control Divergence (If-Else)
SIMD executes the *same* instruction on all data. What if your loop has an `if` condition?
~~~cpp
if (A[i] > 0) { B[i] = sqrt(A[i]); } 
else { B[i] = 0; }
~~~
* **Solution: Masking.** The CPU calculates *both* branches for all elements. It then uses a "mask" vector (based on `A[i] > 0`) to merge the results.
* **Performance Impact:** You pay the cost of both branches.

### C. Non-Contiguous Memory (Gather/Scatter)
Vector loads are fastest when data is contiguous in memory (`A[i], A[i+1]...`).
* **Indirect Access:** `A[B[i]]` (e.g., using an index array).
* **Gather:** Loading data from non-contiguous locations into a vector register.
* **Scatter:** Storing a vector register to non-contiguous locations.
* **Status:** Supported in hardware (AVX2/AVX-512) but significantly slower than contiguous loads.

---

## 7. Mathematical Libraries (MKL)

Writing intrinsics is hard. For standard mathematical operations (Linear Algebra, FFT), use optimized libraries like **Intel MKL (Math Kernel Library)**.
* MKL functions are hand-tuned using assembly for every specific processor generation.
* They automatically use the widest available vector units (AVX-512 on new CPUs, SSE on old ones).
* **Example:** `cblas_dgemm` for matrix multiplication.

---

## 8. Roofline Model and Vectorization

Vectorization is essential to hitting the "Compute Bound" ceiling of the Roofline Model.
* **Scalar Code:** Limited to 1 FLOP/cycle (roughly). Often Memory Bound because it's too slow to saturate bandwidth.
* **Vectorized Code:** Can reach 32+ FLOPs/cycle. Pushes the arithmetic intensity requirement up.
* **FMA (Fused Multiply-Add):** `d = a + b * c`. Modern CPUs do this in 1 cycle. Using FMA doubles the peak FLOPS (2 ops per instruction).