---
title: "C++ Data Types & Sizes: The Complete Reference"
description: "C++ data types."
---

## 1. FUNDAMENTAL TYPE CATEGORIES

C++ has five categories of fundamental types:

1. Void
2. Boolean
3. Character
4. Integer
5. Floating-point

Everything else (arrays, pointers, references, classes, enums, etc.) is a compound or user-defined type built on top of these.

---

## 2. THE VOID TYPE

`void` has no values and no size. It is used as:
- Return type for functions that return nothing
- Base of generic pointers (`void*`)
- Empty parameter list in C-style declarations

`sizeof(void)` is ill-formed in standard C++. GCC extends it to 1 as a non-standard extension.

---

## 3. BOOLEAN TYPE

`bool` — represents `true` or `false`.

Size: 1 byte (guaranteed at least 1 byte by the standard; `sizeof(bool) == 1` on all mainstream platforms).

Internally stores 0 for `false`, 1 for `true`. Any non-zero integer converts to `true`. Arithmetic on `bool` is legal but discouraged — `true + true == 2` (both promote to `int`).

---

## 4. CHARACTER TYPES

Character types are technically integer types but semantically distinct.

`char`
- Size: exactly 1 byte (the definition of "byte" in C++)
- `sizeof(char) == 1` is guaranteed by the standard — always, everywhere
- May be signed or unsigned — this is implementation-defined
- Use for: narrow strings, byte buffers, legacy code
- Range (if signed): -128 to 127
- Range (if unsigned): 0 to 255

`signed char`
- Explicitly signed, 1 byte
- Range: -128 to 127

`unsigned char`
- Explicitly unsigned, 1 byte
- Range: 0 to 255
- The canonical type for "raw byte" manipulation — it can alias any object type legally (important for memcpy, serialization)

`wchar_t`
- Wide character, implementation-defined size
- Windows: 2 bytes (UTF-16 code unit)
- Linux/macOS: 4 bytes (UTF-32 code point)
- Not portable for Unicode work across platforms

`char8_t` (C++20)
- Unsigned, 1 byte
- Semantically holds a UTF-8 code unit
- Distinct type from `unsigned char` (no implicit conversion)

`char16_t` (C++11)
- Unsigned, exactly 2 bytes (`uint_least16_t` underlying)
- Semantically holds a UTF-16 code unit

`char32_t` (C++11)
- Unsigned, exactly 4 bytes (`uint_least32_t` underlying)
- Semantically holds a UTF-32 code point

---

## 5. INTEGER TYPES

### 5.1 Standard Integer Types

The C++ standard specifies minimum sizes, not exact sizes. The actual sizes depend on the implementation (compiler + platform + ABI).

Standard guarantees (minimum bit widths):

```
short         >= 16 bits
int           >= 16 bits  (in practice always 32 on modern platforms)
long          >= 32 bits
long long     >= 64 bits  (since C++11)
```

The standard also mandates:

```
sizeof(char) <= sizeof(short) <= sizeof(int) <= sizeof(long) <= sizeof(long long)
```

Typical sizes on 64-bit Linux/macOS/Windows (x86-64):

```
Type                  Bits    Bytes    Signed Range                          Unsigned Range
──────────────────────────────────────────────────────────────────────────────────────────
short                 16      2        -32,768 to 32,767                     0 to 65,535
unsigned short        16      2        —                                     0 to 65,535
int                   32      4        -2,147,483,648 to 2,147,483,647       0 to 4,294,967,295
unsigned int          32      4        —                                     0 to 4,294,967,295
long                  32/64   4/8      platform-dependent (see below)
unsigned long         32/64   4/8      —
long long             64      8        -9,223,372,036,854,775,808 to ...+807 0 to 18,446,744,073,709,551,615
unsigned long long    64      8        —                                     0 to 18,446,744,073,709,551,615
```

### 5.2 The `long` Portability Trap

This is one of the most common sources of bugs in cross-platform C++ code.

```
Platform         long size
──────────────────────────
LP64 (Linux/macOS 64-bit)   8 bytes (64-bit)
LLP64 (Windows 64-bit)      4 bytes (32-bit)
ILP32 (32-bit systems)      4 bytes (32-bit)
```

`long` is 4 bytes on 64-bit Windows and 8 bytes on 64-bit Linux/macOS for the same source code. Never use `long` for data that must be exactly 32 or 64 bits. Use `<cstdint>` instead.

### 5.3 Fixed-Width Integer Types (`<cstdint>`, C++11)

These are the types you should use whenever exact widths matter.

Exact-width types (may not exist on all platforms, but exist on all mainstream ones):

```
int8_t        signed,   exactly 8 bits
uint8_t       unsigned, exactly 8 bits
int16_t       signed,   exactly 16 bits
uint16_t      unsigned, exactly 16 bits
int32_t       signed,   exactly 32 bits
uint32_t      unsigned, exactly 32 bits
int64_t       signed,   exactly 64 bits
uint64_t      unsigned, exactly 64 bits
```

Minimum-width types (guaranteed to exist, at least that wide):

```
int_least8_t    uint_least8_t
int_least16_t   uint_least16_t
int_least32_t   uint_least32_t
int_least64_t   uint_least64_t
```

Fastest types of at least N bits (optimized for the platform's native word size):

```
int_fast8_t     uint_fast8_t
int_fast16_t    uint_fast16_t
int_fast32_t    uint_fast32_t
int_fast64_t    uint_fast64_t
```

Note: `int_fast16_t` and `int_fast32_t` are often 64-bit on x86-64 Linux because 64-bit operations are equally fast and the platform's native register width is 64 bits. Do not assume they are exactly N bits.

Maximum-width integer:

```
intmax_t      uintmax_t     (typically 64-bit, but could be 128-bit if the platform supports it)
```

### 5.4 Pointer-Sized Integer Types

```
intptr_t      signed integer large enough to hold a void*
uintptr_t     unsigned integer large enough to hold a void*
ptrdiff_t     signed type of the result of pointer subtraction
size_t        unsigned type of sizeof results and container sizes
```

On 64-bit systems, all four are 8 bytes. On 32-bit systems, 4 bytes.

`size_t` and `ptrdiff_t` are in `<cstddef>`. `intptr_t`/`uintptr_t` are in `<cstdint>`.

Critical rule: use `size_t` for sizes and indices, `ptrdiff_t` for pointer differences. Mixing `int` with `size_t` generates warnings and can produce wrong results when the container has more than 2^31 elements.

---

## 6. FLOATING-POINT TYPES

C++ floating-point types follow IEEE 754 (on virtually all platforms, though the standard technically allows other formats).

```
Type           Bits    Bytes    Significant Digits    Approximate Range
───────────────────────────────────────────────────────────────────────────────
float          32      4        ~7 decimal digits     ±1.18e-38 to ±3.40e+38
double         64      8        ~15-16 decimal digits ±2.23e-308 to ±1.80e+308
long double    80/128  10/16    18-19 / 33-34 digits  platform-dependent
```

### 6.1 IEEE 754 Single-Precision `float` (32-bit)

Bit layout:
- 1 sign bit
- 8 exponent bits (biased by 127)
- 23 mantissa bits (+ 1 implicit leading bit = 24 significant bits)

Special values: +∞, -∞, NaN (quiet and signaling), +0 and -0 (compare equal but differ in sign bit).

### 6.2 IEEE 754 Double-Precision `double` (64-bit)

Bit layout:
- 1 sign bit
- 11 exponent bits (biased by 1023)
- 52 mantissa bits (+ 1 implicit = 53 significant bits)

This is the default floating-point type in C++. Unadorned literals like `3.14` are `double`.

### 6.3 `long double` — The Portability Nightmare

This is the most platform-variable type in all of C++:

```
Platform/Compiler                    Size         Format
──────────────────────────────────────────────────────────────────────
x86-64 Linux/macOS (GCC/Clang)       80-bit        x87 extended precision (10 bytes, often padded to 12 or 16)
x86-64 Windows (MSVC)                64-bit        Same as double (no x87 extended)
ARM64 (Apple M-series, AArch64)      128-bit       IEEE 754 quad precision
PowerPC                              128-bit       IBM double-double or IEEE quad
RISC-V (with Q extension)            128-bit       IEEE quad
```

On MSVC, `long double` is identical to `double`. On GCC/Clang for x86-64, it is 80-bit x87. Never use `long double` in portable serialization or cross-platform data exchange.

### 6.4 Extended and Non-standard Float Types

GCC and Clang on x86-64 support `__float128` — a 128-bit IEEE quad-precision type via software emulation. Not part of the C++ standard.

C++23 introduces standard extended floating-point types via `<stdfloat>`:

```
std::float16_t    (16-bit, IEEE 754 half-precision)
std::bfloat16_t   (16-bit, bfloat16 — 8-bit exponent, 7-bit mantissa, same range as float32)
std::float32_t    (alias for float on conforming platforms)
std::float64_t    (alias for double)
std::float128_t   (128-bit quad, if supported)
```

`bfloat16` has become essential in ML/AI workloads because it preserves the dynamic range of `float32` with half the memory, at the cost of less precision.

---

## 7. DATA MODEL SUMMARY TABLE

Different platforms use different data models, which define the mapping of C++ types to bit widths:

```
Data Model    char  short  int   long  long long  pointer  Used By
──────────────────────────────────────────────────────────────────────────
LP32          8     16     16    32    64          32       16-bit DOS era
ILP32         8     16     32    32    64          32       32-bit Windows/Linux
LLP64         8     16     32    32    64          64       64-bit Windows (MSVC)
LP64          8     16     32    64    64          64       64-bit Linux/macOS/Unix
ILP64         8     16     64    64    64          64       Some 64-bit Unix variants
```

The key difference between Windows and Linux 64-bit: `long` is 4 bytes on Windows (LLP64) but 8 bytes on Linux/macOS (LP64).

---

## 8. SIZEOF AND ALIGNOF

`sizeof(T)` returns the size of type T in bytes (where "byte" means `char`-sized units, which is always 8 bits on any platform you will ever use).

`alignof(T)` returns the alignment requirement of T in bytes — the address must be a multiple of this value.

Typical alignment on x86-64:

```
Type            sizeof    alignof
──────────────────────────────────
bool            1         1
char            1         1
short           2         2
int             4         4
long            8         8       (LP64)
long long       8         8
float           4         4
double          8         8
long double     16        16      (x86-64 GCC/Clang, padded from 10-byte x87)
pointer         8         8       (64-bit)
```

---

## 9. STRUCT LAYOUT, PADDING, AND ALIGNMENT

The compiler inserts padding bytes between struct members to satisfy alignment requirements. This means `sizeof(struct)` is often larger than the sum of its members.

Example:

```cpp
struct Bad {
    char  a;    // 1 byte
                // 3 bytes padding
    int   b;    // 4 bytes
    char  c;    // 1 byte
                // 7 bytes padding (to align to 8 for double)
    double d;   // 8 bytes
};
// sizeof(Bad) == 24
```

Reordering members from largest to smallest eliminates padding:

```cpp
struct Good {
    double d;   // 8 bytes  (offset 0)
    int    b;   // 4 bytes  (offset 8)
    char   a;   // 1 byte   (offset 12)
    char   c;   // 1 byte   (offset 13)
                // 2 bytes trailing padding (to round up to alignof(double) == 8)
};
// sizeof(Good) == 16
```

The general rule: sort members largest-to-smallest, then smallest-to-largest for trailing members. This minimizes wasted space.

### 9.1 Controlling Layout

`#pragma pack(n)` — sets packing to n bytes (non-standard but supported by GCC, Clang, MSVC). Use with extreme care, as misaligned accesses can be slow or UB on some architectures.

`__attribute__((packed))` — GCC/Clang extension to remove all padding from a struct.

`alignas(N)` — standard C++11 way to request a specific alignment:

```cpp
alignas(64) float cache_line_buffer[16];  // aligned to cache line
```

`alignas` can be applied to types, variables, and struct members.

---

## 10. BIT FIELDS

Bit fields allow packing multiple values into a single integer word:

```cpp
struct Flags {
    unsigned int read    : 1;
    unsigned int write   : 1;
    unsigned int execute : 1;
    unsigned int level   : 4;
};
```

Important rules and caveats:
- The layout (bit ordering within the word, how they cross word boundaries) is implementation-defined
- You cannot take the address of a bit field
- Bit fields with `bool` type are allowed since C++14
- Only `int`, `unsigned int`, `signed int`, and `bool` are guaranteed legal underlying types; `uint8_t`, `uint16_t`, etc., are common extensions
- Anonymous bit fields (unnamed) create padding bits: `unsigned : 4;`

Bit fields are useful for hardware register mapping and protocol parsing, but should be avoided for portable serialization due to implementation-defined ordering.

---

## 11. ENUMERATIONS

### 11.1 Unscoped Enum (C-style)

```cpp
enum Color { RED, GREEN, BLUE };
```

- Implicitly converts to `int`
- Enumerators leak into the enclosing scope
- Underlying type is implementation-defined (usually `int`, but can be larger if needed)

### 11.2 Scoped Enum (C++11, strongly preferred)

```cpp
enum class Direction : uint8_t { North, South, East, West };
```

- No implicit conversion to integer
- Enumerators are scoped: `Direction::North`
- Underlying type is explicitly specifiable (defaults to `int` if not specified)
- `sizeof(Direction) == 1` in the example above

Always prefer `enum class` in modern C++. Use an explicit underlying type to control size.

---

## 12. POINTERS AND REFERENCES

### 12.1 Raw Pointers

```
Type           sizeof (32-bit)    sizeof (64-bit)
──────────────────────────────────────────────────
T*             4                  8
void*          4                  8
T (*)(...)     4                  8  (function pointer)
T C::*         4-8                8-16 (member pointer, varies by ABI)
```

All data pointers are the same size regardless of what they point to. Function pointers are always the same size as data pointers on mainstream platforms, but the standard technically permits them to differ.

Member pointers (pointer-to-member) can be larger than regular pointers, especially in MSVC with multiple inheritance and virtual inheritance, which uses a struct-based representation that can be up to 20 bytes.

### 12.2 References

References have no size — they are aliases. The compiler may or may not allocate storage for them. When stored in a struct, a reference member behaves like a pointer in size (the compiler stores a pointer internally), but you cannot take `sizeof` of a reference directly (it gives `sizeof` of the referred-to type).

### 12.3 Smart Pointers (`<memory>`)

```
Type                      sizeof (64-bit)    Notes
──────────────────────────────────────────────────────────────────────────────
std::unique_ptr<T>        8                  Same as raw pointer (zero overhead)
std::unique_ptr<T, D>     8 or 16            With custom deleter, uses EBO if stateless
std::shared_ptr<T>        16                 Two pointers: object ptr + control block ptr
std::weak_ptr<T>          16                 Same layout as shared_ptr
```

`std::unique_ptr` with a stateless deleter has zero size overhead over a raw pointer (EBO — Empty Base Optimization applies). `std::shared_ptr` always costs 2 pointers due to reference counting.

---

## 13. ARRAYS

For type T and size N:

```
sizeof(T[N]) == sizeof(T) * N
```

No padding between elements — arrays are always tightly packed. However, arrays of structs will have the per-struct padding included in each element's `sizeof`.

`alignof(T[N]) == alignof(T)` — the array alignment equals the element alignment.

Stack-allocated arrays must fit within the stack frame (typically 1-8 MB). Large arrays should be heap-allocated.

Variable-length arrays (VLAs) are not standard C++ (they are a C99 feature). GCC and Clang support them as an extension, but they are undefined behavior in standard C++. Use `std::vector` or `std::array` instead.

---

## 14. STL CONTAINER SIZES

On 64-bit platforms with libstdc++ or libc++:

```
Container                    sizeof    Notes
──────────────────────────────────────────────────────────────────────────────────
std::string                  24/32     SSO: 24 bytes (libstdc++), 24 bytes (MSVC), 32 bytes (libc++)
std::vector<T>               24        3 pointers: begin, end, end_of_capacity
std::array<T, N>             N*sizeof(T)  Stack-allocated, no overhead
std::deque<T>                80/88     Complex block-based structure
std::list<T>                 24        2 pointers + size
std::forward_list<T>         8         1 pointer (no size stored)
std::map<K,V>                48        Red-black tree with allocator + size
std::unordered_map<K,V>      56-72     Hash table with load factor, bucket array
std::optional<T>             sizeof(T) + 1 (+ padding)
std::variant<Ts...>          sizeof(largest T) + alignment + index
std::tuple<Ts...>            sum of member sizes + padding (reordered for alignment)
std::pair<A,B>               sizeof(A) + sizeof(B) + padding
std::function<Sig>           40-64     Type-erased callable with SBO buffer
std::span<T>                 16        Pointer + size (non-owning)
std::string_view             16        Pointer + size (non-owning)
```

SSO (Small String Optimization): short strings are stored inline inside the `std::string` object itself without heap allocation. GCC's libstdc++ inlines up to 15 characters. MSVC inlines up to 15. libc++ (Clang/macOS) inlines up to 22. This means accessing a short string never touches the heap.

---

## 15. NUMERIC LIMITS

Use `<limits>` for querying type properties at compile time:

```cpp
#include <limits>

std::numeric_limits<int>::min()           // -2147483648
std::numeric_limits<int>::max()           // 2147483647
std::numeric_limits<unsigned int>::max()  // 4294967295
std::numeric_limits<float>::epsilon()     // ~1.19e-7  (machine epsilon)
std::numeric_limits<float>::infinity()    // +inf
std::numeric_limits<float>::is_iec559     // true if IEEE 754
std::numeric_limits<T>::digits            // number of significant bits in mantissa
std::numeric_limits<T>::digits10          // significant decimal digits
```

C-style macros in `<climits>` and `<cfloat>` also work:

```
INT_MIN, INT_MAX, UINT_MAX
LLONG_MIN, LLONG_MAX, ULLONG_MAX
FLT_EPSILON, DBL_EPSILON, LDBL_EPSILON
FLT_MAX, DBL_MAX
```

The C++ `<limits>` interface is preferred in modern code.

---

## 16. INTEGER PROMOTION AND USUAL ARITHMETIC CONVERSIONS

This is one of the most misunderstood areas of C++ and the source of many latent bugs.

### 16.1 Integer Promotion

Any type smaller than `int` (i.e., `bool`, `char`, `unsigned char`, `signed char`, `short`, `unsigned short`) is automatically promoted to `int` (or `unsigned int` if `int` cannot represent all values of the smaller type) before any arithmetic operation.

This means:

```cpp
uint8_t a = 200, b = 100;
auto result = a + b;  // result is int (300), not uint8_t (44)
```

```cpp
uint8_t x = 1;
auto y = ~x;  // y is int (-2), not uint8_t (254)
```

### 16.2 Usual Arithmetic Conversions

When two operands of different types appear in a binary expression, the compiler converts both to a common type following these rules in order:

1. If either is `long double` → both become `long double`
2. If either is `double` → both become `double`
3. If either is `float` → both become `float`
4. Both are integers: apply integer promotions, then:
   a. If same signedness: smaller converts to larger
   b. If different signedness and unsigned type is larger or equal: signed converts to unsigned
   c. If different signedness and signed type is larger: unsigned converts to signed

Rule 4b is the signed/unsigned mismatch trap:

```cpp
int a = -1;
unsigned int b = 1;
if (a < b)  // FALSE! -1 converts to unsigned: becomes 4294967295
    std::cout << "math works";
```

Always enable `-Wsign-conversion` in your compiler flags.

---

## 17. OVERFLOW AND UNDEFINED BEHAVIOR

### 17.1 Signed Integer Overflow

Signed integer overflow is undefined behavior in C++. The compiler is permitted to assume it never happens and optimize accordingly. This is not hypothetical — GCC and Clang actively exploit this assumption.

```cpp
int x = INT_MAX;
x + 1;  // UB — compiler may eliminate checks based on this
```

Avoid by using `__builtin_add_overflow`, `std::add_sat` (C++26), or checking before the operation.

### 17.2 Unsigned Integer Overflow

Unsigned overflow is well-defined — it wraps modulo 2^N. `UINT_MAX + 1 == 0` is guaranteed.

### 17.3 Floating-Point Overflow

IEEE 754 defines overflow to produce ±∞. This is not UB (assuming IEEE 754 semantics, which `-ffast-math` can violate).

---

## 18. TYPE DEDUCTION AND AUTO

`auto` deduces the type like template argument deduction:
- Reference and cv-qualifiers (const, volatile) are stripped from the deduced type
- To keep a reference: `auto&`
- To keep const: `const auto`
- To get an exact forwarding reference: `auto&&`

`decltype(expr)` gives the exact type of an expression, including references and cv-qualifiers, without stripping anything.

`decltype(auto)` (C++14) combines both: deduces as `decltype` but with initialization syntax.

```cpp
int x = 5;
int& ref = x;

auto  a = ref;        // int (reference stripped)
auto& b = ref;        // int& (explicitly kept)
decltype(ref) c = x;  // int& (exact type preserved)
```

---

## 19. CONSTEXPR AND COMPILE-TIME TYPES

`constexpr` variables are evaluated at compile time. Their types follow all the same rules, but values must be computable in a constant expression context.

`std::integral_constant<T, v>` — the foundation of type-level integer programming in template metaprogramming:

```cpp
using Two = std::integral_constant<int, 2>;
Two::value;  // 2 (constexpr static member)
```

`std::bool_constant<B>` is shorthand for `std::integral_constant<bool, B>`. `std::true_type` and `std::false_type` are the canonical aliases.

---

## 20. TYPE TRAITS (`<type_traits>`)

Type traits let you query type properties at compile time.

Size and layout queries:

```cpp
std::is_trivially_copyable<T>::value    // can memcpy?
std::is_standard_layout<T>::value       // C-compatible layout?
std::is_pod<T>::value                   // deprecated C++20; was trivially copyable + standard layout
std::alignment_of<T>::value             // same as alignof(T)
std::rank<T>::value                     // number of array dimensions
std::extent<T, N>::value                // size of Nth dimension
```

Type classification:

```cpp
std::is_integral<T>
std::is_floating_point<T>
std::is_arithmetic<T>
std::is_signed<T>
std::is_unsigned<T>
std::is_void<T>
std::is_pointer<T>
std::is_reference<T>
std::is_array<T>
std::is_enum<T>
std::is_class<T>
```

Type transformations:

```cpp
std::make_signed<T>::type       // e.g., uint32_t → int32_t
std::make_unsigned<T>::type     // e.g., int32_t → uint32_t
std::remove_reference<T>::type
std::remove_const<T>::type
std::remove_cv<T>::type         // remove const and volatile
std::add_pointer<T>::type
std::underlying_type<E>::type   // underlying type of an enum
```

---

## 21. SIMD AND VECTOR TYPES (ADVANCED)

Modern CPUs operate on data in wide registers. On x86-64:

```
__m64     — MMX,  64-bit  register (deprecated)
__m128    — SSE,  128-bit register (4× float)
__m128d   — SSE2, 128-bit register (2× double)
__m128i   — SSE2, 128-bit register (integers: 16×int8, 8×int16, 4×int32, 2×int64)
__m256    — AVX,  256-bit register (8× float)
__m256d   — AVX,  256-bit register (4× double)
__m256i   — AVX2, 256-bit register (32×int8, 16×int16, 8×int32, 4×int64)
__m512    — AVX-512, 512-bit (16× float)
__m512d   — AVX-512, 512-bit (8× double)
__m512i   — AVX-512, 512-bit (64×int8, 32×int16, 16×int32, 8×int64)
```

These types require `<immintrin.h>`. Their alignment requirements:

```
__m128*   — 16-byte aligned (use _mm_loadu_* for unaligned loads)
__m256*   — 32-byte aligned (use _mm256_loadu_* for unaligned)
__m512*   — 64-byte aligned (use _mm512_loadu_* for unaligned)
```

C++23 `std::simd<T, ABI>` (in `<experimental/simd>` as a TS, standardized in C++26) provides a portable abstraction over these:

```cpp
std::simd<float, std::simd_abi::avx> v;  // 8× float, AVX register
```

---

## 22. 128-BIT INTEGER TYPES (COMPILER EXTENSIONS)

GCC and Clang support 128-bit integer arithmetic on x86-64 via extensions:

```cpp
__int128          i128 = ...;
unsigned __int128 u128 = ...;
```

Size: 16 bytes, alignment: 16 bytes.

There is no standard `<limits>` specialization for `__int128`. These types are not part of the C++ standard but are widely supported. They are synthesized from pairs of 64-bit operations — no hardware 128-bit integer instructions exist on x86-64 (unlike 128-bit SIMD).

---

## 23. PLATFORM-SPECIFIC AND COMPILER-SPECIFIC TYPES

MSVC-specific:
- `__int8`, `__int16`, `__int32`, `__int64` — MSVC aliases for fixed-width types (prefer `<cstdint>`)

GCC/Clang extensions:
- `__builtin_int128`, `__float128`
- `_Float16` — 16-bit IEEE half-precision (with appropriate hardware support)

CUDA (for those writing GPU code):
- `half` (16-bit float), `nv_bfloat16` (bfloat16), `half2`, `float2`, `float4`

---

## 24. PRACTICAL DECISION GUIDE

When to use which integer type:

```
Scenario                                      Type to Use
────────────────────────────────────────────────────────────────────────────
Container sizes, loop indices                 size_t
Pointer arithmetic, pointer differences       ptrdiff_t
Exact 8/16/32/64-bit values                   uint8_t / int32_t / etc.
General-purpose integer, speed critical       int (native word is fast)
Flags, bitmasks                               uint32_t or uint64_t
File offsets, timestamps                      int64_t
Enum underlying type when size matters        uint8_t, uint16_t, etc.
Cross-platform serialized data                Explicit fixed-width types only
```

When to use which float type:

```
Scenario                                      Type to Use
────────────────────────────────────────────────────────────────────────────
General numerical computation                 double
Memory-limited: bulk arrays, ML weights       float
Interop with GPU / ML frameworks              float (or bfloat16/float16 via C++23)
High-precision numerical methods              double; long double only if genuinely needed
Financial: never use floats                   Use scaled integers or a decimal library
```

---

## 25. KEY RULES TO INTERNALIZE

1. `sizeof(char) == 1` always. Everything else is platform-dependent unless you use fixed-width types.

2. `long` is 4 bytes on 64-bit Windows, 8 bytes on 64-bit Linux/macOS. Never use it for exact widths.

3. Signed integer overflow is undefined behavior. Unsigned overflow wraps. These are completely different.

4. Any arithmetic on types smaller than `int` (char, short, bool) silently promotes to `int` first.

5. Mixing signed and unsigned in comparisons converts the signed value to unsigned, which can flip comparisons.

6. `std::string` and `std::vector` have their own internal sizing — `sizeof(std::string)` is the object size on the stack, not the string length.

7. Struct sizes include padding. `sizeof(A) + sizeof(B) != sizeof(struct{A;B;})` in general.

8. `alignof(T)` constrains where a T can live in memory. Violating alignment is UB on x86 (in some cases), and a hard crash on ARM.

9. For portable serialization, always use fixed-width types and account for endianness explicitly.

10. `auto` strips references and cv-qualifiers. Use `decltype` when you need the exact type.