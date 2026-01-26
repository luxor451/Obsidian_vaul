**Tags:** #Cpp #Programming #OOP #MemoryManagement #Pointers #SmartPointers #STL #SoftwareEngineering 
![[rp_02_cpp_intro.pdf]]
![[rp_02b_cpp_inheritance.pdf]]
![[rp_03_cpp_metaprogramming.pdf]]

---

## 1. C++ vs C

C++ is effectively a superset of C ("C with Classes").
* **C:** Procedural, focused on functions and algorithms.
* **C++:** Multi-paradigm (Procedural + Object Oriented + Generic).
* **Strict Typing:** C++ enforces stricter type checking than C.
* **Compilation:** Managed by [[Compiler and build system#2. The Build Process (C/C++)|compilers and build tools]].

**Primitive Types:**
* `char` (1 byte)
* `short` (2 bytes)
* `int` (4 bytes)
* `long` (8 bytes)
* `float` (4 bytes)
* `double` (8 bytes)
* **Pointers:** 8 bytes (on 64-bit systems).

---

## 2. Pointers and Memory

A pointer is a variable that stores the **memory address** of another variable.

### Syntax
* `Type* ptr`: Declares a pointer to `Type`.
* `&var`: **Address-of operator**. Returns the memory address of `var`.
* `*ptr`: **Dereference operator**. Accesses the value stored at the address `ptr` points to.

### Example
```cpp
int a = 10;
int* a_ptr = &a; // a_ptr holds the address of a

*a_ptr = 20;     // changes the value of a to 20
```

### Arrays and Pointer Arithmetic
In C++, the name of an array is essentially a pointer to its first element.
* `v[i]` is equivalent to `*(v + i)`.
* Pointer arithmetic moves the pointer by `sizeof(Type)` bytes.

```cpp
int v[3];
int* p = v;
*(p+1) = 5; // sets v[1] to 5
```

---

## 3. References

A reference is an **alias** for an existing variable. It acts like a pointer that is automatically dereferenced.

**Differences from Pointers:**
1.  **Initialization:** Must be initialized when declared.
2.  **Immutability:** Cannot be reassigned to refer to a different variable.
3.  **Null:** Cannot be `nullptr` (must refer to valid memory).
4.  **Syntax:** Accessed like a normal variable (no `*` needed).

```cpp
int a = 10;
int& ref = a; // ref is now an alias for a
ref = 20;     // a becomes 20
```

---

## 4. Functions and Parameter Passing

Arguments can be passed to functions in three ways:

1.  **By Value:**
    * The function gets a **copy** of the variable.
    * Changes inside the function do **not** affect the original variable.
    * Expensive for large objects (copying takes time/memory).
    ```cpp
    void func(int a) { a = 10; } // Original unchanged
    ```

2.  **By Pointer:**
    * The function receives the **address** of the variable.
    * Changes affect the original variable.
    * Requires dereferencing inside the function.
    ```cpp
    void func(int* a) { *a = 10; } // Original changed
    ```

3.  **By Reference (C++ Style):**
    * The function receives an **alias**.
    * Changes affect the original variable.
    * Cleaner syntax (no pointer arithmetic).
    * **Best Practice:** Use `const Type&` for large objects (like images or matrices) to avoid copying while preventing modification.
    ```cpp
    void func(int& a) { a = 10; } // Original changed
    ```

---

## 5. User Defined Types (Structures)

Structures group variables (members) into a new data type.

### Struct vs. Class
In C++, `struct` and `class` are almost identical.
* **struct:** Members are `public` by default.
* **class:** Members are `private` by default.

### Layout in Memory
Members are stored sequentially in memory (with potential padding for alignment).
* **Access:** Use `.` for instances and `->` for pointers.

```cpp
struct Vector3 {
    float x;
    float y;
    float z;
};

Vector3 v;
v.x = 1.0;

Vector3* v_ptr = &v;
v_ptr->y = 2.0; // Equivalent to (*v_ptr).y
```

---

## 6. Memory Management: Stack vs. Heap

### The Stack (Automatic)
* **Allocation:** Fast, automatic when entering a scope (e.g., function call).
* **Deallocation:** Automatic when the variable goes out of scope.
* **Limit:** Size is limited (stack overflow possible).
* **Usage:** Local variables, function arguments.

```cpp
void foo() {
    int a;       // Allocated on stack
    Vector3 v;   // Allocated on stack
} // Both destroyed here
```

### The Heap (Dynamic)
* **Allocation:** Manual using `new`.
* **Deallocation:** Manual using `delete`.
* **Limit:** Limited only by system RAM.
* **Persistence:** Variables persist until explicitly deleted.
* **Risk:** Forgetting to `delete` causes **Memory Leaks**.

```cpp
void foo() {
    int* ptr = new int; // Allocates integer on heap
    *ptr = 5;
    delete ptr;         // Must release memory
}
```

**Arrays on Heap:**
```cpp
int* arr = new int[10];
delete[] arr; // Note the [] for arrays
```

---

## 7. Preprocessor & Namespaces

### Preprocessor Macros
Directives starting with `#` are processed before compilation.
* `#include <file>`: Pastes the content of `file` into the current file.
* `#define KEY val`: Replaces `KEY` with `val` everywhere.
* **Include Guards:** Prevent infinite recursion in header files.
    ```cpp
    #ifndef MY_HEADER_H
    #define MY_HEADER_H
    // code
    #endif
    ```

### Namespaces
Used to avoid name collisions (e.g., two libraries having a `Vector` struct).
* **Declaration:**
    ```cpp
    namespace geometry {
        struct Vector { ... };
    }
    ```
* **Usage:**
    ```cpp
    geometry::Vector v;
    using namespace geometry; // allows using Vector directly
    ```

---

# C++ Classes and Inheritance

## 1. Classes and Objects

In C++, classes are the building blocks of Object Oriented Programming (OOP). They encapsulate data (attributes) and behavior (methods).

### The `this` Pointer
* Inside any class method, the keyword `this` is a pointer to the instance on which the function was called.
* It is useful for disambiguating between member variables and parameters with the same name.

### Constructors and Destructors
* **Constructor:** A special method with the same name as the class. It initializes the object.
* **Destructor:** A special method named `~ClassName`. It cleans up resources (e.g., memory) when the object is destroyed.

```cpp
class MyClass {
public:
    int x;

    // Constructor with Initialization List
    MyClass(int val) : x(val) {
        // Body executes after initialization list
    }

    // Destructor
    ~MyClass() {
        // Cleanup code
    }
};
```

---

## 2. Inheritance Basics

Inheritance allows a class (Derived) to acquire properties and behaviors from another class (Base). It establishes an "Is-A" relationship (e.g., a `Car` is a `Vehicle`).

### Syntax
```cpp
class Base { ... };

class Derived : public Base {
    // Derived inherits members from Base
};
```

### Access Specifiers
* **public:** Accessible from anywhere.
* **private:** Accessible only within the class itself.
* **protected:** Accessible within the class **and its derived classes**, but private to the outside world.

**Inheritance Visibility:**
* `public` inheritance (standard): Public members of Base remain public in Derived.
* `protected` inheritance: Public/Protected members of Base become protected in Derived.
* `private` inheritance: Public/Protected members of Base become private in Derived.

### Construction and Destruction Order
1.  **Construction:** Base class constructor is called **first**, then the Derived class constructor.
2.  **Destruction:** Derived class destructor is called **first**, then the Base class destructor.

---

## 3. Shadowing (Overriding)

If a derived class defines a function with the same name as one in the base class, the derived version **shadows** (hides) the base version.

```cpp
struct Base {
    void print() { cout << "Base"; }
};

struct Derived : public Base {
    void print() { cout << "Derived"; }
};

Derived d;
d.print();       // Prints "Derived"
d.Base::print(); // Prints "Base" (explicit scoping)
```

---

## 4. Polymorphism and Virtual Functions

### The Problem
If you use a pointer of type `Base*` to point to a `Derived` object, calling a method will execute the **Base** version, not the Derived one. This happens because standard function calls are resolved at compile time (Static Binding).

```cpp
Derived d;
Base* ptr = &d;
ptr->print(); // Prints "Base" (Incorrect behavior for polymorphism)
```

### The Solution: `virtual`
The `virtual` keyword tells the compiler to resolve the function call at **runtime** (Dynamic Binding) based on the actual object type, not the pointer type.

```cpp
struct Base {
    virtual void print() { cout << "Base"; }
};

struct Derived : public Base {
    void print() override { cout << "Derived"; }
};

Base* ptr = new Derived();
ptr->print(); // Prints "Derived" (Correct!)
```

### V-Table (Virtual Method Table)
* To implement this, the compiler creates a hidden table (`vtable`) of function pointers for each class.
* Each object has a hidden pointer (`vptr`) pointing to this table.
* When a virtual function is called, the program looks up the correct address in the table.

---

## 5. Virtual Destructors

If a class has virtual functions, it **must** have a virtual destructor.
* **Reason:** If you `delete` a `Derived` object through a `Base*` pointer, and the destructor is not virtual, only `~Base()` is called. `~Derived()` is skipped, causing memory leaks.

```cpp
struct Base {
    virtual ~Base() {} // Crucial for cleanup!
};
```

---

## 6. Abstract Classes and Interfaces

### Pure Virtual Functions
A function that has no implementation in the base class and must be implemented by derived classes.
Syntax: `virtual void func() = 0;`

### Abstract Class
* A class containing at least one pure virtual function.
* **Cannot be instantiated** directly.
* Used to define **Interfaces** (a contract that derived classes must fulfill).

```cpp
struct Shape {
    virtual void draw() = 0; // Pure virtual
};

struct Circle : public Shape {
    void draw() override {
        // Must implement this to be instantiable
    }
};
```

---

# C++ Meta-programming and Smart Pointers


## 1. Templates and Generic Programming

**Meta-programming** refers to writing code that runs during compilation to generate the actual source code. In C++, this is achieved through **Templates**.

Templates allow you to write a single function or class that works with multiple data types. The compiler generates the specific version of the code for each type used at compile time.

### Function Templates
Instead of writing multiple functions for different types (e.g., `print(int)`, `print(float)`), you write one generic function.

**Syntax:**
```cpp
template <typename T>
void print(const T& val) {
    cout << "Value: " << val << endl;
}
```

**Usage:**
The compiler often infers the type automatically:
```cpp
print(10);      // T is int
print(5.5);     // T is double
print("Hello"); // T is const char*
```

### Class Templates
Classes can also be parameterized by type. This is essential for container classes (like `std::vector` or custom generic lists).

**Syntax:**
```cpp
template <typename T>
struct Entry {
    std::string key;
    T value;

    Entry(std::string k, T v) : key(k), value(v) {}
};
```

**Usage:**
Unlike functions, class templates usually require explicit type specification:
```cpp
Entry<int> e1("age", 25);
Entry<float> e2("height", 1.75f);
```

---

## 2. Advanced Template Features

### Non-Type Template Parameters
Templates can take values (integers, pointers) as parameters, not just types. This is useful for fixed-size containers or configuration.

**Example: Fixed-size Vector**
```cpp
template <typename T, int Dim>
struct Vec {
    T v[Dim]; // Array size is fixed at compile time

    T& operator[](int i) { return v[i]; }
};

Vec<float, 3> v3; // Creates a float vector of size 3
Vec<int, 10> v10; // Creates an int vector of size 10
```
*Note: `Vec<float, 3>` and `Vec<float, 4>` are distinct, incompatible types.*

### Template Specialization
You can define a specific implementation for a particular type, overriding the generic template.

**Example:**
```cpp
// Generic version
template <typename T>
void print(T v) { cout << "Generic: " << v << endl; }

// Specialized version for int
template <>
void print<int>(int v) { cout << "Specialized int: " << v << endl; }
```

This is commonly used for optimization or handling types that don't fit the generic logic (e.g., `std::vector<bool>` is a specialized version of `std::vector`).

---

## 3. Smart Pointers (C++11)

Raw pointers (`new`/`delete`) are prone to memory leaks (forgetting `delete`) and dangling pointers (accessing deleted memory). **Smart Pointers** automate memory management using the RAII (Resource Acquisition Is Initialization) paradigm: when the pointer goes out of scope, the memory is automatically freed.

### `std::unique_ptr`
* Represents **exclusive ownership**.
* Only one `unique_ptr` can point to a resource at a time.
* Cannot be copied, only **moved** (ownership transfer).
* Low overhead (almost same as raw pointer).



```cpp
#include <memory>

std::unique_ptr<int> p1(new int(10));
// std::unique_ptr<int> p2 = p1; // ERROR: Cannot copy
std::unique_ptr<int> p2 = std::move(p1); // OK: p1 is now empty, p2 owns the memory
```

### `std::shared_ptr`
* Represents **shared ownership**.
* Multiple `shared_ptr` instances can point to the same resource.
* Uses **Reference Counting**: The memory is deleted only when the last `shared_ptr` pointing to it is destroyed.
* Thread-safe reference counting (but slight overhead).



```cpp
std::shared_ptr<int> p1(new int(10)); // Ref count = 1
std::shared_ptr<int> p2 = p1;         // Ref count = 2
p1.reset();                           // Ref count = 1 (p2 still exists)
```

**`std::make_shared`:**
Always prefer `std::make_shared<T>()` over `new T()`. It allocates the object and the control block (reference counters) in a single memory block, improving performance and cache locality.

### `std::weak_ptr`
* A non-owning reference to a resource managed by `shared_ptr`.
* Does **not** increase the reference count.
* Used to break **Cyclic Dependencies** (e.g., A points to B, B points to A; with `shared_ptr`, they would never be deleted).
* Must be converted to `shared_ptr` (via `.lock()`) to access the data.

```cpp
std::shared_ptr<int> s_ptr = std::make_shared<int>(5);
std::weak_ptr<int> w_ptr = s_ptr;

if (std::shared_ptr<int> locked = w_ptr.lock()) {
    // Safe to use 'locked' here
} else {
    // The memory has been deleted
}
```