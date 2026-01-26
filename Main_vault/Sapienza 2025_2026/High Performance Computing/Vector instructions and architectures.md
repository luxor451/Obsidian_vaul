![[HPC-3-Vector.pptx.pdf]]
# Data parallelism

**Data parallelism** is a computation model where the same
operation is performed simultaneously on multiple pieces of data.

CPU architectures provides data parallelism using vectorization :
- Fixed-Width Parallelism : operate on fixed-width lanes (128/256/512 bits) $\implies$ x86 SIMD instructions
- Variable-Length Parallelism : operates on variable length vector registers
# SIMD (Single Instruction Multiple Data)
![[2025-10-06-171003_hyprshot.png]]The SIMD instruction executes 8 DP floating point operations, corresponding to a whole cache line

SIMD instruction moved from 64 bits of MMX to 512 bits of AVX512

A single SIMD register can be configured with different data types 
### Compiler flags
GCC/CLANG compilers can exploit vectorisation with the right flags :
```
-msse
-msse2
...
-msse4.2
-mavx
-mavx2
-mavx512f
```
[Full list](https://www.intel.com/content/www/us/en/docs/intrinsics-guide/index.html) of x86 instructions

### Intrinsic Name Convention
The naming of SIMD instruction follow this convention
```
_mm[bits]_[operation]_[type][modifier]
```
1. Prefix
	-  \_mm -> 128-bit (xmm register)  
	-  \_mm -> 256-bit (ymm register)  
	- \_mm -> 512-bit (zmm register)  
2. Operation
	- add, sub, mul, div, and, or, xor...
	- Sometimes longer : fma (fused multiply-add), cvtepi32_ps (convert int32 to float) ...
3. Type Suffix
	- ps = packed single-precision float (float32)
	- pd = packed double-precision float (float64)
	- epi8/16/32/64 = signed integers of given width
	- epu8/16/32/64 = unsigned integers of given width
4. Optional Modifiers 
	- s = scalar (lowest element only)
	- u = unaligned load/store (e.g. \_mm_loadu_si128)
##### Example
1. \_mm256_add_ps
	- \_mm256 → 256-bit vector (ymm)
	- add → addition
	- ps → packed single-precision floats
adds 8 floats in parallel.
2. \_mm512_cvtepi32_pd
	- \_mm512 → 512-bit (zmm)
	- cvt → convert epi32 → from signed 32-bit integers
	- pd → to double-precision floats
converts 8 int32 → 16 float32.

```C
#include <immintrin.h>

...

int main(){
	float a[N], b[N], c[N]
	// Inint
	for (int i; i < N; i++){
		a[i] = i * 1.f; 
		b[i] = (N - i) * 1.f;
	}
	// perform vectorized addition using AVX
	for (int i; i < N; i++){
		// Load 8 floats from each array 
		_mm256 va = _mm256_load_ps(&a[i]);
		_mm256 vb = _mm256_load_ps(&b[i]);
		// Add the two vectors
		_mm256 vc = _mm256_add_ps(va,vb);
		// Store results into c
		_mm256_store_ps(&c[i], vc)
	}
}
```

### Code vectorization
**Code vectorization** : The software changes required to exploit SIMD instructions 

**Autovectorization** : today all the major compilers support
autovectorization for the popular processors, i.e., they can
generate SIMD instructions straight from high-level code written
in C/C++, Java, Rust, and other languages.

*However*, in some case, autovectorization does not work :
- Loop indices may overflow
- Pointers may overlap
- processors that don’t have efficient vector instructions for operations . (e.g. predicated operations) are not available on most processors


Modern compilers can report whether a certain loop was vectorized. If the compiler cannot vectorize a loop, it is also able to tell the reason why it failed.

Sometimes you can use compiler hints to drive vectorization

##### They are 3 phases to vectorize

1. **Legality-check**: in this phase, the compiler checks if it is legal to transform the loop using vectors.
	- checks that the iterations of the loop are consecutive,
	- ensures that all memory and arithmetic operations can be widened into consecutive operations.
	- check that the order of operations will be preserved.
 2. **Profitability-check**: the vectorizer checks if a transformation is profitable.
	- It compares different vectorization widths and figures out which one would be the fastest to execute.
	- The vectorizer uses a cost model to predict the cost of different operations, such as scalar add or vector load.
	- It takes into account the added instructions that shuffle data into registers, and estimate the cost of the loop guards that ensure that preconditions that allow vectorizations are met
3. **Transformation**: The vectorizer actually transforms the code
	- This process also includes the insertion of guards that enable vectorization. If the loop use an unknown iteration count, the compiler has to generate a scalar version of the loop (remainder), in addition to the vectorized version of the loop, to handle the last few iterations.
	- The compiler also has to check if pointers don’t overlap, etc. 
	
	All of these transformations are done using information that is collected during the legality check phase

##### Vectorization opportunities
A good indicator for potential vectorization opportunities is a
high [[CPU Microarchitecture#Out-Of-Order Execution|retiring rate]] (above 80%). 

Perhaps a workload executes a lot of simple instructions that
can be replaced by vector instructions. In such situations, high retiring metric doesn’t translate into high performance

##### Example 1 : illegal vectorization
```C
void vectorDependance(int *A, int n){
	for (int i = 1; i < n; i++){
		A[i]= A[i-1] * 2;
	}
}
```
This code cannot be vectorized due to read-after-write
```bash
clang -Rpass-analysis=loop-vectorize -march=native -O3 vectorDependence.c
```
![[2025-10-06-182443_hyprshot.png]]

Maybe unrolling ? 
```C
void vectorDependance_unrolled(int *A){
	// First element is addumed to be init
	for (int i = 1; i+7 < N; i+= 8){
		A[i]= A[i-1] * 2;
		A[i+1]= A[i-1] * 4;
		A[i+2]= A[i-1] * 8;
		A[i+3]= A[i-1] * 16;
		A[i+4]= A[i-1] * 32;
		A[i+5]= A[i-1] * 64;
		A[i+6]= A[i-1] * 128;
		A[i+7]= A[i-1] * 256;
	}
}
```
does not work(as in the openMP loop)
![[2025-10-06-182538_hyprshot.png]]

Directly use avx instructions
(differs from the openMP loop)

```C
void vectorDependance_unrolled_mm(int *A){
	// broadcast A[0] to 8 ints
	__m256i base = _mm256_set1_epi32(A[0]): 
	const int power2_const[8] = {2,4,8,16,32,64,128,256};
	__m256i power2 = _mm256_loadu_si256((__mm256i*)power2_const)
	
	__m256i vals = _mm256_mullo_epi32(base,power2)
	_mm256_storeu_si256((__m256i*)&A[1], vals);
	// Broadcast 256 to 8 bits
	base = _mm256set1_epi32(256); 
	for (int i = 9; i < N; i+= 8){
		// Compute the values by multiplying with A[i-8]
		vals = _mm256_mullo_epi32(vals, base);
		_mm256_storeu_si256((__m256i*)&A[i], vals);
	}
}
```
This compile but the loop is not vectorized
![[2025-10-06-182556_hyprshot.png]]

```C
void vectorDependence_vectorizable(int A*) {
	int base = A[0];
	for (int i = 1; i < N; i++) {
		A[i] = base << i;
	}
}
```
Does  vectorize !![[2025-10-06-182657_hyprshot.png]]
width:8 🡪 8 integer in the same SIMD
interleaving:4 🡪 4 SIMD instructions for each loop iteration

Can test at [godbolt.org](https://godbolt.org/);

##### Example 2 : floating-point arithmetic

```C
float calcSum(float* a, unsigned N) {
	float sum = 0.f;
	for (unsigned i = 0; i < N; i++) {
		sum += a[i];
	}
	return sum;
}
```
This code cannot be vectorized due to not associative FP operations
```bash
clang -Rpass-analysis=loop-vectorize -march=native -O3 calcsum.c
```
![[2025-10-07-135317_hyprshot 1.png]]
But, with the right option, it does vectorize
```shell
clang -Rpass=loop-vectorize -Rpass-analysis=loop-vectorize \
-ffast-math -march=native -O3 calcsum.c
```
![[2025-10-07-135516_hyprshot.png]]
##### Example 3 : memory overlap

```C
void foo(float* a; float* b, float* c; unsigned N) {
	for (unsigned i = 1; i < N; i++) {
		c[i] = b[i];
		a[i] = c[i-1];
	}
}
// Does not vectorized
```
When compilers cannot prove that a loop operates on arrays with
non-overlapping memory regions, they usually choose to be on the
safe side
```bash
clang -Rpass=loop-vectorize -Rpass-analysis=loop-vectorize \ 
-march=native -O3 overlap.c
```
![[2025-10-06-182730_hyprshot.png]]
When a developer knows arrays a, b, and c do not overlap, it can
insert the `__restrict__` keyword


```C
void foo(float* __restrict__ a; 
		 float* __restrict__ b, 
		 float* __restrict__ c; 
		 unsigned N) {
	 
	for (unsigned i = 1; i < N; i++) {
		c[i] = b[i];
		a[i] = c[i-1];
	}
}
```
![[2025-10-06-183052_hyprshot.png]]

##### Example 4: Vectorization Is Not Beneficial
```C
void stripedLoads(int *A, int* B, int n) {
	for (int i = 0; i < n; i++){
		A[i] += B[i * 3];
	}
}
```

```bash
clang -c -O3 -mavx strideload.c -Rpass-missed=loop-vectorize
```
![[2025-10-06-183259_hyprshot.png]]
### Programming style for better vectorization

- Use `__restrict__` attribute on pointers
- Use pragmas carefully:
	- to inform compiler
	- `#pragma unroll `can prevent loop vectorization
- avoid exceptions and break
-  Use contiguous and aligned memory access (e.g. use SoA)
-  Define local variables directly in the loop
-  Avoid function call (use inlining)

*Compiler flags*
`-ftree-vectorize`: enables automatic vectorization of loops
`-march=native`: generate code optimized for the CPU of the machine you are compiling on
`-fopt-info-vec`: generate vectorization info.
`-fopt-info-vec-missed` : reports loops that couldn’t be vectorized.

# Vector Architectures
