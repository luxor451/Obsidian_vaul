**Tags:** #HPC #Optimization #CoreBound #DataDependency #ILP #LoopUnrolling #Compiler

---

## 1. Core Bound: The Problem

When applying the **[[6 Top-down Microarchitecture Analysis|TMA (Top-down Microarchitecture Analysis)]]** methodology, inefficient computations manifest as **[[6 Top-down Microarchitecture Analysis#4. Back-End Bound (The Stall)|Back-End Bound]] $\to$ Core Bound**.

There are two primary causes for Core Bound stalls:
1.  **Data Dependencies:** A long chain of dependent operations limits Instruction Level Parallelism (ILP). The CPU has slots, but cannot use them because instruction $N+1$ needs the result of instruction $N$.
2.  **Execution Port Contention:** The code issues too many instructions of a specific type (e.g., Divisions or Square Roots), overloading the limited number of hardware units available for that task.

---

## 2. Data Dependencies & Dependency Chains

A **Dependency Chain** occurs when a statement refers to the output of a preceding statement.

### Example: Pointer Chasing (Linked List)
~~~c
while (n) {
    sum += n->val;
    n = n->next; // Dependency!
}
~~~
* **Analysis:** The CPU cannot know the address of the next node until the current node is loaded. This serializes execution, defeating the **[[2 CPU Microarchitecture#Superscalar Execution|Superscalar]]** nature of modern CPUs.

### Example: Accumulator Dependency
~~~c
for (int i=0; i<N; i++) {
    sum += a[i]; // Dependency on 'sum'
}
~~~
* **Analysis:** Each addition depends on the previous value of `sum`. The latency of the ADD instruction (e.g., 3-4 cycles for float) becomes the bottleneck.
* **Impact:** Even if the CPU can do 4 additions per cycle, this code will do 1 addition every 4 cycles.

---

## 3. Breaking Dependencies: Multiple Accumulators

To break the dependency chain in a summation loop, we can use multiple variables to accumulate partial sums.

### Unoptimized Code
~~~c
double sum = 0;
for (int i=0; i<N; i++) {
    sum += a[i];
}
~~~

### Optimized Code (4 Accumulators)
~~~c
double sum1 = 0, sum2 = 0, sum3 = 0, sum4 = 0;
for (int i=0; i<N; i+=4) {
    sum1 += a[i];
    sum2 += a[i+1];
    sum3 += a[i+2];
    sum4 += a[i+3];
}
double sum = sum1 + sum2 + sum3 + sum4;
~~~
* **Result:** Now `sum1`, `sum2`, `sum3`, and `sum4` are independent. The CPU can execute 4 additions in parallel (limited only by throughput, not latency).
* **Compiler Support:** Compilers usually **will not** do this automatically for floating-point numbers because floating-point addition is not associative (`(a+b)+c != a+(b+c)`). You must do it manually or use `-ffast-math`.

---

## 4. Function Inlining

Function calls have overhead:
* Pushing arguments to stack/registers.
* Branching to the function address.
* Creating a stack frame.
* **Optimization Barrier:** The compiler treats function calls as "black boxes," preventing optimizations across the boundary.

**Solution: Inlining**
Replaces the function call with the actual body of the function.
* **Pros:** Eliminates overhead, enables further optimizations (constant propagation, dead code elimination).
* **Cons:** Increases code size (Instruction Cache pressure).

~~~c
// Before
int square(int x) { return x * x; }
y = square(a);

// After Inlining
y = a * a;
~~~

**How to trigger:**
* `inline` keyword (hint).
* `__attribute__((always_inline))` (force).
* Compile with `-O3` (auto-inline small functions).
* **LTO (Link Time Optimization):** Allows inlining functions defined in different `.c` files. Use `-flto`.

---

## 5. Loop Optimizations

Loops are where HPC programs spend most of their time. Optimizing them is crucial.

### A. Loop Invariant Code Motion (Hoisting)
Move computations that do not change inside the loop to the outside.

**Bad:**
~~~c
for (int i=0; i<N; i++) {
    x[i] = y[i] * (a + b); // (a+b) computed N times
}
~~~

**Good:**
~~~c
float factor = a + b; // Computed once
for (int i=0; i<N; i++) {
    x[i] = y[i] * factor;
}
~~~

### B. Loop Unswitching
If a loop contains a conditional that doesn't change per iteration, duplicate the loop.

**Bad:**
~~~c
for (int i=0; i<N; i++) {
    if (x < 5) a[i] = 0;
    else       a[i] = b[i];
}
~~~

**Good:**
~~~c
if (x < 5) {
    for (int i=0; i<N; i++) a[i] = 0;
} else {
    for (int i=0; i<N; i++) a[i] = b[i];
}
~~~
* **Benefit:** Removes the branch from the inner loop, enabling **[[3 Vector instructions SMID|vectorization]]**.

### C. Loop Fusion (Jamming)
Combine two loops into one.
* **Benefit:** Improves **Temporal Locality**. Data loaded for the first operation is reused immediately for the second, while it's still in the cache/registers.

**Bad:**
~~~c
for (int i=0; i<N; i++) a[i] = b[i] + 1;
for (int i=0; i<N; i++) c[i] = a[i] * 2; // a[i] needs to be reloaded
~~~

**Good:**
~~~c
for (int i=0; i<N; i++) {
    a[i] = b[i] + 1;
    c[i] = a[i] * 2; // a[i] is hot in register
}
~~~

### D. Loop Fission (Distribution)
Split one large loop into two smaller ones.
* **Benefit:** Reduces **Register Pressure**. If a loop body is too complex, the compiler runs out of registers and "spills" variables to stack (slow). Splitting allows each loop to use the full register set.
* **Benefit:** Improve **[[6 Top-down Microarchitecture Analysis#3. Front-End Bound (The Starvation)|I-Cache]]** usage.

**Bad (High Register Pressure):**
~~~c
for (int i=0; i<N; i++) {
    sum += a[i] + b[i];
    dot += c[i] * d[i];
}
~~~

**Good:**
~~~c
for (int i=0; i<N; i++) sum += a[i] + b[i];
for (int i=0; i<N; i++) dot += c[i] * d[i];
~~~

### E. Loop Unrolling
Manually duplicate the loop body $K$ times and decrease loop overhead (increment and branch).

**Normal:**
~~~c
for (int i=0; i<N; i++) {
    diff += a[i] - b[i];
}
~~~

**Unrolled (Factor 2):**
~~~c
for (int i=0; i<N; i+=2) {
    diff1 += a[i] - b[i];
    diff2 += a[i+1] - b[i+1];
}
diff = diff1 + diff2;
~~~
* **Benefit:** Increases ILP (independent instructions).
* **Risk:** Too much unrolling causes **Register Spilling** or I-Cache pollution.

---

## 6. Strength Reduction

Replace expensive operations with cheaper ones.

**Example 1: Division to Multiplication**
Division is very slow (20-80 cycles).
~~~c
// Slow
for (int i=0; i<N; i++) a[i] = b[i] / 2.5;

// Fast (Precompute reciprocal)
float inv = 1.0 / 2.5;
for (int i=0; i<N; i++) a[i] = b[i] * inv;
~~~
