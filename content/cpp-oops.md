---
title: "C++ Complete Reference: Beginner to Advanced"
description: "in depth resource on cpp"
---

## PART 1: FOUNDATIONS

---

### 1.1 Basic Program Structure

Every C++ program begins execution at `main`. The preprocessor runs before compilation, handling `#include`, `#define`, and conditional compilation directives.

```cpp
#include <iostream>   // standard library header
#include "myfile.h"   // user-defined header

int main(int argc, char* argv[]) {
    std::cout << "Hello, World!\n";
    return 0;         // 0 = success, non-zero = error
}
```

Compilation pipeline: source (.cpp) → preprocessor → compiler → assembler → object file (.o) → linker → executable.

---

### 1.2 Fundamental Types

```cpp
// Integer types
bool        b = true;           // 1 byte, true/false
char        c = 'A';            // 1 byte, -128 to 127 or 0 to 255
signed char sc = -5;
unsigned char uc = 200;
short       s = 32767;          // at least 16 bits
int         i = 2147483647;     // at least 16 bits, typically 32
long        l = 2147483647L;    // at least 32 bits
long long   ll = 9223372036854775807LL; // at least 64 bits

// Unsigned variants
unsigned int ui = 4294967295U;
unsigned long long ull = 18446744073709551615ULL;

// Floating point
float       f = 3.14f;          // ~7 decimal digits, 32-bit IEEE 754
double      d = 3.141592653589793; // ~15 decimal digits, 64-bit IEEE 754
long double ld = 3.141592653589793L; // 80-bit extended or 128-bit

// Fixed-width types (preferred in systems code)
#include <cstdint>
int8_t, int16_t, int32_t, int64_t
uint8_t, uint16_t, uint32_t, uint64_t
int_fast32_t    // fastest type that is at least 32 bits
int_least32_t   // smallest type that is at least 32 bits
```

Type sizes are implementation-defined. Always use `sizeof(T)` at runtime or `<cstdint>` fixed-width types for portable code.

---

### 1.3 Variables, Initialization, and Storage Duration

```cpp
int x;              // default-initialized: indeterminate value (UB to read)
int y = 5;          // copy-initialization
int z(5);           // direct-initialization
int w{5};           // list-initialization (C++11, preferred: no narrowing)
int v{};            // value-initialization: zero for scalars
auto a = 42;        // type deduced as int
auto b = 3.14;      // type deduced as double
auto c = "hello";   // type deduced as const char*

// Storage duration
static int s_counter = 0; // static storage: lives for program lifetime
extern int external_var;  // declared here, defined elsewhere

// Thread-local (C++11)
thread_local int tl_var = 0; // each thread has its own copy

// Constants
const int MAX = 100;      // runtime constant
constexpr int SIZE = 256; // compile-time constant (C++11)
```

---

### 1.4 Operators

```cpp
// Arithmetic
+  -  *  /  %   // basic arithmetic; / truncates for integers
++x  x++        // pre/post increment; pre is generally preferred
--x  x--

// Relational
==  !=  <  >  <=  >=
<=>                // three-way comparison (C++20), returns std::strong_ordering

// Logical
&&  ||  !       // short-circuit: right side not evaluated if result known

// Bitwise
&   |   ^   ~   <<  >>
// << and >> on signed types: left shift of negative UB, right shift impl-defined

// Assignment
=   +=  -=  *=  /=  %=  &=  |=  ^=  <<=  >>=

// Other
?:              // ternary
,               // comma: evaluates both, yields right
sizeof          // compile-time size in bytes
alignof         // compile-time alignment requirement (C++11)
typeid          // runtime type info (requires RTTI)
static_cast, dynamic_cast, const_cast, reinterpret_cast
new  delete     // dynamic allocation

// Operator precedence (high to low, partial):
// :: . -> [] () post++/-- | pre++/-- unary+- ~ ! & * sizeof new delete cast
// .* ->* | * / % | + - | << >> | <=> | < > <= >= | == != | & | ^ | | | && | ||
// ?: = op= | , 
```

---

### 1.5 Control Flow

```cpp
// if / else if / else
if (x > 0) { ... }
else if (x == 0) { ... }
else { ... }

// if with init (C++17)
if (int n = compute(); n > 0) { use(n); }

// switch
switch (x) {
    case 1: ...; break;
    case 2: [[fallthrough]]; // intentional fallthrough (C++17)
    case 3: ...; break;
    default: ...;
}

// while
while (condition) { ... }

// do-while
do { ... } while (condition);

// for
for (int i = 0; i < n; ++i) { ... }

// range-based for (C++11)
for (auto& elem : container) { ... }
for (const auto& [key, val] : map) { ... } // structured bindings (C++17)

// Jump
break;      // exit loop or switch
continue;   // skip to next iteration
return val; // return from function
goto label; // unconditional jump (use sparingly, legitimate: error cleanup in C)

// Exception-based flow
try { ... }
catch (const std::exception& e) { ... }
catch (...) { ... }  // catch all
```

---

### 1.6 Functions

```cpp
// Declaration and definition
int add(int a, int b);          // declaration (prototype)
int add(int a, int b) { return a + b; } // definition

// Default arguments (must be rightmost)
void print(int x, int y = 0, int z = 0);

// Inline (hint to compiler, mandatory for header-defined non-templates)
inline int square(int x) { return x * x; }

// Function overloading
int    process(int x);
double process(double x);
void   process(const std::string& s);

// Variadic functions (C-style, type-unsafe)
#include <cstdarg>
int sum(int count, ...) {
    va_list args;
    va_start(args, count);
    int total = 0;
    for (int i = 0; i < count; ++i)
        total += va_arg(args, int);
    va_end(args);
    return total;
}

// Variadic templates (C++11, type-safe)
template<typename... Args>
void log(Args&&... args) { (std::cout << ... << args); } // fold expression (C++17)

// Function pointers
int (*fp)(int, int) = add;
int result = fp(2, 3);

// std::function (type-erased callable)
#include <functional>
std::function<int(int,int)> f = add;
f = [](int a, int b) { return a + b; }; // can hold lambdas too
```

---

### 1.7 Pointers and References

```cpp
int x = 42;

// Pointers
int* p = &x;        // p holds address of x
*p = 100;           // dereference: modifies x
int** pp = &p;      // pointer to pointer
int* q = nullptr;   // null pointer (C++11; prefer over NULL or 0)

// Pointer arithmetic (only valid within arrays)
int arr[5] = {1,2,3,4,5};
int* pa = arr;
pa++;               // advances by sizeof(int) bytes
*(pa + 2) == arr[3]; // true

// References
int& r = x;         // r is an alias for x; must be initialized
r = 200;            // modifies x
const int& cr = x;  // read-only alias; can bind to temporaries

// Rvalue references (C++11)
int&& rr = 42;      // binds to rvalue/temporary
void foo(int&& x);  // move semantics

// Pointer to const vs const pointer
const int* pc = &x;  // pointer to const int; can reassign pc
int* const cp = &x;  // const pointer to int; cannot reassign cp
const int* const cpc = &x; // both const

// Casting pointers
void* vp = p;               // implicit to void*
int* ip = static_cast<int*>(vp); // explicit from void*
char* cp2 = reinterpret_cast<char*>(p); // byte-level reinterpret

// Smart pointers (covered in depth later)
#include <memory>
std::unique_ptr<int> up = std::make_unique<int>(42);
std::shared_ptr<int> sp = std::make_shared<int>(42);
std::weak_ptr<int>   wp = sp;
```

---

### 1.8 Arrays and Strings

```cpp
// C-style arrays
int a[5] = {1, 2, 3, 4, 5};
int b[] = {1, 2, 3};        // size deduced as 3
int c[3]{};                 // value-initialized to {0,0,0}
int matrix[3][4];           // 2D array, row-major in memory

// Decay: array name decays to pointer to first element
int* p = a;                 // sizeof(p) != sizeof(a)

// std::array (C++11, fixed-size, no decay, stack allocated)
#include <array>
std::array<int, 5> sa = {1, 2, 3, 4, 5};
sa.size(); sa.front(); sa.back(); sa.data();

// std::vector (dynamic, heap allocated)
#include <vector>
std::vector<int> v = {1, 2, 3};
v.push_back(4);
v.emplace_back(5);          // constructs in place
v.size(); v.capacity();
v.reserve(100);             // preallocate; no reallocation until 100 elements
v.resize(10);               // resize; new elements value-initialized
v[2];                       // no bounds check
v.at(2);                    // bounds-checked; throws std::out_of_range

// C-style strings
char s[] = "hello";         // 6 chars including '\0'
char* ps = "world";         // deprecated in C++11, UB to modify
strlen(s); strcpy; strcat; strcmp; // <cstring>

// std::string
#include <string>
std::string str = "hello";
str += " world";
str.size(); str.length();
str.substr(0, 5);
str.find("world");          // returns std::string::npos if not found
str.c_str();                // null-terminated C string
std::to_string(42);
std::stoi("42"); std::stod("3.14");

// std::string_view (C++17, non-owning reference to string data)
#include <string_view>
std::string_view sv = str;  // no copy; watch lifetime
sv.substr(0, 5);            // returns another view, no allocation
```

---

## PART 2: OBJECT-ORIENTED PROGRAMMING

---

### 2.1 Classes and Structs

In C++, `struct` and `class` are nearly identical. The only difference is default access: `struct` defaults to `public`, `class` defaults to `private`.

```cpp
class Point {
public:                         // access specifier
    double x, y;

    // Constructors
    Point() : x(0), y(0) {}              // default constructor
    Point(double x, double y) : x(x), y(y) {} // parameterized
    Point(const Point& other) : x(other.x), y(other.y) {} // copy constructor
    Point(Point&& other) noexcept : x(other.x), y(other.y) {} // move constructor

    // Destructor
    ~Point() {}                           // called when object goes out of scope

    // Copy and move assignment
    Point& operator=(const Point& other) { x = other.x; y = other.y; return *this; }
    Point& operator=(Point&& other) noexcept { x = other.x; y = other.y; return *this; }

    // Member functions
    double distance() const { return std::sqrt(x*x + y*y); } // const: doesn't modify *this
    void translate(double dx, double dy) { x += dx; y += dy; }

    // Static members
    static int instance_count;
    static int getCount() { return instance_count; }

private:
    // private members inaccessible outside class
};

int Point::instance_count = 0; // static member definition (outside class)
```

---

### 2.2 The Rule of Zero, Three, Five

The "Rule of Three" (pre-C++11): if you define any of destructor, copy constructor, or copy assignment operator, you probably need to define all three.

The "Rule of Five" (C++11): also include move constructor and move assignment operator.

The "Rule of Zero" (best practice): design your class to not need any of the five special members by using RAII types (smart pointers, std::string, std::vector). Let the compiler generate them all.

```cpp
// Rule of Five example for a class managing raw memory
class Buffer {
    size_t size_;
    int* data_;

public:
    explicit Buffer(size_t size) : size_(size), data_(new int[size]) {}

    ~Buffer() { delete[] data_; }   // 1. destructor

    Buffer(const Buffer& other)     // 2. copy constructor
        : size_(other.size_), data_(new int[other.size_]) {
        std::copy(other.data_, other.data_ + size_, data_);
    }

    Buffer& operator=(const Buffer& other) { // 3. copy assignment
        if (this != &other) {
            delete[] data_;
            size_ = other.size_;
            data_ = new int[size_];
            std::copy(other.data_, other.data_ + size_, data_);
        }
        return *this;
    }

    Buffer(Buffer&& other) noexcept // 4. move constructor
        : size_(other.size_), data_(other.data_) {
        other.size_ = 0;
        other.data_ = nullptr;
    }

    Buffer& operator=(Buffer&& other) noexcept { // 5. move assignment
        if (this != &other) {
            delete[] data_;
            size_ = other.size_;
            data_ = other.data_;
            other.size_ = 0;
            other.data_ = nullptr;
        }
        return *this;
    }
};

// Rule of Zero equivalent (preferred):
class BufferZero {
    std::vector<int> data_;
public:
    explicit BufferZero(size_t size) : data_(size) {}
    // compiler generates all five correctly
};
```

---

### 2.3 Operator Overloading

```cpp
class Vector2 {
public:
    double x, y;
    Vector2(double x = 0, double y = 0) : x(x), y(y) {}

    // Arithmetic operators (as member functions)
    Vector2 operator+(const Vector2& rhs) const { return {x + rhs.x, y + rhs.y}; }
    Vector2 operator-(const Vector2& rhs) const { return {x - rhs.x, y - rhs.y}; }
    Vector2 operator*(double scalar)      const { return {x * scalar, y * scalar}; }
    Vector2& operator+=(const Vector2& rhs) { x += rhs.x; y += rhs.y; return *this; }

    // Unary
    Vector2 operator-() const { return {-x, -y}; }

    // Comparison (C++20 can auto-generate with =default)
    bool operator==(const Vector2& rhs) const { return x == rhs.x && y == rhs.y; }
    bool operator!=(const Vector2& rhs) const { return !(*this == rhs); }

    // Subscript
    double& operator[](int idx) { return idx == 0 ? x : y; }
    const double& operator[](int idx) const { return idx == 0 ? x : y; }

    // Function call (functor)
    double operator()() const { return std::sqrt(x*x + y*y); }

    // Conversion operator
    explicit operator bool() const { return x != 0 || y != 0; }

    // Increment/decrement
    Vector2& operator++() { ++x; ++y; return *this; }          // pre-increment
    Vector2  operator++(int) { Vector2 tmp = *this; ++*this; return tmp; } // post-increment

    // Stream operators (must be free/friend for symmetry)
    friend std::ostream& operator<<(std::ostream& os, const Vector2& v) {
        return os << "(" << v.x << ", " << v.y << ")";
    }
    friend std::istream& operator>>(std::istream& is, Vector2& v) {
        return is >> v.x >> v.y;
    }
};

// Scalar * vector (must be free function for commutativity)
Vector2 operator*(double scalar, const Vector2& v) { return v * scalar; }
```

Rules for operator overloading:
- Cannot change arity, precedence, or associativity of operators
- Cannot overload `::`, `.`, `.*`, `?:`, `sizeof`, `typeid`, `alignof`
- `=`, `[]`, `()`, `->` must be member functions
- Prefer free functions for symmetric binary operators
- Overloading `&&`, `||` loses short-circuit evaluation

---

### 2.4 Inheritance

```cpp
class Animal {
protected:              // accessible in derived classes
    std::string name_;
    int age_;

public:
    Animal(std::string name, int age) : name_(std::move(name)), age_(age) {}
    virtual ~Animal() {}        // ALWAYS virtual destructor in polymorphic base

    virtual std::string sound() const { return "..."; } // virtual: overridable
    virtual void describe() const {
        std::cout << name_ << " (" << age_ << " years)\n";
    }
    std::string getName() const { return name_; } // non-virtual: not overridable
};

class Dog : public Animal {         // public inheritance: is-a relationship
public:
    Dog(std::string name, int age) : Animal(std::move(name), age) {} // delegating to base

    std::string sound() const override { return "Woof"; } // override (C++11): compiler checks
    void describe() const override final {                // final: no further override
        Animal::describe();                               // call base class version
        std::cout << "  Sound: " << sound() << "\n";
    }

    void fetch() { std::cout << name_ << " fetches!\n"; } // Dog-specific method
};

class Cat : public Animal {
public:
    Cat(std::string name, int age) : Animal(std::move(name), age) {}
    std::string sound() const override { return "Meow"; }
};

// Polymorphism via pointers/references
std::vector<std::unique_ptr<Animal>> animals;
animals.push_back(std::make_unique<Dog>("Rex", 3));
animals.push_back(std::make_unique<Cat>("Whiskers", 5));

for (const auto& a : animals)
    a->describe();     // calls correct override at runtime (vtable dispatch)

// Inheritance types
class B : public A {};      // public: public/protected members of A remain so
class B : protected A {};   // protected: public members of A become protected
class B : private A {};     // private: all members of A become private (has-a via inheritance)

// Explicit upcast/downcast
Animal* a = new Dog("Buddy", 2);    // implicit upcast (safe)
Dog* d = dynamic_cast<Dog*>(a);     // downcast with runtime check; null if fails
Dog& dr = dynamic_cast<Dog&>(*a);   // throws std::bad_cast if fails
Dog* d2 = static_cast<Dog*>(a);     // downcast without check (UB if wrong type)
```

---

### 2.5 Multiple Inheritance and Virtual Inheritance

```cpp
class Flyable {
public:
    virtual void fly() { std::cout << "Flying\n"; }
    virtual ~Flyable() {}
};

class Swimmable {
public:
    virtual void swim() { std::cout << "Swimming\n"; }
    virtual ~Swimmable() {}
};

class Duck : public Animal, public Flyable, public Swimmable {
public:
    Duck(std::string name) : Animal(std::move(name), 1) {}
    void fly() override { std::cout << name_ << " flies low\n"; }
    void swim() override { std::cout << name_ << " paddles\n"; }
    std::string sound() const override { return "Quack"; }
};

// Diamond problem and virtual inheritance
class A { public: int val; };
class B : virtual public A {};   // virtual base
class C : virtual public A {};   // virtual base
class D : public B, public C {}; // only one A subobject in D

D d;
d.val = 42;         // unambiguous
d.A::val = 42;      // also valid, same object
d.B::val = 42;      // same object

// Ambiguity resolution
class B2 : public A {};
class C2 : public A {};
class D2 : public B2, public C2 {};
D2 d2;
d2.B2::val = 1;     // disambiguate explicitly
d2.C2::val = 2;     // separate copies
```

---

### 2.6 Abstract Classes and Interfaces

```cpp
// Abstract class: has at least one pure virtual function
class Shape {
public:
    virtual ~Shape() {}
    virtual double area() const = 0;       // pure virtual: no implementation here
    virtual double perimeter() const = 0;
    virtual void draw() const = 0;

    // Concrete method in abstract class (allowed)
    virtual void printInfo() const {
        std::cout << "Area: " << area() << ", Perimeter: " << perimeter() << "\n";
    }
};

// Cannot instantiate: Shape s; // error

class Circle : public Shape {
    double radius_;
public:
    explicit Circle(double r) : radius_(r) {}
    double area()      const override { return M_PI * radius_ * radius_; }
    double perimeter() const override { return 2 * M_PI * radius_; }
    void   draw()      const override { std::cout << "Drawing circle\n"; }
};

class Rectangle : public Shape {
    double w_, h_;
public:
    Rectangle(double w, double h) : w_(w), h_(h) {}
    double area()      const override { return w_ * h_; }
    double perimeter() const override { return 2 * (w_ + h_); }
    void   draw()      const override { std::cout << "Drawing rectangle\n"; }
};

// Pure interface (all pure virtual, no data members)
class ISerializable {
public:
    virtual ~ISerializable() = default;
    virtual std::string serialize() const = 0;
    virtual void deserialize(const std::string& data) = 0;
};
```

---

### 2.7 The Vtable Mechanism

When a class has at least one virtual function, the compiler creates a virtual function table (vtable): a static array of function pointers, one per virtual function. Each object of the class contains a hidden vptr (virtual pointer) pointing to this table.

```
Object layout (approximate):
[ vptr ] --> vtable: [&Base::foo, &Base::bar, ...]
[ member1 ]
[ member2 ]
...

Derived object:
[ vptr ] --> vtable: [&Derived::foo, &Base::bar, ...] // override replaced
[ Base members ]
[ Derived members ]
```

Virtual call cost: one pointer dereference (vptr) + one indexed load (vtable) + indirect call. This inhibits inlining. Devirtualization by the compiler is possible when the dynamic type is known at compile time.

`final` on a class or function allows compilers to devirtualize more aggressively.

---

### 2.8 Constructors: Deep Dive

```cpp
class MyClass {
    int a_;
    std::string b_;
    std::vector<int> c_;

public:
    // Member initializer list (always prefer over assignment in body)
    MyClass(int a, std::string b) : a_(a), b_(std::move(b)), c_() {}

    // Delegating constructors (C++11): one constructor calls another
    MyClass() : MyClass(0, "") {}
    MyClass(int a) : MyClass(a, "default") {}

    // Explicit: prevents implicit conversion
    explicit MyClass(int a) : a_(a), b_(), c_() {}
    // MyClass obj = 42; // error with explicit
    // MyClass obj(42);  // OK

    // = default: compiler-generated
    MyClass(const MyClass&) = default;
    MyClass& operator=(const MyClass&) = default;

    // = delete: suppress generation
    MyClass(const MyClass&) = delete;
    MyClass& operator=(const MyClass&) = delete;
};

// Aggregate initialization (no user-declared constructors, etc.)
struct Point { int x, y, z; };
Point p = {1, 2, 3};       // OK
Point q{1, 2, 3};          // OK (C++11)
Point r{.x=1, .z=3};       // designated initializers (C++20)

// Uniform initialization and std::initializer_list
class IntList {
public:
    IntList(std::initializer_list<int> il) {
        for (auto v : il) data_.push_back(v);
    }
private:
    std::vector<int> data_;
};
IntList il = {1, 2, 3, 4, 5}; // calls initializer_list constructor
```

Member initialization order follows declaration order in the class, not the order in the initializer list. The compiler warns if these differ.

---

### 2.9 Friend Functions and Classes

```cpp
class Matrix;  // forward declaration

class Vector {
    double* data_;
    int size_;
public:
    Vector(int n) : data_(new double[n]), size_(n) {}
    ~Vector() { delete[] data_; }

    // Grant Matrix access to private members
    friend class Matrix;

    // Grant a specific free function access
    friend double dot(const Vector& a, const Vector& b);

    // Friend operator (common pattern)
    friend std::ostream& operator<<(std::ostream& os, const Vector& v);
};

double dot(const Vector& a, const Vector& b) {
    // Can access a.data_, a.size_ directly
    double sum = 0;
    for (int i = 0; i < a.size_; ++i) sum += a.data_[i] * b.data_[i];
    return sum;
}
```

Friendship is not inherited, not transitive, and not symmetric.

---

## PART 3: TEMPLATES AND GENERIC PROGRAMMING

---

### 3.1 Function Templates

```cpp
// Basic function template
template<typename T>
T max(T a, T b) { return a > b ? a : b; }

// Usage
max(3, 5);           // T deduced as int
max(3.14, 2.71);     // T deduced as double
max<std::string>("hello", "world"); // explicit instantiation

// Multiple type parameters
template<typename T, typename U>
auto add(T a, U b) -> decltype(a + b) {  // trailing return type
    return a + b;
}

// C++14: auto return type deduction
template<typename T, typename U>
auto add14(T a, U b) { return a + b; }

// Non-type template parameters
template<typename T, int N>
void printArray(T (&arr)[N]) {
    for (int i = 0; i < N; ++i) std::cout << arr[i] << " ";
}

int arr[5] = {1,2,3,4,5};
printArray(arr); // N deduced as 5

// Template with default argument
template<typename T = int>
T zero() { return T(0); }

// Function template specialization (partial not allowed for functions)
template<>
const char* max<const char*>(const char* a, const char* b) {
    return strcmp(a, b) > 0 ? a : b;
}
```

---

### 3.2 Class Templates

```cpp
template<typename T>
class Stack {
    std::vector<T> data_;

public:
    void push(const T& val) { data_.push_back(val); }
    void push(T&& val) { data_.push_back(std::move(val)); }

    template<typename... Args>
    void emplace(Args&&... args) { data_.emplace_back(std::forward<Args>(args)...); }

    void pop() {
        if (data_.empty()) throw std::underflow_error("Stack is empty");
        data_.pop_back();
    }

    T& top() { return data_.back(); }
    const T& top() const { return data_.back(); }
    bool empty() const { return data_.empty(); }
    size_t size() const { return data_.size(); }
};

Stack<int> si;
Stack<std::string> ss;

// Partial specialization (only for class templates)
template<typename T>
class Stack<T*> {  // specialization for pointer types
    std::vector<T*> data_;
    // ... different implementation
};

// Full specialization
template<>
class Stack<bool> { // bit-optimized implementation
    std::vector<uint8_t> data_;
    // ...
};

// Class template with non-type parameter
template<typename T, size_t N>
class FixedArray {
    T data_[N];
public:
    T& operator[](size_t i) { return data_[i]; }
    constexpr size_t size() const { return N; }
};

// Template template parameters
template<typename T, template<typename, typename> class Container = std::vector>
class GenericStack {
    Container<T, std::allocator<T>> data_;
};
```

---

### 3.3 Variadic Templates

```cpp
// Base case
template<typename T>
void print(T t) { std::cout << t << "\n"; }

// Recursive variadic template
template<typename T, typename... Rest>
void print(T first, Rest... rest) {
    std::cout << first << " ";
    print(rest...);  // recursive call with one fewer argument
}

// sizeof... operator
template<typename... Args>
void countArgs(Args... args) {
    std::cout << sizeof...(args) << " arguments\n";
}

// Fold expressions (C++17): cleaner alternative to recursion
template<typename... Args>
auto sum(Args... args) { return (args + ...); }        // unary right fold
// (args + ...) expands to: a1 + (a2 + (a3 + ... + aN))

template<typename... Args>
auto sumLeft(Args... args) { return (... + args); }    // unary left fold

template<typename... Args>
auto sumInit(Args... args) { return (0 + ... + args); } // binary left fold with init

// Fold with comma operator (for side effects)
template<typename... Args>
void printAll(Args&&... args) { (std::cout << ... << args); }

// Pack expansion in initializer list
template<typename... Args>
std::vector<int> makeVec(Args... args) { return {args...}; }

// Perfect forwarding with variadic templates
template<typename T, typename... Args>
std::unique_ptr<T> make(Args&&... args) {
    return std::unique_ptr<T>(new T(std::forward<Args>(args)...));
}
```

---

### 3.4 Template Metaprogramming (TMP)

TMP exploits the fact that template instantiation happens at compile time, enabling computations performed entirely by the compiler.

```cpp
// Compile-time factorial
template<int N>
struct Factorial {
    static constexpr int value = N * Factorial<N-1>::value;
};
template<>
struct Factorial<0> {
    static constexpr int value = 1;
};
static_assert(Factorial<5>::value == 120);

// Type traits
template<typename T>
struct IsPointer { static constexpr bool value = false; };
template<typename T>
struct IsPointer<T*> { static constexpr bool value = true; };

// Using <type_traits> (prefer the standard library)
#include <type_traits>
std::is_pointer<int*>::value      // true
std::is_integral<int>::value      // true
std::is_floating_point<double>::value
std::is_same<int, int>::value
std::is_base_of<Base, Derived>::value
std::is_convertible<Derived*, Base*>::value
std::remove_const<const int>::type  // int
std::remove_reference<int&>::type   // int
std::decay<int[]>::type             // int*
std::conditional<true, int, double>::type  // int
std::enable_if<condition, T>::type  // T if condition true, else error

// Compile-time type list manipulation
template<typename... Types>
struct TypeList {};
using MyList = TypeList<int, double, std::string>;

// constexpr (C++11/14/17/20): preferred over TMP for value computation
constexpr int factorial(int n) {
    return n <= 1 ? 1 : n * factorial(n - 1);
}
static_assert(factorial(5) == 120);

// if constexpr (C++17): compile-time branching
template<typename T>
void process(T val) {
    if constexpr (std::is_integral_v<T>) {
        // Only compiled for integral types
        std::cout << "Integer: " << val << "\n";
    } else if constexpr (std::is_floating_point_v<T>) {
        std::cout << "Float: " << val << "\n";
    } else {
        std::cout << "Other: " << val << "\n";
    }
}
```

---

### 3.5 SFINAE (Substitution Failure Is Not An Error)

When template argument substitution fails, it's not a compilation error — that overload is simply removed from consideration.

```cpp
// enable_if-based SFINAE
template<typename T>
typename std::enable_if<std::is_integral<T>::value, T>::type
double_it(T x) { return x * 2; }

template<typename T>
typename std::enable_if<std::is_floating_point<T>::value, T>::type
double_it(T x) { return x * 2.0; }

// Shorter with _v and _t aliases (C++17)
template<typename T, std::enable_if_t<std::is_integral_v<T>, int> = 0>
T double_it2(T x) { return x * 2; }

// void_t idiom (C++17): detect if an expression is valid
template<typename, typename = void>
struct has_begin : std::false_type {};

template<typename T>
struct has_begin<T, std::void_t<decltype(std::declval<T>().begin())>>
    : std::true_type {};

// Detection idiom
template<typename T>
constexpr bool is_iterable_v = has_begin<T>::value;

// Concepts (C++20): much cleaner replacement for SFINAE
template<typename T>
concept Integral = std::is_integral_v<T>;

template<typename T>
concept Container = requires(T c) {
    c.begin();
    c.end();
    c.size();
    typename T::value_type;
};

template<Integral T>
T double_concept(T x) { return x * 2; }

template<Container C>
void printContainer(const C& c) {
    for (const auto& elem : c) std::cout << elem << " ";
}

// Concept with requires clause
template<typename T>
requires std::is_arithmetic_v<T> && (sizeof(T) >= 4)
T largeMath(T a, T b) { return a * b + a - b; }
```

---

### 3.6 Move Semantics and Perfect Forwarding

```cpp
// lvalue: has an address, can appear on left of assignment
// rvalue: temporary, no persistent identity, can only appear on right

int x = 5;    // x is lvalue, 5 is rvalue
int y = x;    // x is lvalue used as rvalue (copy)

// std::move: cast to rvalue reference (does not actually move)
std::vector<int> v1 = {1, 2, 3};
std::vector<int> v2 = std::move(v1);  // v1 is now "moved-from" (valid but unspecified state)

// Move constructor: O(1) for containers, just pointer steal
class String {
    char* data_;
    size_t size_;
public:
    String(const char* s) : size_(strlen(s)), data_(new char[size_+1]) {
        memcpy(data_, s, size_+1);
    }
    ~String() { delete[] data_; }

    // Move constructor: steal the pointer, null the source
    String(String&& other) noexcept
        : data_(other.data_), size_(other.size_) {
        other.data_ = nullptr;
        other.size_ = 0;
    }
};

// Perfect forwarding: forward the value category (lvalue vs rvalue)
template<typename T>
void wrapper(T&& arg) {           // T&& here is a "forwarding reference", not rvalue ref
    // T deduced as int&  if arg is lvalue: arg becomes int& &&, collapses to int&
    // T deduced as int   if arg is rvalue: arg becomes int&&
    process(std::forward<T>(arg)); // preserves value category
}

// Universal/forwarding references vs rvalue references
template<typename T>
void f(T&& x);      // forwarding reference (T is deduced)
void g(int&& x);    // rvalue reference (not a template)
auto&& a = expr;    // forwarding reference (type deduced)

// Reference collapsing rules:
// T& &   -> T&
// T& &&  -> T&
// T&& &  -> T&
// T&& && -> T&&

// std::forward implementation:
template<typename T>
T&& forward(std::remove_reference_t<T>& t) noexcept {
    return static_cast<T&&>(t);
}
```

---

## PART 4: MEMORY MANAGEMENT

---

### 4.1 Stack vs Heap

Stack: LIFO, automatic management, size limited (typically 1-8 MB), very fast allocation (just decrement stack pointer), no fragmentation.

Heap: explicit allocation, virtually unlimited, slower, can fragment, requires deallocation.

```cpp
// Stack allocation
void f() {
    int arr[1000];       // 4000 bytes on stack; fine
    int bigArr[1000000]; // likely stack overflow
}

// Heap allocation
int* p = new int(42);       // allocate single int
int* arr = new int[100];    // allocate array
delete p;                   // free single
delete[] arr;               // free array (MUST match new[])

// Placement new: construct in pre-allocated memory
char buf[sizeof(MyClass)];
MyClass* obj = new(buf) MyClass(args);  // no allocation, just construction
obj->~MyClass();                        // must explicitly call destructor
```

---

### 4.2 Smart Pointers

```cpp
#include <memory>

// unique_ptr: sole ownership, zero overhead vs raw pointer
std::unique_ptr<int> up1 = std::make_unique<int>(42);  // C++14
std::unique_ptr<int> up2 = std::make_unique<int>();    // value-initialized

// Cannot copy; can only move
std::unique_ptr<int> up3 = std::move(up1); // up1 is now null
up2.reset();         // delete and set to null
up2.reset(new int);  // delete old, take ownership of new
int* raw = up3.get(); // access raw pointer (up3 still owns)
int* raw2 = up3.release(); // relinquish ownership; caller must delete

// Custom deleter
auto fileDeleter = [](FILE* f) { if (f) fclose(f); };
std::unique_ptr<FILE, decltype(fileDeleter)> fp(fopen("f.txt","r"), fileDeleter);

// unique_ptr for arrays
std::unique_ptr<int[]> arr = std::make_unique<int[]>(100);
arr[5] = 42;         // operator[] supported

// shared_ptr: shared ownership via reference counting
std::shared_ptr<int> sp1 = std::make_shared<int>(42); // single allocation (preferred)
std::shared_ptr<int> sp2 = sp1;  // reference count = 2
sp1.reset();                     // count = 1; object not deleted
sp2.reset();                     // count = 0; object deleted

sp1.use_count();     // get reference count (avoid using in logic; racy in MT)

// weak_ptr: non-owning observer; breaks cycles
std::weak_ptr<int> wp = sp1;
if (auto locked = wp.lock()) {   // returns shared_ptr; null if object destroyed
    *locked = 100;
}
wp.expired();        // true if object is destroyed

// Cyclic reference problem and solution
struct Node {
    std::shared_ptr<Node> next;  // cycle if two nodes point to each other
    std::weak_ptr<Node> prev;    // use weak_ptr for back-pointers to break cycle
};

// enable_shared_from_this: get shared_ptr from within member function
class MyClass : public std::enable_shared_from_this<MyClass> {
public:
    std::shared_ptr<MyClass> getShared() { return shared_from_this(); }
    // Never return std::shared_ptr<MyClass>(this) — creates independent ownership
};
```

---

### 4.3 RAII (Resource Acquisition Is Initialization)

RAII is the foundational C++ resource management idiom. Resources are acquired in constructors and released in destructors. Stack unwinding guarantees destructor calls even when exceptions are thrown.

```cpp
// RAII lock guard
class LockGuard {
    std::mutex& mtx_;
public:
    explicit LockGuard(std::mutex& m) : mtx_(m) { mtx_.lock(); }
    ~LockGuard() { mtx_.unlock(); }
    LockGuard(const LockGuard&) = delete;            // non-copyable
    LockGuard& operator=(const LockGuard&) = delete; // non-assignable
};

// RAII file handle
class FileHandle {
    FILE* fp_;
public:
    explicit FileHandle(const char* path, const char* mode)
        : fp_(fopen(path, mode)) {
        if (!fp_) throw std::runtime_error("Cannot open file");
    }
    ~FileHandle() { if (fp_) fclose(fp_); }
    FileHandle(const FileHandle&) = delete;
    FILE* get() const { return fp_; }
};

// Use RAII everywhere:
// std::lock_guard, std::unique_lock  — mutexes
// std::unique_ptr, std::shared_ptr   — heap memory
// std::fstream                        — files
// std::vector, std::string            — dynamic buffers
// std::jthread (C++20)               — threads
```

---

### 4.4 Allocators and Memory Arenas

```cpp
// Custom allocator interface
template<typename T>
class PoolAllocator {
public:
    using value_type = T;

    PoolAllocator() = default;

    template<typename U>
    PoolAllocator(const PoolAllocator<U>&) noexcept {}

    T* allocate(std::size_t n) {
        return static_cast<T*>(pool_.alloc(n * sizeof(T)));
    }
    void deallocate(T* p, std::size_t n) {
        pool_.free(p, n * sizeof(T));
    }
private:
    static MemoryPool pool_;
};

// std::pmr (C++17) polymorphic memory resources
#include <memory_resource>
std::pmr::monotonic_buffer_resource arena(1024 * 1024); // 1 MB buffer
std::pmr::polymorphic_allocator<int> alloc(&arena);
std::pmr::vector<int> v(&arena);  // vector backed by arena
// arena.release() frees everything at once; O(1) deallocation
```

---

## PART 5: EXCEPTIONS AND ERROR HANDLING

---

### 5.1 Exceptions

```cpp
// Throwing
throw std::runtime_error("something went wrong");
throw 42;                          // can throw anything (bad practice)
throw;                             // rethrow current exception (inside catch)

// Catching
try {
    mightThrow();
} catch (const std::invalid_argument& e) {  // derived first
    std::cerr << "Invalid arg: " << e.what() << "\n";
} catch (const std::runtime_error& e) {
    std::cerr << "Runtime: " << e.what() << "\n";
} catch (const std::exception& e) {         // base class catches the rest
    std::cerr << "Exception: " << e.what() << "\n";
} catch (...) {                             // catch all; cannot inspect
    std::cerr << "Unknown exception\n";
    throw;                                  // rethrow
}

// Exception hierarchy (std)
// std::exception
//   std::logic_error
//     std::invalid_argument
//     std::domain_error
//     std::length_error
//     std::out_of_range
//   std::runtime_error
//     std::overflow_error
//     std::underflow_error
//     std::range_error
//     std::system_error

// Custom exception
class AppError : public std::runtime_error {
    int code_;
public:
    AppError(int code, const std::string& msg)
        : std::runtime_error(msg), code_(code) {}
    int code() const { return code_; }
};

// noexcept: function guarantees no exceptions propagate
void f() noexcept { /* cannot propagate exceptions */ }
// If an exception escapes a noexcept function, std::terminate is called

// noexcept is checked at compile time for move semantics:
// std::vector uses move if noexcept, otherwise copy (for strong exception guarantee)
// Always mark moves and swaps noexcept

// Exception safety guarantees:
// No-throw guarantee: operation never throws (noexcept)
// Strong guarantee: operation either succeeds or has no effect (transaction-like)
// Basic guarantee: no resources leaked, invariants maintained, but state may change
// No guarantee: anything goes (avoid)

// std::terminate and std::unexpected
std::set_terminate([]() { std::cerr << "Terminate called\n"; std::abort(); });
```

---

### 5.2 Error Handling Alternatives

```cpp
// std::optional (C++17): value or nothing
#include <optional>
std::optional<int> findUser(int id) {
    if (id < 0) return std::nullopt;
    return id * 2;  // implicit conversion to optional
}
auto result = findUser(5);
if (result) std::cout << *result;    // dereference
result.value();                      // throws std::bad_optional_access if empty
result.value_or(-1);                 // default if empty
result.has_value();

// std::variant (C++17): type-safe union
#include <variant>
std::variant<int, std::string, double> v;
v = 42;
v = "hello";
std::get<std::string>(v);            // throws std::bad_variant_access if wrong type
std::get_if<std::string>(&v);        // returns pointer; null if wrong type
std::holds_alternative<int>(v);      // bool check
std::visit([](auto&& val) { std::cout << val; }, v); // visitor pattern

// std::expected (C++23): value or error
#include <expected>
std::expected<int, std::string> parse(const std::string& s) {
    try { return std::stoi(s); }
    catch (...) { return std::unexpected("not a number"); }
}
auto r = parse("42");
if (r) std::cout << *r;
else std::cerr << r.error();
```

---

## PART 6: THE STANDARD LIBRARY

---

### 6.1 Containers

```cpp
// Sequential containers
std::vector<T>        // dynamic array; O(1) amortized push_back; O(1) random access
std::deque<T>         // double-ended queue; O(1) push_front/push_back; O(1) random access
std::list<T>          // doubly linked list; O(1) insert/erase anywhere (with iterator)
std::forward_list<T>  // singly linked list; lower overhead than list
std::array<T, N>      // fixed-size array; stack allocated
std::string           // specialized for characters

// Associative containers (ordered, backed by red-black tree)
std::map<K, V>         // K->V mapping; keys unique; O(log n) all operations
std::multimap<K, V>    // allows duplicate keys
std::set<K>            // unique keys only
std::multiset<K>       // allows duplicates

// Unordered associative (hash table)
std::unordered_map<K, V>     // average O(1) lookup; requires hash<K>
std::unordered_multimap<K,V>
std::unordered_set<K>
std::unordered_multiset<K>

// Container adapters
std::stack<T>               // LIFO; backed by deque by default
std::queue<T>               // FIFO; backed by deque
std::priority_queue<T>      // max-heap by default

// Common interface
c.size(); c.empty(); c.clear();
c.begin(); c.end(); c.cbegin(); c.cend();   // const iterators
c.rbegin(); c.rend();                        // reverse iterators
c.insert(pos, val); c.erase(pos);
c.swap(other);

// map/unordered_map specifics
m[key] = val;                // insert or assign
m.at(key);                   // throws if not found
m.find(key);                 // returns iterator; end() if not found
m.count(key);                // 0 or 1 for map; 0+ for multimap
m.contains(key);             // C++20; bool
auto [it, inserted] = m.emplace(key, val);  // structured binding (C++17)
m.try_emplace(key, args...); // only inserts if key not present

// Custom hash for unordered_map
struct PairHash {
    size_t operator()(const std::pair<int,int>& p) const {
        return std::hash<int>{}(p.first) ^ (std::hash<int>{}(p.second) << 32);
    }
};
std::unordered_map<std::pair<int,int>, int, PairHash> grid;
```

---

### 6.2 Iterators

```cpp
// Iterator categories (hierarchy of capabilities):
// Input        — read-only, single-pass (istream_iterator)
// Output       — write-only, single-pass (ostream_iterator, back_inserter)
// Forward      — read/write, multi-pass, single direction (forward_list)
// Bidirectional — + can decrement (list, map, set)
// Random Access — + arithmetic, subscript, comparison (vector, deque, array)
// Contiguous    — C++17; random access + contiguous memory guarantee (vector, array)

std::vector<int> v = {1,2,3,4,5};
auto it = v.begin();
*it;         // dereference
++it;        // advance
it += 3;     // random access arithmetic
it - v.begin(); // distance
v.end() - v.begin(); // == v.size()

// Iterator adapters
std::back_inserter(v);   // calls push_back on assignment
std::front_inserter(v);  // calls push_front (not for vector)
std::inserter(v, pos);   // calls insert at pos

// istream_iterator / ostream_iterator
std::istream_iterator<int> in(std::cin), eof;
std::ostream_iterator<int> out(std::cout, " ");
std::copy(in, eof, std::back_inserter(v));
std::copy(v.begin(), v.end(), out);

// std::advance, std::distance, std::next, std::prev
std::advance(it, 3);               // advance by 3
std::distance(v.begin(), v.end()); // count elements
std::next(it, 2);                  // iterator 2 ahead (no mutation)
std::prev(it, 1);                  // iterator 1 behind
```

---

### 6.3 Algorithms

```cpp
#include <algorithm>
#include <numeric>

std::vector<int> v = {3, 1, 4, 1, 5, 9, 2, 6};

// Non-modifying
std::find(v.begin(), v.end(), 5);           // iterator to first 5
std::find_if(v.begin(), v.end(), [](int x){ return x > 4; });
std::count(v.begin(), v.end(), 1);          // count of 1s
std::count_if(v.begin(), v.end(), [](int x){ return x % 2 == 0; });
std::all_of(v.begin(), v.end(), [](int x){ return x > 0; }); // true
std::any_of(v.begin(), v.end(), [](int x){ return x > 8; }); // true
std::none_of(v.begin(), v.end(), [](int x){ return x < 0; }); // true
std::for_each(v.begin(), v.end(), [](int& x){ x *= 2; });

// Modifying
std::fill(v.begin(), v.end(), 0);
std::fill_n(v.begin(), 3, 42);
std::generate(v.begin(), v.end(), []{ return rand() % 100; });
std::transform(v.begin(), v.end(), v.begin(), [](int x){ return x * x; });
std::replace(v.begin(), v.end(), 1, 99);
std::replace_if(v.begin(), v.end(), [](int x){ return x < 0; }, 0);
std::copy(src.begin(), src.end(), dst.begin());
std::copy_if(src.begin(), src.end(), std::back_inserter(dst), pred);
std::move(src.begin(), src.end(), dst.begin());
std::reverse(v.begin(), v.end());
std::rotate(v.begin(), v.begin() + 2, v.end()); // rotate left by 2
std::unique(v.begin(), v.end());     // remove consecutive duplicates; must sort first
v.erase(std::unique(v.begin(), v.end()), v.end()); // erase-remove idiom
std::remove(v.begin(), v.end(), 5);  // does not resize; use with erase
v.erase(std::remove(v.begin(), v.end(), 5), v.end());

// Sorting
std::sort(v.begin(), v.end());                   // introsort; O(n log n)
std::sort(v.begin(), v.end(), std::greater<int>()); // descending
std::stable_sort(v.begin(), v.end());            // preserves equal element order
std::partial_sort(v.begin(), v.begin()+k, v.end()); // top k elements sorted
std::nth_element(v.begin(), v.begin()+k, v.end()); // k-th element correct; rest partitioned
std::is_sorted(v.begin(), v.end());

// Binary search (on sorted range)
std::binary_search(v.begin(), v.end(), 4);      // bool
std::lower_bound(v.begin(), v.end(), 4);        // first >= 4
std::upper_bound(v.begin(), v.end(), 4);        // first > 4
auto [lo, hi] = std::equal_range(v.begin(), v.end(), 4); // range of 4s

// Numeric
std::accumulate(v.begin(), v.end(), 0);                  // sum
std::accumulate(v.begin(), v.end(), 1, std::multiplies<int>()); // product
std::inner_product(a.begin(), a.end(), b.begin(), 0);    // dot product
std::partial_sum(v.begin(), v.end(), result.begin());    // prefix sums
std::adjacent_difference(v.begin(), v.end(), result.begin()); // differences
std::reduce(v.begin(), v.end());                         // C++17; parallelizable
std::transform_reduce(v.begin(), v.end(), 0, std::plus<>{},
                       [](int x){ return x * x; });      // map-reduce

// Min/max
std::min_element(v.begin(), v.end());
std::max_element(v.begin(), v.end());
auto [minIt, maxIt] = std::minmax_element(v.begin(), v.end());
std::min(3, 5);
std::max({1, 2, 3});   // initializer list version

// Heap operations
std::make_heap(v.begin(), v.end());   // max-heap
std::push_heap(v.begin(), v.end());   // after push_back
std::pop_heap(v.begin(), v.end());    // moves max to end; then pop_back
std::sort_heap(v.begin(), v.end());   // heap sort; destroys heap property

// Set operations (on sorted ranges)
std::set_union(a.begin(),a.end(), b.begin(),b.end(), out);
std::set_intersection(...);
std::set_difference(...);
std::set_symmetric_difference(...);
std::includes(a.begin(),a.end(), b.begin(),b.end()); // is b a subset of a?

// Permutations
std::next_permutation(v.begin(), v.end());
std::prev_permutation(v.begin(), v.end());

// Execution policies (C++17)
#include <execution>
std::sort(std::execution::par, v.begin(), v.end());    // parallel
std::sort(std::execution::par_unseq, v.begin(), v.end()); // parallel + vectorized
std::sort(std::execution::seq, v.begin(), v.end());    // sequential
```

---

### 6.4 Lambdas

```cpp
// Basic syntax: [capture](params) -> return_type { body }
auto add = [](int a, int b) -> int { return a + b; };
auto add2 = [](int a, int b) { return a + b; }; // return type deduced

// Capture modes
int x = 10, y = 20;
auto f1 = [x]()   { return x; };       // capture x by value (copy)
auto f2 = [&x]()  { return ++x; };     // capture x by reference
auto f3 = [=]()   { return x + y; };   // capture all by value
auto f4 = [&]()   { return x + y; };   // capture all by reference
auto f5 = [=, &y](){ return x + y; };  // all by value except y by reference
auto f6 = [x = x*2]() { return x; };   // init capture (C++14): copies x*2

// mutable: allows modifying value-captured variables
auto f7 = [x]() mutable { ++x; return x; }; // x copy is modified, original unchanged

// Generic lambda (C++14)
auto identity = [](auto x) { return x; };
identity(42);
identity("hello");

// Variadic generic lambda
auto printer = [](auto&&... args) { (std::cout << ... << args); };

// Immediately invoked lambda
int result = [](int a, int b){ return a + b; }(3, 4);

// Lambda as template parameter (C++20)
auto sorter = []<typename T>(const T& a, const T& b) { return a < b; };

// Recursive lambda (C++14)
std::function<int(int)> fib = [&fib](int n) -> int {
    return n <= 1 ? n : fib(n-1) + fib(n-2);
};

// Stateless lambda can convert to function pointer
int (*fp)(int,int) = [](int a, int b){ return a + b; };
```

---

### 6.5 I/O Streams

```cpp
#include <iostream>  // cin, cout, cerr, clog
#include <fstream>   // ifstream, ofstream, fstream
#include <sstream>   // istringstream, ostringstream, stringstream

// Console I/O
std::cout << "Hello" << std::endl;   // endl flushes buffer (slow)
std::cout << "Hello\n";              // prefer '\n' (no flush)
std::cout << std::flush;             // explicit flush

int n; std::cin >> n;               // skips whitespace, reads int
std::string line; std::getline(std::cin, line); // reads entire line

// ios_base::sync_with_stdio(false) for fast I/O
std::ios_base::sync_with_stdio(false);
std::cin.tie(nullptr);

// Formatting
#include <iomanip>
std::cout << std::setw(10) << std::setfill('0') << 42;   // "0000000042"
std::cout << std::left << std::setw(10) << "hi";          // "hi        "
std::cout << std::fixed << std::setprecision(4) << 3.14;  // "3.1400"
std::cout << std::scientific << 3.14;                      // "3.14e+00"
std::cout << std::hex << 255;       // "ff"
std::cout << std::oct << 8;         // "10"
std::cout << std::boolalpha << true; // "true"

// File I/O
std::ifstream in("input.txt");
if (!in.is_open()) throw std::runtime_error("Cannot open file");
int val; while (in >> val) { process(val); }
std::string line; while (std::getline(in, line)) { process(line); }

std::ofstream out("output.txt");
out << "Result: " << 42 << "\n";
out.flush();

// Binary I/O
std::fstream f("data.bin", std::ios::binary | std::ios::in | std::ios::out);
f.write(reinterpret_cast<const char*>(&x), sizeof(x));
f.read(reinterpret_cast<char*>(&x), sizeof(x));
f.seekg(0, std::ios::beg);    // seek get position
f.seekp(0, std::ios::end);    // seek put position
f.tellg(); f.tellp();         // current positions

// String streams
std::ostringstream oss;
oss << "Value: " << 42 << ", PI: " << 3.14;
std::string s = oss.str();

std::istringstream iss("10 20 30");
int a, b, c; iss >> a >> b >> c;

// std::format (C++20): printf-like with type safety
#include <format>
std::string s2 = std::format("Hello, {}! You are {} years old.", "Alice", 30);
std::format("{:>10.4f}", 3.14159);  // right-aligned, 4 decimal places
```

---

## PART 7: MULTITHREADING AND CONCURRENCY

---

### 7.1 Thread Basics

```cpp
#include <thread>
#include <iostream>

// Creating threads
std::thread t1([]() { std::cout << "Thread 1\n"; });
std::thread t2([](int n) { std::cout << "Thread " << n << "\n"; }, 2);

void worker(int id, int& result) { result = id * 2; } // reference arg
int r = 0;
std::thread t3(worker, 3, std::ref(r)); // std::ref required for references

// Join and detach
t1.join();     // wait for t1 to finish; required before t1 is destroyed
t2.detach();   // let t2 run independently; no ownership

// If thread is neither joined nor detached, destructor calls std::terminate

// Thread ID
std::thread::id id = t3.get_id();
std::this_thread::get_id();          // ID of current thread
std::this_thread::sleep_for(std::chrono::milliseconds(100));
std::this_thread::sleep_until(tp);   // sleep until time_point
std::this_thread::yield();           // hint to scheduler; useful in busy loops

// Hardware concurrency
unsigned int n = std::thread::hardware_concurrency(); // number of logical cores

// RAII thread (C++20)
#include <stop_token>
std::jthread jt([](std::stop_token st) {
    while (!st.stop_requested()) {
        doWork();
    }
});
jt.request_stop(); // signal to stop
// jt joins automatically on destruction
```

---

### 7.2 Mutexes and Locking

```cpp
#include <mutex>
#include <shared_mutex>

std::mutex mtx;

// Basic lock
mtx.lock();
// critical section
mtx.unlock();

// RAII lock_guard (simple, non-transferable)
{
    std::lock_guard<std::mutex> lock(mtx);
    // critical section; unlocks when lock goes out of scope
}

// unique_lock (flexible: deferred, timed, transferable)
std::unique_lock<std::mutex> lock(mtx);                     // immediate lock
std::unique_lock<std::mutex> lock(mtx, std::defer_lock);    // no immediate lock
std::unique_lock<std::mutex> lock(mtx, std::try_to_lock);   // try without blocking
std::unique_lock<std::mutex> lock(mtx, std::adopt_lock);    // assume already locked

lock.lock();
lock.unlock();
lock.try_lock();
lock.owns_lock();

// Avoid deadlock: always acquire locks in the same order
// Or use std::lock for multiple locks atomically
std::lock(mtx1, mtx2);  // deadlock-free; locks both or neither
std::lock_guard<std::mutex> lg1(mtx1, std::adopt_lock);
std::lock_guard<std::mutex> lg2(mtx2, std::adopt_lock);

// scoped_lock (C++17): RAII for multiple mutexes; cleaner
std::scoped_lock sl(mtx1, mtx2); // variadic; deadlock-free

// Timed mutex
std::timed_mutex tmtx;
if (tmtx.try_lock_for(std::chrono::milliseconds(100))) {
    // got the lock
    tmtx.unlock();
}

// Shared mutex (reader-writer lock)
std::shared_mutex rwmtx;

// Writers get exclusive access
std::unique_lock<std::shared_mutex> write_lock(rwmtx);

// Readers share access (multiple concurrent readers allowed)
std::shared_lock<std::shared_mutex> read_lock(rwmtx);

// Recursive mutex (same thread can lock multiple times)
std::recursive_mutex rmtx;
rmtx.lock();
rmtx.lock(); // OK; must unlock same number of times
```

---

### 7.3 Condition Variables

```cpp
#include <condition_variable>
#include <queue>
#include <mutex>

// Producer-Consumer with condition_variable
template<typename T>
class ThreadSafeQueue {
    std::queue<T> queue_;
    mutable std::mutex mtx_;
    std::condition_variable cv_;

public:
    void push(T value) {
        {
            std::lock_guard<std::mutex> lock(mtx_);
            queue_.push(std::move(value));
        }
        cv_.notify_one();   // wake one waiting consumer
        // cv_.notify_all() wakes all waiters
    }

    T pop() {
        std::unique_lock<std::mutex> lock(mtx_);
        cv_.wait(lock, [this]{ return !queue_.empty(); });
        // wait: atomically unlocks and sleeps until notified
        // predicate prevents spurious wakeups (ALWAYS use predicate form)
        T val = std::move(queue_.front());
        queue_.pop();
        return val;
    }

    bool try_pop(T& val) {
        std::lock_guard<std::mutex> lock(mtx_);
        if (queue_.empty()) return false;
        val = std::move(queue_.front());
        queue_.pop();
        return true;
    }
};

// condition_variable_any (works with any lockable)
std::condition_variable_any cva;
std::shared_mutex smtx;
std::shared_lock<std::shared_mutex> sl(smtx);
cva.wait(sl, []{ return ready; });
```

---

### 7.4 Atomics

```cpp
#include <atomic>

// Basic atomic types
std::atomic<int>    ai{0};
std::atomic<bool>   ab{false};
std::atomic<float>  af{0.0f};  // not all platforms support lock-free for float
std::atomic<void*>  ap{nullptr};

// Operations
ai.store(42);                    // atomic write
int val = ai.load();             // atomic read
int old = ai.exchange(100);      // atomic swap; returns old value
int expected = 42;
bool ok = ai.compare_exchange_strong(expected, 100);
// compare_exchange_strong: if (ai == expected) { ai = 100; return true; }
//                          else { expected = ai; return false; }
// compare_exchange_weak: may spuriously fail; use in loops (faster on some platforms)

ai.fetch_add(1);   // atomic ai += 1; returns old value
ai.fetch_sub(1);
ai.fetch_and(mask);
ai.fetch_or(bits);
ai.fetch_xor(bits);

++ai; --ai; ai += 5; // also atomic for integral types

// is_lock_free: true if hardware supports the operation natively
ai.is_lock_free();  // true on most platforms for int
std::atomic<LargeStruct> big; big.is_lock_free(); // likely false

// Memory ordering (from weakest to strongest):
// relaxed: no ordering constraints; just atomicity
// consume: data-dependency ordering (mostly deprecated in practice)
// acquire: no reads/writes in current thread can be reordered before this load
// release: no reads/writes in current thread can be reordered after this store
// acq_rel: acquire + release (for read-modify-write ops)
// seq_cst: sequential consistency; total global order; default; most expensive

ai.store(1, std::memory_order_release);
int v = ai.load(std::memory_order_acquire);
ai.fetch_add(1, std::memory_order_relaxed);

// Common pattern: flag-based synchronization
std::atomic<bool> ready{false};
std::atomic<int> data{0};

// Producer thread:
data.store(42, std::memory_order_relaxed);
ready.store(true, std::memory_order_release);  // release: data write is visible before this

// Consumer thread:
while (!ready.load(std::memory_order_acquire)); // acquire: subsequent reads see data write
int v2 = data.load(std::memory_order_relaxed);  // guaranteed to see 42

// std::atomic_ref (C++20): apply atomic operations to non-atomic objects
int ordinary = 0;
std::atomic_ref<int> ref(ordinary);
ref.fetch_add(1);

// Fences
std::atomic_thread_fence(std::memory_order_release);
std::atomic_thread_fence(std::memory_order_acquire);
std::atomic_signal_fence(std::memory_order_seq_cst); // only between thread and signal handler
```

---

### 7.5 Futures and Async

```cpp
#include <future>
#include <async>

// std::async: run a function asynchronously
std::future<int> fut = std::async(std::launch::async, []{ return 42; });
// launch::async: new thread always
// launch::deferred: lazy evaluation on .get()
// launch::async | launch::deferred: implementation decides

int result = fut.get();  // block until result is ready; can only call once
fut.wait();              // block without retrieving
fut.wait_for(std::chrono::seconds(1));  // timed wait; returns future_status
fut.valid();             // false after get() or default-constructed

// std::promise: manual future/promise pair
std::promise<int> prom;
std::future<int> f = prom.get_future();

std::thread([&prom]{
    // compute result
    prom.set_value(42);
    // prom.set_exception(std::current_exception()); // propagate exceptions
}).detach();

int val = f.get(); // blocks until set_value called

// std::packaged_task: wraps callable, ties result to a future
std::packaged_task<int(int, int)> task([](int a, int b){ return a + b; });
std::future<int> tf = task.get_future();
std::thread(std::move(task), 3, 4).detach();
std::cout << tf.get(); // 7

// std::shared_future: multiple get() calls; multiple threads can wait
std::shared_future<int> sf = fut.share();  // fut is now invalid
// Multiple threads can all call sf.get() simultaneously

// Structured concurrency with futures
auto f1 = std::async(task1);
auto f2 = std::async(task2);
auto f3 = std::async(task3);
// All three run in parallel; collect results:
auto r1 = f1.get();
auto r2 = f2.get();
auto r3 = f3.get();
```

---

### 7.6 Thread Pool

```cpp
#include <vector>
#include <thread>
#include <queue>
#include <functional>
#include <future>
#include <mutex>
#include <condition_variable>
#include <atomic>

class ThreadPool {
public:
    explicit ThreadPool(size_t numThreads) : stop_(false) {
        for (size_t i = 0; i < numThreads; ++i) {
            workers_.emplace_back([this] {
                while (true) {
                    std::function<void()> task;
                    {
                        std::unique_lock<std::mutex> lock(mutex_);
                        cv_.wait(lock, [this] { return stop_ || !tasks_.empty(); });
                        if (stop_ && tasks_.empty()) return;
                        task = std::move(tasks_.front());
                        tasks_.pop();
                    }
                    task();
                }
            });
        }
    }

    template<typename F, typename... Args>
    auto enqueue(F&& f, Args&&... args)
        -> std::future<std::invoke_result_t<F, Args...>>
    {
        using ReturnType = std::invoke_result_t<F, Args...>;
        auto task = std::make_shared<std::packaged_task<ReturnType()>>(
            std::bind(std::forward<F>(f), std::forward<Args>(args)...)
        );
        std::future<ReturnType> fut = task->get_future();
        {
            std::lock_guard<std::mutex> lock(mutex_);
            if (stop_) throw std::runtime_error("ThreadPool stopped");
            tasks_.emplace([task]{ (*task)(); });
        }
        cv_.notify_one();
        return fut;
    }

    ~ThreadPool() {
        {
            std::lock_guard<std::mutex> lock(mutex_);
            stop_ = true;
        }
        cv_.notify_all();
        for (auto& t : workers_) t.join();
    }

private:
    std::vector<std::thread> workers_;
    std::queue<std::function<void()>> tasks_;
    std::mutex mutex_;
    std::condition_variable cv_;
    bool stop_;
};

// Usage
ThreadPool pool(4);
auto f1 = pool.enqueue([](int x) { return x * x; }, 5);
auto f2 = pool.enqueue([]{ return std::string("hello"); });
std::cout << f1.get() << "\n"; // 25
std::cout << f2.get() << "\n"; // hello
```

---

### 7.7 Memory Model and Data Races

A data race occurs when two threads access the same memory location concurrently, at least one access is a write, and they are not synchronized. Data races are undefined behavior in C++.

```cpp
// UB: data race
int x = 0;
std::thread t1([&]{ x = 1; });
std::thread t2([&]{ x = 2; });
t1.join(); t2.join();
// x is indeterminate; behavior is undefined

// Fix 1: mutex
std::mutex m;
std::thread t1([&]{ std::lock_guard<std::mutex> l(m); x = 1; });

// Fix 2: atomic
std::atomic<int> x{0};
std::thread t1([&]{ x.store(1, std::memory_order_relaxed); });

// Happens-before relationships:
// - sequenced-before (same thread; program order)
// - synchronizes-with (release store synchronizes with acquire load of same atomic)
// - inter-thread happens-before (transitivity of the above)

// Common mistakes:
// 1. Double-checked locking without atomics (classic pre-C++11 bug)
// Singleton* Singleton::getInstance() {
//     if (!instance_) { lock; if (!instance_) instance_ = new Singleton; }
//     return instance_; // UB: first check has no synchronization
// }
// Fix: use atomic with acquire/release, or just std::call_once

std::once_flag flag;
std::shared_ptr<Singleton> instance;
void createInstance() {
    std::call_once(flag, []{
        instance = std::make_shared<Singleton>();
    });
}

// 2. Benign races (illusion): no such thing in C++; all races are UB

// Thread sanitizer: -fsanitize=thread (gcc/clang) detects races at runtime
```

---

## PART 8: ADVANCED C++ FEATURES

---

### 8.1 Type Deduction

```cpp
// auto: deduce from initializer
auto i = 42;              // int
auto d = 3.14;            // double
auto& r = i;              // int&
const auto& cr = i;       // const int&
auto* p = &i;             // int*
auto f = [](){ return 1; }; // lambda type (unique, unnamed)

// auto drops top-level cv-qualifiers and references:
const int ci = 42;
auto a = ci;          // int (not const int)
int& ri = i;
auto b = ri;          // int (not int&)

// decltype: deduce type of expression without evaluating it
int x = 42;
decltype(x) y = 0;    // int
decltype(x + 0.0) z;  // double
decltype((x)) w = x;  // int& (parentheses make it lvalue expression)

// decltype(auto): like auto but preserves references and cv-qualifiers
decltype(auto) f() { int x = 0; return x; }    // returns int
decltype(auto) g() { int x = 0; return (x); }  // returns int& (dangerous!)

// Template argument deduction (TAD)
template<typename T> void f(T x);    // T deduced from x
template<typename T> void f(T& x);   // T deduced from x (strips &)
template<typename T> void f(T* x);   // T deduced from *x
template<typename T> void f(const T& x); // T deduced, const&

// Class template argument deduction (CTAD, C++17)
std::pair p(1, 3.14);           // deduced as std::pair<int, double>
std::vector v = {1, 2, 3};      // deduced as std::vector<int>
std::lock_guard lg(mtx);        // deduced as std::lock_guard<std::mutex>

// Deduction guides
template<typename T, typename U>
struct MyPair { T first; U second; };
// deduction guide:
template<typename T, typename U>
MyPair(T, U) -> MyPair<T, U>;
MyPair p2(1, 3.14); // works
```

---

### 8.2 constexpr and Compile-Time Programming

```cpp
// constexpr variable: compile-time constant
constexpr int SIZE = 1024;
constexpr double PI = 3.14159265358979323846;

// constexpr function (C++11): computable at compile time if args are constexpr
constexpr int factorial(int n) {
    return n <= 1 ? 1 : n * factorial(n - 1);
}
static_assert(factorial(10) == 3628800); // compile-time check
int arr[factorial(5)];                   // VLA-free: size is compile-time

// C++14: constexpr functions can have loops, local variables, multiple returns
constexpr int fibonacci(int n) {
    if (n <= 1) return n;
    int a = 0, b = 1;
    for (int i = 2; i <= n; ++i) { int t = a + b; a = b; b = t; }
    return b;
}

// C++17: constexpr lambdas
auto square = [](int x) constexpr { return x * x; };

// C++20: constexpr virtual functions, try/catch, dynamic_cast
// C++20: consteval: ONLY computable at compile time
consteval int alwaysCompileTime(int n) { return n * 2; }
// alwaysCompileTime(x); // error if x is not compile-time constant

// C++20: constinit: variable must be statically initialized (no dynamic init)
constinit int global = factorial(5);  // must be constant-initialized

// std::integral_constant and bool_constant
std::integral_constant<int, 42> ic;
ic.value;          // 42
using TrueType  = std::bool_constant<true>;
using FalseType = std::bool_constant<false>;

// constexpr if (C++17): compile-time branch, else branch not instantiated
template<typename T>
std::string typeToString() {
    if constexpr (std::is_integral_v<T>) return "integral";
    else if constexpr (std::is_floating_point_v<T>) return "floating";
    else return "other";
}
```

---

### 8.3 Structured Bindings (C++17)

```cpp
// Decompose aggregates, pairs, tuples, and custom types
std::pair<int, double> p{1, 3.14};
auto [first, second] = p;      // copies
auto& [f, s] = p;              // references

std::tuple<int, double, std::string> t{1, 2.0, "hello"};
auto [a, b, c] = t;

struct Point { int x, y, z; };
Point pt{1, 2, 3};
auto [x, y, z] = pt;

// Particularly useful in loops
std::map<std::string, int> scores;
for (auto& [name, score] : scores) {
    std::cout << name << ": " << score << "\n";
}

// With if/while init
if (auto [it, success] = mymap.emplace("key", 42); success) {
    std::cout << "Inserted: " << it->second << "\n";
}

// Custom structured bindings via tuple_size, tuple_element, get
// (Needed for non-aggregate, non-pair/tuple types)
class MyPair {
    int a_, b_;
public:
    MyPair(int a, int b) : a_(a), b_(b) {}
    template<int N>
    int get() const { if constexpr (N == 0) return a_; else return b_; }
};
namespace std {
    template<> struct tuple_size<MyPair> : integral_constant<size_t, 2> {};
    template<size_t N> struct tuple_element<N, MyPair> { using type = int; };
}
auto [x2, y2] = MyPair{1, 2};
```

---

### 8.4 Ranges (C++20)

```cpp
#include <ranges>
#include <algorithm>

std::vector<int> v = {1,2,3,4,5,6,7,8,9,10};

// Range adaptors compose lazily (no intermediate containers)
auto even_squares = v
    | std::views::filter([](int x){ return x % 2 == 0; })
    | std::views::transform([](int x){ return x * x; });

for (int x : even_squares) std::cout << x << " "; // 4 16 36 64 100

// Common views
std::views::iota(0, 10);               // lazy range [0, 10)
std::views::iota(0);                   // infinite range; must pipe with take
std::views::take(5);                   // first N elements
std::views::drop(3);                   // skip first N
std::views::reverse;                   // reverse iteration
std::views::keys;                      // keys of map
std::views::values;                    // values of map
std::views::elements<1>;               // N-th element of tuple/pair
std::views::zip(v1, v2);               // pairs from two ranges (C++23)
std::views::enumerate(v);              // (index, value) pairs (C++23)
std::views::split(str, delim);         // split by delimiter
std::views::join;                      // flatten range of ranges
std::views::chunk(3);                  // groups of N (C++23)

// Range algorithms (no iterators needed)
std::ranges::sort(v);
std::ranges::sort(v, std::greater{});
std::ranges::find(v, 5);
std::ranges::count_if(v, [](int x){ return x > 5; });
std::ranges::transform(v, v.begin(), [](int x){ return x * 2; });
std::ranges::copy(v, std::ostream_iterator<int>(std::cout, " "));

// Projections (apply function to elements before comparing)
struct Person { std::string name; int age; };
std::vector<Person> people;
std::ranges::sort(people, {}, &Person::age);           // sort by age
std::ranges::max_element(people, {}, &Person::name);   // max by name
```

---

### 8.5 Coroutines (C++20)

Coroutines are functions that can suspend and resume. They are stackless (state saved on heap) and enable cooperative multitasking, generators, and async I/O without threads.

```cpp
#include <coroutine>
#include <optional>

// A Generator coroutine: produces values lazily
template<typename T>
struct Generator {
    struct promise_type {
        T current_value;

        Generator get_return_object() {
            return Generator{std::coroutine_handle<promise_type>::from_promise(*this)};
        }
        std::suspend_always initial_suspend() { return {}; }
        std::suspend_always final_suspend() noexcept { return {}; }
        std::suspend_always yield_value(T value) {
            current_value = value;
            return {};
        }
        void return_void() {}
        void unhandled_exception() { std::terminate(); }
    };

    using Handle = std::coroutine_handle<promise_type>;
    Handle handle;

    explicit Generator(Handle h) : handle(h) {}
    ~Generator() { if (handle) handle.destroy(); }

    bool next() {
        handle.resume();
        return !handle.done();
    }

    T value() const { return handle.promise().current_value; }
};

// Coroutine function
Generator<int> range(int from, int to) {
    for (int i = from; i < to; ++i)
        co_yield i;  // suspend and produce value
}

// Usage
auto gen = range(0, 5);
while (gen.next()) std::cout << gen.value() << " "; // 0 1 2 3 4

// Three coroutine keywords:
// co_yield expr    — suspend and produce a value
// co_return expr   — return and finalize
// co_await expr    — suspend until the awaitable is ready

// Task coroutine (basic async task)
struct Task {
    struct promise_type {
        Task get_return_object() { return {}; }
        std::suspend_never initial_suspend() { return {}; }
        std::suspend_never final_suspend() noexcept { return {}; }
        void return_void() {}
        void unhandled_exception() {}
    };
};

Task asyncWork() {
    co_await std::suspend_always{};  // manually suspend
    // resume from here later
    co_return;
}
```

---

### 8.6 Modules (C++20)

Modules replace the header/translation-unit model with a cleaner, faster compilation model. Module interfaces are parsed once; no header guards needed.

```cpp
// math.ixx (module interface unit)
export module math;                    // declare module

export int add(int a, int b) { return a + b; }  // exported: visible to importers

int internal_helper(int x) { return x; }        // not exported: module-internal

export namespace math_utils {
    double sqrt(double x);
    double pow(double base, double exp);
}

// main.cpp
import math;                           // import the module

int main() {
    int result = add(3, 4);            // works; no #include needed
}

// Module partitions
export module myapp:networking;        // partition declaration
import :networking;                    // import partition within module
```

Modules provide: faster compilation (preprocessed once), no macro leakage, no include-order issues, and stronger encapsulation.

---

### 8.7 Miscellaneous Modern Features

```cpp
// nullptr (C++11): type-safe null pointer constant
int* p = nullptr;          // type: nullptr_t; implicitly converts to any pointer type

// Range-based for with init (C++20)
for (auto& v = getVector(); auto& elem : v) { ... }

// std::span (C++20): non-owning view over contiguous data
#include <span>
void process(std::span<int> s) {
    for (auto& x : s) x *= 2;
    s.size(); s.data(); s.front(); s.back();
    s.subspan(1, 3);
}
int arr[] = {1,2,3,4,5};
process(arr);                       // implicit conversion
process(std::span<int>(arr, 3));    // first 3 elements

// std::string_view (C++17): non-owning string reference
void printUpperCase(std::string_view sv) { ... }
// accepts const char*, std::string, std::string_view without copying

// std::byte (C++17): distinct type for raw bytes
#include <cstddef>
std::byte b{0xFF};
std::to_integer<int>(b);
b & std::byte{0x0F};

// Attributes
[[nodiscard]] int important();      // compiler warns if return value ignored
[[nodiscard("reason")]] int f();    // C++20: with message
[[maybe_unused]] int x = compute(); // suppress unused warning
[[deprecated]] void oldFunc();      // warns on use
[[deprecated("use newFunc")]] void oldFunc2();
[[likely]] if (hot_path) { ... }    // C++20: branch prediction hint
[[unlikely]] if (error_path) { ... }
[[noreturn]] void terminate_app();   // function never returns

// std::launder (C++17): pointer optimization barrier
// Needed when reusing storage for a new object at the same address
new(ptr) T(args);
T* p2 = std::launder(ptr); // tells optimizer: this might point to new object

// Designated initializers (C++20)
struct Config { int width = 800, height = 600; bool fullscreen = false; };
Config c = { .width = 1920, .height = 1080 }; // .fullscreen uses default

// Three-way comparison / spaceship operator (C++20)
struct Point {
    int x, y;
    auto operator<=>(const Point&) const = default; // auto-generates all 6 comparisons
};
Point p1{1,2}, p2{3,4};
p1 < p2; p1 == p2; p1 >= p2; // all work
```

---

## PART 9: DESIGN PATTERNS IN C++

---

### 9.1 Creational Patterns

```cpp
// Singleton (thread-safe in C++11: static local initialization is thread-safe)
class Singleton {
public:
    static Singleton& getInstance() {
        static Singleton instance; // initialized once, destroyed at program exit
        return instance;
    }
    Singleton(const Singleton&) = delete;
    Singleton& operator=(const Singleton&) = delete;
private:
    Singleton() {}
};

// Factory Method
class Shape { public: virtual ~Shape() {} virtual void draw() = 0; };
class Circle : public Shape { public: void draw() override {} };
class Square : public Shape { public: void draw() override {} };

std::unique_ptr<Shape> createShape(const std::string& type) {
    if (type == "circle") return std::make_unique<Circle>();
    if (type == "square") return std::make_unique<Square>();
    throw std::invalid_argument("Unknown shape: " + type);
}

// Abstract Factory
class GUIFactory {
public:
    virtual ~GUIFactory() {}
    virtual std::unique_ptr<Button> createButton() = 0;
    virtual std::unique_ptr<TextBox> createTextBox() = 0;
};
class WindowsFactory : public GUIFactory { ... };
class LinuxFactory   : public GUIFactory { ... };

// Builder
class QueryBuilder {
    std::string table_, conditions_, orderBy_;
    int limit_ = -1;
public:
    QueryBuilder& from(std::string t) { table_ = std::move(t); return *this; }
    QueryBuilder& where(std::string c) { conditions_ = std::move(c); return *this; }
    QueryBuilder& order(std::string o) { orderBy_ = std::move(o); return *this; }
    QueryBuilder& limit(int l) { limit_ = l; return *this; }
    std::string build() const {
        std::string q = "SELECT * FROM " + table_;
        if (!conditions_.empty()) q += " WHERE " + conditions_;
        if (!orderBy_.empty())    q += " ORDER BY " + orderBy_;
        if (limit_ >= 0)          q += " LIMIT " + std::to_string(limit_);
        return q;
    }
};
auto query = QueryBuilder{}.from("users").where("age > 18").limit(10).build();

// Prototype
class Prototype {
public:
    virtual ~Prototype() {}
    virtual std::unique_ptr<Prototype> clone() const = 0;
};
class ConcreteProto : public Prototype {
    int data_;
public:
    ConcreteProto(int d) : data_(d) {}
    std::unique_ptr<Prototype> clone() const override {
        return std::make_unique<ConcreteProto>(*this);
    }
};
```

---

### 9.2 Structural Patterns

```cpp
// Adapter
class OldInterface {
public:
    void specificRequest() {}
};
class NewInterface {
public:
    virtual void request() = 0;
};
class Adapter : public NewInterface {
    OldInterface old_;
public:
    void request() override { old_.specificRequest(); }
};

// Decorator (wraps and extends behavior)
class Coffee {
public:
    virtual ~Coffee() {}
    virtual std::string description() const = 0;
    virtual double cost() const = 0;
};
class SimpleCoffee : public Coffee {
public:
    std::string description() const override { return "Simple coffee"; }
    double cost() const override { return 1.0; }
};
class MilkDecorator : public Coffee {
    std::unique_ptr<Coffee> coffee_;
public:
    MilkDecorator(std::unique_ptr<Coffee> c) : coffee_(std::move(c)) {}
    std::string description() const override { return coffee_->description() + ", milk"; }
    double cost() const override { return coffee_->cost() + 0.25; }
};

// Composite (tree structures)
class FileSystemNode {
public:
    virtual ~FileSystemNode() {}
    virtual void print(int indent = 0) const = 0;
    virtual size_t size() const = 0;
};
class File : public FileSystemNode {
    std::string name_; size_t size_;
public:
    File(std::string n, size_t s) : name_(std::move(n)), size_(s) {}
    void print(int indent) const override { std::cout << std::string(indent,' ') << name_ << "\n"; }
    size_t size() const override { return size_; }
};
class Directory : public FileSystemNode {
    std::string name_;
    std::vector<std::unique_ptr<FileSystemNode>> children_;
public:
    Directory(std::string n) : name_(std::move(n)) {}
    void add(std::unique_ptr<FileSystemNode> node) { children_.push_back(std::move(node)); }
    void print(int indent) const override {
        std::cout << std::string(indent,' ') << name_ << "/\n";
        for (auto& c : children_) c->print(indent + 2);
    }
    size_t size() const override {
        size_t total = 0;
        for (auto& c : children_) total += c->size();
        return total;
    }
};

// CRTP (Curiously Recurring Template Pattern): static polymorphism
template<typename Derived>
class Animal {
public:
    void speak() {
        static_cast<Derived*>(this)->speakImpl(); // no virtual dispatch; fully inlinable
    }
};
class Dog : public Animal<Dog> {
public:
    void speakImpl() { std::cout << "Woof\n"; }
};
class Cat : public Animal<Cat> {
public:
    void speakImpl() { std::cout << "Meow\n"; }
};
// CRTP use cases: mixin behavior, static interface enforcement, operator generation
template<typename Derived>
class Comparable {
public:
    bool operator!=(const Derived& rhs) const { return !(static_cast<const Derived&>(*this) == rhs); }
    bool operator> (const Derived& rhs) const { return rhs < static_cast<const Derived&>(*this); }
    bool operator<=(const Derived& rhs) const { return !(static_cast<const Derived&>(*this) > rhs); }
    bool operator>=(const Derived& rhs) const { return !(static_cast<const Derived&>(*this) < rhs); }
};
```

---

### 9.3 Behavioral Patterns

```cpp
// Observer
class Event {
    std::vector<std::function<void()>> listeners_;
public:
    void subscribe(std::function<void()> f) { listeners_.push_back(std::move(f)); }
    void fire() { for (auto& f : listeners_) f(); }
};

// Strategy
class Sorter {
    std::function<bool(int, int)> comparator_;
public:
    explicit Sorter(std::function<bool(int, int)> cmp) : comparator_(std::move(cmp)) {}
    void sort(std::vector<int>& v) { std::sort(v.begin(), v.end(), comparator_); }
};
Sorter asc([](int a, int b){ return a < b; });
Sorter desc([](int a, int b){ return a > b; });

// Command
class Command { public: virtual ~Command() {} virtual void execute() = 0; virtual void undo() = 0; };
class CommandHistory {
    std::stack<std::unique_ptr<Command>> history_;
public:
    void execute(std::unique_ptr<Command> cmd) { cmd->execute(); history_.push(std::move(cmd)); }
    void undo() { if (!history_.empty()) { history_.top()->undo(); history_.pop(); } }
};

// Visitor (double dispatch without dynamic_cast)
class Circle; class Rectangle;  // forward declarations

class ShapeVisitor {
public:
    virtual ~ShapeVisitor() {}
    virtual void visit(const Circle& c) = 0;
    virtual void visit(const Rectangle& r) = 0;
};

class Shape2 {
public:
    virtual ~Shape2() {}
    virtual void accept(ShapeVisitor& v) const = 0;
};
class Circle : public Shape2 {
public:
    double radius;
    void accept(ShapeVisitor& v) const override { v.visit(*this); }
};
class AreaCalculator : public ShapeVisitor {
public:
    double total = 0;
    void visit(const Circle& c) override { total += M_PI * c.radius * c.radius; }
    void visit(const Rectangle&) override { ... }
};

// State machine
class TrafficLight {
    enum class State { Red, Yellow, Green };
    State state_ = State::Red;
public:
    void next() {
        switch (state_) {
            case State::Red:    state_ = State::Green;  break;
            case State::Green:  state_ = State::Yellow; break;
            case State::Yellow: state_ = State::Red;    break;
        }
    }
};
```

---

## PART 10: PERFORMANCE AND SYSTEMS PROGRAMMING

---

### 10.1 Cache and Memory Performance

The memory hierarchy: registers (< 1 ns) → L1 cache (1-4 ns, 32-64 KB) → L2 (4-12 ns, 256 KB-1 MB) → L3 (12-50 ns, 4-32 MB) → DRAM (50-100 ns) → SSD (100 µs) → HDD (10 ms).

Cache lines are typically 64 bytes. Any access brings in an entire cache line.

```cpp
// Cache-friendly: access memory sequentially (row-major for arrays)
// Cache-unfriendly: random access, pointer chasing, column-major

// Bad: column-major traversal
for (int j = 0; j < N; ++j)
    for (int i = 0; i < N; ++i)
        sum += matrix[i][j]; // each access is N*4 bytes apart; cache misses

// Good: row-major traversal
for (int i = 0; i < N; ++i)
    for (int j = 0; j < N; ++j)
        sum += matrix[i][j]; // sequential; fully cache-friendly

// Data Oriented Design (DOD): Structure of Arrays (SoA) vs Array of Structures (AoS)
// AoS (Object-Oriented, often cache-unfriendly for bulk processing)
struct Particle { float x, y, z, vx, vy, vz, mass; };
std::vector<Particle> particles; // mass interleaved; wasted loads when only processing positions

// SoA (Data-Oriented, cache-friendly for SIMD and bulk transforms)
struct Particles {
    std::vector<float> x, y, z;     // all positions contiguous
    std::vector<float> vx, vy, vz;  // all velocities contiguous
    std::vector<float> mass;
};
// When updating positions: only load x, y, z, vx, vy, vz; mass not touched

// False sharing: two threads writing to different variables on same cache line
struct Bad {
    std::atomic<int> counter1; // same cache line
    std::atomic<int> counter2; // contention: updating counter1 invalidates counter2's cache
};
struct Good {
    alignas(64) std::atomic<int> counter1; // on its own cache line
    alignas(64) std::atomic<int> counter2;
};
```

---

### 10.2 Alignment, Padding, and Bit Fields

```cpp
// Struct padding: compiler inserts padding for alignment
struct Padded {
    char a;     // 1 byte
    // 3 bytes padding
    int b;      // 4 bytes; must be 4-byte aligned
    char c;     // 1 byte
    // 3 bytes padding (to make total size multiple of 4)
};
// sizeof(Padded) == 12

struct Packed {
    int b;      // 4 bytes
    char a;     // 1 byte
    char c;     // 1 byte
    // 2 bytes padding
};
// sizeof(Packed) == 8 (reordering fields by decreasing size minimizes padding)

// #pragma pack (non-standard but widely supported)
#pragma pack(push, 1)
struct NetworkHeader { uint16_t port; uint32_t ip; uint8_t flags; };
#pragma pack(pop)
// sizeof(NetworkHeader) == 7; no padding

// alignas specifier
alignas(64) char cache_line_buffer[64]; // aligned to 64 bytes
alignas(16) float simd_data[4];         // for SSE alignment

// Bit fields
struct Flags {
    uint32_t isAlive    : 1;  // 1 bit
    uint32_t isVisible  : 1;
    uint32_t layer      : 4;  // 0-15
    uint32_t type       : 8;  // 0-255
    uint32_t reserved   : 18;
};
// Layout and size are implementation-defined for bit fields

// alignof
alignof(int);        // typically 4
alignof(double);     // typically 8
alignof(std::max_align_t); // maximum fundamental alignment
```

---

### 10.3 Compiler Optimizations and Hints

```cpp
// Optimization levels (GCC/Clang):
// -O0: no optimization (debug)
// -O1: basic optimizations
// -O2: common optimizations (good default)
// -O3: aggressive (vectorization, etc.)
// -Os: optimize for size
// -Ofast: -O3 + unsafe math optimizations

// Link-Time Optimization (LTO): optimize across translation units
// GCC: -flto, Clang: -flto=thin

// Profile-Guided Optimization (PGO):
// 1. Compile with -fprofile-generate
// 2. Run with representative workload
// 3. Recompile with -fprofile-use

// Prevent optimizations for profiling/inspection
__asm__ volatile("" : : "r,m"(x) : "memory"); // GCC/Clang: use x; prevent optimization

// restrict (C hint; not standard C++, but __restrict in GCC/Clang)
void add(float* __restrict a, float* __restrict b, float* __restrict c, int n) {
    for (int i = 0; i < n; ++i) c[i] = a[i] + b[i]; // enables auto-vectorization
    // restrict: a, b, c don't alias; compiler can vectorize freely
}

// __builtin hints (GCC/Clang)
__builtin_expect(x == 0, 1);  // x == 0 is likely (1) or unlikely (0)
// C++20 equivalent: [[likely]] and [[unlikely]]

__builtin_prefetch(ptr, 0, 3); // prefetch ptr; read (0) or write (1); locality 0-3

// volatile: tells compiler not to optimize away reads/writes
// Used for memory-mapped I/O; NOT for threading synchronization
volatile int* mmio_reg = (volatile int*)0x1000;
*mmio_reg = 0xFF; // write will not be optimized away

// std::launder: optimizer barrier for pointer after placement new

// Branch prediction and branchless code
// Instead of: if (a > b) max = a; else max = b;
// Branchless: int max = a + ((b - a) & ((b - a) >> 31)); // bit trick
// Or:         int max = a ^ ((a ^ b) & -(a < b));
```

---

### 10.4 SIMD Intrinsics

```cpp
// Intel SSE/AVX2/FMA intrinsics for 4/8 float operations in parallel
#include <immintrin.h>  // all Intel SIMD

// SSE2: 128-bit registers, 4x float or 2x double
__m128  a = _mm_loadu_ps(float_ptr);      // load 4 floats (unaligned)
__m128  b = _mm_loadu_ps(float_ptr + 4);
__m128  c = _mm_add_ps(a, b);             // add 4 floats
_mm_storeu_ps(result_ptr, c);             // store

// AVX2: 256-bit registers, 8x float or 4x double
__m256  v8 = _mm256_loadu_ps(ptr);
__m256  v8b = _mm256_mul_ps(v8, v8);     // 8 float multiplications
__m256i iv8 = _mm256_loadu_si256((__m256i*)iptr); // 8 int32s
__m256i sum8 = _mm256_add_epi32(iv8, iv8b);

// FMA (Fused Multiply-Add): a*b+c in one instruction, full precision
__m256 result = _mm256_fmadd_ps(a, b, c); // 8 FMAs in one instruction

// Horizontal operations
__m128 v = ...; // [a, b, c, d]
__m128 hadd = _mm_hadd_ps(v, v); // [(a+b), (c+d), (a+b), (c+d)]
// Sum all 4: (repeat hadd and extract)

// Alignment: AVX2 aligned loads are faster when data is 32-byte aligned
// Use alignas(32) for AVX2 buffers
```

---

## PART 11: COMPILATION, LINKING, AND BUILD SYSTEMS

---

### 11.1 Compilation Internals

```
Preprocessing:  g++ -E source.cpp -o source.i    # expand macros, includes
Compilation:    g++ -S source.i   -o source.s    # C++ -> assembly
Assembly:       g++ -c source.s   -o source.o    # assembly -> object file
Linking:        g++ source.o -o executable       # resolve symbols, produce binary

# Common flags
-std=c++20         # C++ standard
-Wall -Wextra      # enable warnings
-Werror            # treat warnings as errors
-g                 # debug info (DWARF)
-O2 / -O3          # optimization
-fsanitize=address,undefined  # AddressSanitizer + UBSan
-fsanitize=thread  # ThreadSanitizer
-fprofile-generate # PGO instrumentation
-march=native      # use all CPU features of current machine
-fvisibility=hidden # default symbol visibility
```

---

### 11.2 ODR, Linkage, and Translation Units

One Definition Rule (ODR): every symbol may be declared multiple times but defined only once across the entire program, with exceptions:
- Inline functions/variables may be defined in multiple translation units if definitions are identical
- Template instantiations are exempt
- Class definitions may appear in multiple TUs if identical (via headers)

```cpp
// Internal linkage (not visible outside TU)
static int x = 0;                  // old style
namespace { int y = 0; }           // preferred: anonymous namespace

// External linkage (visible across TUs)
int global_var = 0;                // default for global variables
extern int another_var;            // declaration; defined elsewhere
extern "C" void c_function();      // C linkage (no name mangling)
extern "C" { void f1(); void f2(); }

// inline variable (C++17): defined in header, single definition across TUs
inline int inline_global = 42;     // safe to put in header

// constexpr implies inline for functions and variables at namespace scope
```

---

### 11.3 CMake Basics

```cmake
cmake_minimum_required(VERSION 3.20)
project(MyProject VERSION 1.0 LANGUAGES CXX)

set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)         # no compiler-specific extensions

# Executable
add_executable(myapp main.cpp src/foo.cpp src/bar.cpp)

# Library
add_library(mylib STATIC src/lib.cpp)            # static (.a)
add_library(mylib SHARED src/lib.cpp)            # shared (.so / .dll)
add_library(mylib INTERFACE)                     # header-only

# Target properties (preferred over global settings)
target_include_directories(myapp PRIVATE include/ PUBLIC include/public)
target_compile_options(myapp PRIVATE -Wall -Wextra -O2)
target_compile_definitions(myapp PRIVATE DEBUG_MODE=1)
target_link_libraries(myapp PRIVATE mylib pthread)

# Find and link external packages
find_package(Threads REQUIRED)
target_link_libraries(myapp PRIVATE Threads::Threads)

find_package(fmt REQUIRED)
target_link_libraries(myapp PRIVATE fmt::fmt)

# Fetch dependencies
include(FetchContent)
FetchContent_Declare(googletest URL https://github.com/google/googletest/archive/...)
FetchContent_MakeAvailable(googletest)
target_link_libraries(tests PRIVATE gtest_main)

# Install rules
install(TARGETS myapp DESTINATION bin)
install(FILES include/mylib.h DESTINATION include)

# Testing
enable_testing()
add_test(NAME mytest COMMAND tests)
```

---

## PART 12: TESTING, DEBUGGING, AND TOOLING

---

### 12.1 Unit Testing with Google Test

```cpp
#include <gtest/gtest.h>

// Basic test
TEST(FactorialTest, HandlesZero) {
    EXPECT_EQ(factorial(0), 1);
}
TEST(FactorialTest, HandlesPositive) {
    EXPECT_EQ(factorial(5), 120);
    EXPECT_EQ(factorial(10), 3628800);
}

// ASSERT vs EXPECT:
// ASSERT_* aborts the test on failure
// EXPECT_* continues the test on failure

EXPECT_EQ(a, b);  ASSERT_EQ(a, b);
EXPECT_NE(a, b);  EXPECT_LT(a, b);  EXPECT_LE(a, b);
EXPECT_GT(a, b);  EXPECT_GE(a, b);
EXPECT_TRUE(cond); EXPECT_FALSE(cond);
EXPECT_STREQ(s1, s2);    // C strings
EXPECT_THROW(expr, ExceptionType);
EXPECT_NO_THROW(expr);
EXPECT_NEAR(a, b, abs_error);  // floating point

// Test fixtures
class StackTest : public ::testing::Test {
protected:
    void SetUp() override { stack_.push(1); stack_.push(2); }
    void TearDown() override {}
    Stack<int> stack_;
};
TEST_F(StackTest, PopReturnsLastPushed) {
    EXPECT_EQ(stack_.top(), 2);
    stack_.pop();
    EXPECT_EQ(stack_.top(), 1);
}

// Parameterized tests
class EvenTest : public ::testing::TestWithParam<int> {};
TEST_P(EvenTest, IsEven) { EXPECT_EQ(GetParam() % 2, 0); }
INSTANTIATE_TEST_SUITE_P(EvenNumbers, EvenTest, ::testing::Values(2, 4, 6, 8));

// Mock objects (Google Mock)
#include <gmock/gmock.h>
class MockDatabase : public Database {
    MOCK_METHOD(bool, connect, (const std::string& url), (override));
    MOCK_METHOD(int,  query,   (const std::string& sql), (override));
};
TEST(ServiceTest, ConnectsToDatabase) {
    MockDatabase db;
    EXPECT_CALL(db, connect("localhost")).Times(1).WillOnce(Return(true));
    Service svc(&db);
    svc.initialize();
}
```

---

### 12.2 Sanitizers and Valgrind

```bash
# AddressSanitizer: detect heap/stack overflows, use-after-free
g++ -fsanitize=address -g program.cpp && ./a.out

# UndefinedBehaviorSanitizer: detect signed overflow, null deref, etc.
g++ -fsanitize=undefined -g program.cpp && ./a.out

# ThreadSanitizer: detect data races
g++ -fsanitize=thread -g program.cpp && ./a.out

# LeakSanitizer: detect memory leaks (included with ASan)
LSAN_OPTIONS=exitcode=1 ./a.out

# MemorySanitizer (Clang only): uninitialized memory reads
clang++ -fsanitize=memory -g program.cpp && ./a.out

# Valgrind (Linux, slower but thorough)
valgrind --leak-check=full ./a.out
valgrind --tool=callgrind ./a.out    # profiling
valgrind --tool=helgrind  ./a.out    # thread errors
valgrind --tool=cachegrind ./a.out   # cache simulation
```

---

### 12.3 Profiling

```bash
# gprof (simple, compile-time instrumentation)
g++ -pg program.cpp && ./a.out && gprof a.out gmon.out | less

# perf (Linux, hardware counters, low overhead)
perf stat ./a.out              # overview: cycles, cache-misses, instructions
perf record ./a.out            # sample call stacks
perf report                    # interactive viewer
perf top                       # live top-like view

# Flame graphs (visual profiling)
perf record -g ./a.out
perf script | stackcollapse-perf.pl | flamegraph.pl > flame.svg

# Compiler Explorer (godbolt.org): see generated assembly
# Quick benchmarking with std::chrono
auto start = std::chrono::high_resolution_clock::now();
compute();
auto end = std::chrono::high_resolution_clock::now();
auto duration = std::chrono::duration_cast<std::chrono::microseconds>(end - start);
std::cout << duration.count() << " µs\n";

# Google Benchmark (micro-benchmarking)
#include <benchmark/benchmark.h>
static void BM_Sort(benchmark::State& state) {
    std::vector<int> v(state.range(0));
    for (auto _ : state) {
        std::iota(v.begin(), v.end(), 0);
        std::shuffle(v.begin(), v.end(), rng);
        std::sort(v.begin(), v.end());
    }
    state.SetComplexityN(state.range(0));
}
BENCHMARK(BM_Sort)->RangeMultiplier(2)->Range(8, 8<<10)->Complexity();
BENCHMARK_MAIN();
```

---

## PART 13: IDIOMS AND BEST PRACTICES

---

### 13.1 Core Guidelines Summary

1. Prefer RAII for all resource management. Never naked `new`/`delete` in application code.
2. Use `unique_ptr` by default; `shared_ptr` only when shared ownership is genuinely needed.
3. Prefer stack allocation over heap allocation; prefer `std::array` over raw arrays.
4. Pass by `const&` for types you don't need to copy; by value when you will copy anyway (lets callers move); by `&&` in constructors/setters where you'll store the value (then `std::move`).
5. Mark move constructors and move assignment `noexcept` so `std::vector` uses moves during reallocation.
6. Use `override` on every overriding virtual function.
7. Use `final` on classes or functions not intended to be further derived/overridden.
8. Keep functions short and focused (single responsibility). Prefer free functions over member functions for symmetric operations.
9. Prefer `const` everywhere applicable. Immutability is correctness-by-default.
10. Use `nullptr` (never `NULL` or `0` for pointers).
11. Never ignore `[[nodiscard]]` return values.
12. Always initialize variables. Use `{}` initialization to prevent narrowing.
13. Use `static_assert` to enforce compile-time invariants.
14. Use `enum class` over plain `enum` for type safety and scoping.
15. Use `std::variant` or `std::optional` rather than sentinel values or out-parameters.
16. Never use `using namespace std;` in header files.
17. Prefer `std::format` (C++20) over `printf` / `sprintf`.
18. Profile before optimizing. Measure, don't guess.

---

### 13.2 Pimpl Idiom (Pointer to Implementation)

Reduces compile-time dependencies and binary compatibility issues. The class definition in the header exposes no implementation details.

```cpp
// Widget.h
class Widget {
public:
    Widget();
    ~Widget();
    Widget(Widget&&) noexcept;
    Widget& operator=(Widget&&) noexcept;
    // Widget(const Widget&) = delete; // or implement deep copy

    void doSomething();

private:
    struct Impl;                        // forward declaration only
    std::unique_ptr<Impl> pImpl_;       // no Impl definition needed in header
};

// Widget.cpp
struct Widget::Impl {
    int data_;
    std::string name_;
    SomeHeavyDependency dep_;           // not visible to Widget.h consumers

    void doSomethingImpl() { ... }
};

Widget::Widget() : pImpl_(std::make_unique<Impl>()) {}
Widget::~Widget() = default;           // must be defined where Impl is complete
Widget::Widget(Widget&&) noexcept = default;
Widget& Widget::operator=(Widget&&) noexcept = default;

void Widget::doSomething() { pImpl_->doSomethingImpl(); }
```

---

### 13.3 Type Erasure

Type erasure hides the concrete type behind a uniform interface without virtual inheritance in user code. `std::function`, `std::any`, and `std::shared_ptr<Base>` are all examples.

```cpp
// Manual type erasure: std::function-like
class AnyCallable {
    struct Concept {
        virtual ~Concept() {}
        virtual void call() = 0;
        virtual std::unique_ptr<Concept> clone() const = 0;
    };

    template<typename F>
    struct Model : Concept {
        F f_;
        Model(F f) : f_(std::move(f)) {}
        void call() override { f_(); }
        std::unique_ptr<Concept> clone() const override {
            return std::make_unique<Model<F>>(f_);
        }
    };

    std::unique_ptr<Concept> impl_;

public:
    template<typename F>
    AnyCallable(F f) : impl_(std::make_unique<Model<F>>(std::move(f))) {}

    AnyCallable(const AnyCallable& other) : impl_(other.impl_->clone()) {}
    AnyCallable(AnyCallable&&) = default;

    void operator()() { impl_->call(); }
};

// std::any: type-safe container for any copyable value
#include <any>
std::any a = 42;
a = std::string("hello");
std::any_cast<std::string>(a);               // throws std::bad_any_cast if wrong
std::any_cast<std::string*>(&a);             // pointer; null if wrong type
a.has_value();
a.type() == typeid(std::string);
```

---

### 13.4 Policy-Based Design

```cpp
// Policies as template parameters (more flexible than virtual inheritance)
template<typename StoragePolicy, typename LoggingPolicy, typename ValidationPolicy>
class DataManager : private StoragePolicy,
                    private LoggingPolicy,
                    private ValidationPolicy
{
public:
    void save(const Data& d) {
        ValidationPolicy::validate(d);      // static dispatch; fully inlinable
        LoggingPolicy::log("saving...");
        StoragePolicy::store(d);
    }
};

// Concrete policies
struct FileStorage   { void store(const Data& d); };
struct DBStorage     { void store(const Data& d); };
struct ConsoleLogger { void log(const std::string& msg); };
struct NoLogger      { void log(const std::string&) {} };

using ProdManager  = DataManager<DBStorage,   ConsoleLogger, StrictValidation>;
using TestManager  = DataManager<FileStorage, NoLogger,      PermissiveValidation>;
```

---

## PART 14: UNDEFINED BEHAVIOR REFERENCE

Undefined behavior (UB) allows the compiler to assume it never happens, enabling optimizations that break code when it does occur. UB can corrupt unrelated code, loop forever, or produce any result. The compiler is not required to warn about UB.

```
Most common C++ UB:
1.  Signed integer overflow (use unsigned or check before operation)
2.  Dereferencing null or dangling pointer
3.  Out-of-bounds array access (including one-past-end write)
4.  Use-after-free / use-after-scope
5.  Data race on non-atomic object
6.  Reading uninitialized variables
7.  Signed left shift of negative value or into sign bit
8.  Division by zero (integer or float)
9.  Returning from non-void function without return (other than main)
10. Stack overflow (deep recursion, large stack allocations)
11. Casting incompatible pointer types and dereferencing (strict aliasing violation)
12. Violating the strict aliasing rule (accessing object via wrong pointer type)
    Exception: char*, unsigned char*, std::byte* may alias anything
13. Breaking the One Definition Rule (multiple different definitions of same entity)
14. Invalid pointer arithmetic (outside array bounds, not even to form the address)
15. Calling a virtual function from a constructor/destructor (called on incomplete type)
16. Double-delete or delete of non-heap pointer
17. std::terminate called (unhandled exception, noexcept violation, pure virtual call)
18. Modifying a string literal
19. Invalid use of moved-from object (valid but unspecified state)
20. Infinite loop without side effects (compiler may eliminate it)
```

Tools to detect UB: `-fsanitize=undefined`, Valgrind, static analyzers (clang-tidy, cppcheck, PVS-Studio).

---

## PART 15: QUICK REFERENCE CHEATSHEETS

---

### Type Traits Quick Reference (C++17 _v aliases)

```cpp
std::is_void_v<T>
std::is_null_pointer_v<T>
std::is_integral_v<T>
std::is_floating_point_v<T>
std::is_array_v<T>
std::is_enum_v<T>
std::is_class_v<T>
std::is_function_v<T>
std::is_pointer_v<T>
std::is_lvalue_reference_v<T>
std::is_rvalue_reference_v<T>
std::is_member_pointer_v<T>
std::is_const_v<T>
std::is_volatile_v<T>
std::is_trivial_v<T>
std::is_trivially_copyable_v<T>
std::is_standard_layout_v<T>
std::is_pod_v<T>          // deprecated in C++20
std::is_empty_v<T>
std::is_polymorphic_v<T>
std::is_abstract_v<T>
std::is_final_v<T>
std::is_constructible_v<T, Args...>
std::is_default_constructible_v<T>
std::is_copy_constructible_v<T>
std::is_move_constructible_v<T>
std::is_assignable_v<T, U>
std::is_copy_assignable_v<T>
std::is_move_assignable_v<T>
std::is_destructible_v<T>
std::is_nothrow_move_constructible_v<T>
std::is_same_v<T, U>
std::is_base_of_v<Base, Derived>
std::is_convertible_v<From, To>
std::is_invocable_v<F, Args...>
std::is_invocable_r_v<R, F, Args...>

// Transformation
std::remove_const_t<T>
std::remove_volatile_t<T>
std::remove_cv_t<T>
std::remove_reference_t<T>
std::remove_cvref_t<T>     // C++20
std::add_const_t<T>
std::add_lvalue_reference_t<T>
std::add_rvalue_reference_t<T>
std::remove_pointer_t<T>
std::add_pointer_t<T>
std::make_signed_t<T>
std::make_unsigned_t<T>
std::decay_t<T>
std::common_type_t<T, U>
std::underlying_type_t<T>  // for enum
std::invoke_result_t<F, Args...>
std::conditional_t<B, T, F>
std::enable_if_t<B, T>
std::void_t<T...>          // C++17
```

---

### Standard Library Headers Reference

```
<algorithm>      sort, find, copy, transform, accumulate...
<array>          std::array
<atomic>         std::atomic, memory_order
<bit>            C++20: std::popcount, std::countl_zero, std::bit_cast
<chrono>         time_point, duration, clocks
<concepts>       C++20: concept, requires
<condition_variable> std::condition_variable
<coroutine>      C++20: co_yield, co_await, coroutine_handle
<cstdint>        int8_t, uint64_t, etc.
<exception>      std::exception, std::terminate
<execution>      C++17: par, par_unseq policies
<filesystem>     C++17: path, directory_iterator, file operations
<format>         C++20: std::format, std::print
<functional>     std::function, bind, hash, less, greater
<future>         std::future, promise, async, packaged_task
<initializer_list>
<iterator>       iterator tags, adapters
<limits>         std::numeric_limits<T>
<map>            std::map, std::multimap
<memory>         unique_ptr, shared_ptr, weak_ptr, allocator
<memory_resource> C++17: pmr allocators
<mutex>          mutex, lock_guard, unique_lock, scoped_lock
<numeric>        accumulate, reduce, iota, partial_sum
<optional>       C++17: std::optional
<queue>          std::queue, std::priority_queue
<ranges>         C++20: views, range adaptors
<regex>          std::regex, match, search
<semaphore>      C++20: std::counting_semaphore, binary_semaphore
<set>            std::set, std::multiset
<shared_mutex>   C++14: std::shared_mutex, shared_lock
<span>           C++20: std::span
<sstream>        istringstream, ostringstream
<stack>          std::stack
<stdexcept>      logic_error, runtime_error, etc.
<stop_token>     C++20: stop_token, stop_source
<string>         std::string
<string_view>    C++17: std::string_view
<thread>         std::thread, jthread
<tuple>          std::tuple, std::tie, std::apply
<type_traits>    all type traits
<unordered_map>  std::unordered_map, unordered_multimap
<unordered_set>  std::unordered_set
<utility>        std::pair, move, forward, swap, exchange
<variant>        C++17: std::variant, std::visit
<vector>         std::vector
```