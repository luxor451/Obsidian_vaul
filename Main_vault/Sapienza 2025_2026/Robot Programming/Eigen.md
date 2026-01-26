**Tags:** #Cpp #Eigen #LinearAlgebra #Robotics #MatrixLibrary
![[rp_06_eigen_light.pdf]]

---

## 1. What is Eigen?

**Eigen** is a high-level C++ library for linear algebra, matrix and vector operations, geometrical transformations, numerical solvers, and related algorithms.

**Key Features:**
* **Header-only:** No need to compile libraries; just include the headers.
* **Fast:** Uses Expression Templates to optimize operations and avoid temporary objects.
* **Versatile:** Supports fixed-size (stack) and dynamic-size (heap) matrices.
* **Standard:** Widely used in robotics (ROS, PCL, Google Ceres).

---

## 2. The Core: `Matrix` Class

In Eigen, all matrices and vectors are objects of the `Matrix` template class.

**Template Signature:**
```cpp
Matrix<typename Scalar, int RowsAtCompileTime, int ColsAtCompileTime>
```

* `Scalar`: The type of coefficients (e.g., `float`, `double`, `int`).
* `Rows`: Number of rows.
* `Cols`: Number of columns.

### Common Typedefs
Eigen provides convenient aliases for common types:
* `Matrix3f` $\rightarrow$ 3x3 matrix of floats.
* `Vector3f` $\rightarrow$ Column vector of 3 floats (`Matrix<float, 3, 1>`).
* `RowVector2d` $\rightarrow$ Row vector of 2 doubles.
* `MatrixXd` $\rightarrow$ Dynamic size matrix of doubles.

---

## 3. Fixed vs. Dynamic Size

### Fixed Size
* Dimensions are known at **compile time**.
* Allocated on the **Stack**.
* Very fast.
* Used for small objects (e.g., 3D points, rotation matrices).
* *Example:* `Matrix4f`

### Dynamic Size
* Dimensions are set at **runtime**.
* Allocated on the **Heap**.
* Used for large data structures or when size is unknown (e.g., Maps, Point Clouds).
* *Example:* `MatrixXd`

```cpp
// Fixed
Matrix3f A; // 3x3 float matrix

// Dynamic
MatrixXf B;       // Empty dynamic matrix
MatrixXf C(10,5); // 10x5 dynamic matrix, allocated now
```

---

## 4. Initialization and Access

### Comma Initializer
Allows intuitive filling of matrices.
```cpp
Matrix3f m;
m << 1, 2, 3,
     4, 5, 6,
     7, 8, 9;
```

### Predefined Matrices
```cpp
Matrix3f::Identity();       // Identity matrix
Vector3f::Zero();           // Vector of zeros
MatrixXd::Random(10, 10);   // 10x10 random matrix
MatrixXd::Constant(5, 5, 1.2); // 5x5 matrix filled with 1.2
```

### Accessing Elements
Uses parenthesis operator `(row, col)`.
```cpp
float x = m(0, 0); // Read
m(1, 2) = 5.0f;    // Write
```
*Note: Vectors can also use `[]` or `x()`, `y()`, `z()` accessors.*

---

## 5. Arithmetic Operations

Eigen overloads standard C++ operators for linear algebra.

**Basic Operations:**
```cpp
Matrix2f a, b, c;
// ... initialize ...
c = a + b;  // Matrix addition
c = a - b;  // Matrix subtraction
c = a * 2.0f; // Scalar multiplication
```

**Matrix Multiplication:**
The `*` operator performs standard matrix multiplication (rows $\times$ columns).
```cpp
Matrix2f A;
Vector2f v, result;
result = A * v; // Matrix-Vector multiplication
```

**Transposition and Inversion:**
```cpp
Matrix2f At = A.transpose();
Matrix2f Ai = A.inverse();
float det = A.determinant();
```

**Element-wise Operations:**
To perform operations element-by-element (like adding a scalar to every element, or multiplying two matrices element-wise), you must convert the matrix to an `array()`.
```cpp
// Adds 1 to every element
A = A.array() + 1.0f; 

// Coefficient-wise multiplication
C = A.array() * B.