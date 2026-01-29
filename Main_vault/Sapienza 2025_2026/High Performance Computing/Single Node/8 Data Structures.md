**Tags:** #HPC #DataOrientedDesign #MemoryLayout #CacheOptimization #AoS #SoA #FalseSharing #Prefetching

---

## 1. Data Oriented Design (DOD)

**Data Oriented Design** is a programming paradigm that focuses on the layout of data in memory and how it is transformed, rather than organizing code around "objects" and "methods".

### DOD vs OOP (Object Oriented Programming)
**OOP Problems in HPC:**
1.  **Deep Call Stacks:** OOP encourages many small methods and deep hierarchies. Without aggressive inlining, this overhead is significant.
2.  **Poor Cache Utilization:** Objects are often allocated individually on the heap (`new Object()`). This scatters data across memory (fragmentation), causing poor **Spatial Locality**.
3.  **V-Tables:** Polymorphism requires Virtual Tables. Calling a virtual function adds a layer of indirection (pointer chasing) and causes **[[6 Top-down Microarchitecture Analysis#2. Bad Speculation (The Waste)|Branch Mispredictions]]**.

**DOD Principles:**
* **Data First:** Think about the cache line, not the individual variable.
* **Contiguous Memory:** Operate on arrays of data, not pointers to objects.
* **Batch Processing:** Process similar data in bulk to maximize bandwidth.
* **Control Allocation:** Avoid hidden/implicit memory allocations behind the scenes.

---

## 2. Multidimensional Arrays

In C/C++, there are two common ways to represent a 2D Matrix ($N \times M$).

### The "Naive" Way (Array of Pointers)
Commonly taught in basic programming, but terrible for performance.
~~~c
double **x = (double **)malloc(rows * sizeof(double *));
for (int i=0; i < rows; i++) {
    x[i] = (double *)malloc(cols * sizeof(double));
}
// Access: x[i][j]
~~~
* **Issue:** Each row is allocated separately. Row 0 might be at address `0x1000` and Row 1 at `0x8000`.
* **Result:** No spatial locality between rows. The hardware prefetcher cannot predict the next row.

### The "HPC" Way (Linearized Array)
Allocate one giant block of memory and map 2D coordinates to 1D index.
~~~c
double *x = (double *)malloc(rows * cols * sizeof(double));
// Access: x[i * cols + j]
~~~
* **Benefit:** Guaranteed contiguous memory. High cache hit rate.
* **Perf:** Significantly faster than array of pointers.

---

## 3. Matrix Multiplication Optimization

The classic $C = A \times B$ problem illustrates the importance of access patterns.

### Naive Implementation ($ijk$ order)
~~~c
for (int i=0; i<N; i++) {
    for (int j=0; j<N; j++) {
        for (int k=0; k<N; k++) {
            C[i*N + j] += A[i*N + k] * B[k*N + j];
        }
    }
}
~~~
* **Analysis:**
    * `A`: Accessed row-wise (Good, stride-1).
    * `B`: Accessed column-wise (Bad, stride-N).
    * `C`: Constant for inner loop.
* **Result:** For every step of `k`, we jump `N * sizeof(double)` bytes in B. This causes a cache miss on almost every access.



### Loop Interchange ($ikj$ order)
By simply swapping the `j` and `k` loops, we change the access pattern without changing the result.
~~~c
for (int i=0; i<N; i++) {
    for (int k=0; k<N; k++) {
        double r = A[i*N + k]; // Stored in register
        for (int j=0; j<N; j++) {
            C[i*N + j] += r * B[k*N + j];
        }
    }
}
~~~
* **Analysis:**
    * `C`: Accessed row-wise (Stride-1).
    * `B`: Accessed row-wise (Stride-1).
    * `A`: Constant for inner loop.
* **Result:** 10x-20x speedup just by swapping loops. The CPU reads contiguous memory, maximizing bandwidth and **[[3 Vector instructions SMID|vectorization potential]]**.

---

## 4. Cache Blocking (Tiling)

Even with Loop Interchange, if the matrix $N$ is huge, a single row might not fit in L1/L2 cache. By the time we finish a row and come back, the data is evicted.

**Solution:** Divide the matrix into sub-matrices (Blocks) of size $B \times B$ that fit into the cache.



### Tiled Implementation
~~~c
for (int i0 = 0; i0 < N; i0 += BlockSize) {
    for (int k0 = 0; k0 < N; k0 += BlockSize) {
        for (int j0 = 0; j0 < N; j0 += BlockSize) {
            
            // Standard matmul on the small block
            for (int i = i0; i < min(i0+BlockSize, N); i++) {
                for (int k = k0; k < min(k0+BlockSize, N); k++) {
                    for (int j = j0; j < min(j0+BlockSize, N); j++) {
                        C[i*N+j] += A[i*N+k] * B[k*N+j];
                    }
                }
            }
        }
    }
}
~~~

### Theoretical Benefit (Arithmetic Intensity)
* **Naive:** Total Memory Accesses = $2N^3$ (load) + $N^3$ (store).
    * Ratio $\approx 1$ Op per Byte. (Memory Bound).
* **Blocked:** We reuse the block in cache $N/B$ times.
    * Memory Accesses $\approx N^3 / B$.
    * Ratio $\approx B$ Ops per Byte.
    * **Result:** As long as $B$ fits in cache, performance scales with compute power, not memory bandwidth.

---

## 5. Data Layout: AoS vs SoA

When dealing with lists of objects (e.g., Particles with x, y, z, mass), layout matters.

### Array of Structures (AoS)
The intuitive object-oriented way.
~~~c
struct Particle {
    double x, y, z, mass;
};
Particle *p = malloc(N * sizeof(Particle));
~~~
* **Memory:** `x0 y0 z0 m0 x1 y1 z1 m1 ...`
* **Pros:** Good if you need all attributes of *one* particle (e.g., `update(p[i])`).
* **Cons:** Bad for SIMD. If you want to update just `x` for all particles, you load `y, z, mass` into cache uselessly (wasting 75% bandwidth).

### Structure of Arrays (SoA)
The data-oriented way.
~~~c
struct ParticleSystem {
    double *x;
    double *y;
    double *z;
    double *mass;
};
// Allocation
s.x = malloc(N * sizeof(double));
s.y = malloc(N * sizeof(double)); ...
~~~
* **Memory:** `x0 x1 x2 ... y0 y1 y2 ...`
* **Pros:** Perfect for SIMD. Loading `x` fills the cache line only with `x` values.
* **Cons:** Harder to manage (reallocating/resizing requires touching 4 arrays).
* **Recommendation:** Use SoA for heavy computation loops.



### Hybrid: Array of Structures of Arrays (AoSoA)
Also known as **Tiling**. Splits data into chunks where each chunk is SoA.
~~~c
struct Packet {
    float x[8]; // Matches SIMD width (AVX)
    float y[8];
    float z[8];
};
Packet *p = malloc(N/8 * sizeof(Packet));
~~~
* Combines SIMD efficiency with locality.

---

## 6. False Sharing

A concurrency performance killer in shared-memory systems.

**Scenario:**
* Core 1 writes to variable `A`.
* Core 2 writes to variable `B`.
* `A` and `B` are different variables, but they sit on the **same cache line** (64 bytes).

**Mechanism:**
1.  Core 1 modifies the cache line.
2.  Hardware Coherency (MESI protocol) marks the line as **Modified** on Core 1 and **Invalid** on Core 2.
3.  Core 2 tries to write `B`. It sees the line is Invalid.
4.  Core 2 must force Core 1 to write back to RAM/L3, then reload the line.
5.  **Ping-Pong Effect:** The cache line bounces between cores, killing performance.

**Solution:**
* **Padding:** Add dummy bytes between variables to ensure they are on different lines.
    ~~~c
    struct ShardedCounter {
        long count;
        char padding[64 - sizeof(long)];
    };
    ~~~
* **Thread Local Storage:** Each thread updates a local copy, merge at the end.

---

## 7. Data Alignment

CPUs read memory in chunks (words/lines). Accessing data that straddles a chunk boundary is inefficient.

* **Natural Alignment:** An `int` (4 bytes) should start at an address divisible by 4.
* **Cache Alignment:** Structures should ideally start at 64-byte boundaries.
* **SIMD Alignment:** AVX instructions (like `vmovaps`) crash if data is not aligned to 32 bytes (or 64 for AVX-512).

**How to align:**
~~~c
// Stack
struct alignas(64) AlignedStruct { ... };

// Heap (POSIX)
void *ptr;
posix_memalign(&ptr, 64, size);

// Intel Intrinsic
float *p = (float*)_mm_malloc(size, 32);
_mm_free(p);
~~~

---

## 8. Prefetching

Hiding memory latency by fetching data before it is needed.

### Hardware Prefetching
Modern CPUs automatically detect sequential access patterns (Stride-1).
* **Works for:** Iterating arrays `A[i]`.
* **Fails for:** Linked lists `node->next`, Hash maps, Random access `A[idx[i]]`.

### Software Prefetching
We can manually tell the CPU to load an address.
~~~c
#include <xmmintrin.h>

for (int i=0; i<N; i++) {
    // 1. Calculate address of FUTURE element
    // "distance" must be tuned (too small = wait; too large = cache pollution)
    int lookahead = i + 16; 
    
    // 2. Issue prefetch hint
    _mm_prefetch(&data[lookahead], _MM_HINT_T0);
    
    // 3. Do work on current element
    process(data[i]);
}
~~~

**Example: Binary Search**
Binary search is terrible for cache (jumps are random). Software prefetching can load the likely children of the current node before the comparison finishes.
~~~c
// Prefetching in irregular access
int idx = random_idx[i];
__builtin_prefetch(&arr[ next_random_idx ]); // GCC builtin
x = arr[idx];
~~~