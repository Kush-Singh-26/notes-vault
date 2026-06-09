---
title: "AVX2 & FMA SIMD Intrinsics: The Definitive Reference"
description: "Gold-standard reference for SIMD programming with AVX2 and FMA on x86-64."
---

## 1. SIMD Fundamentals

### What Is SIMD?

**SIMD** (Single Instruction, Multiple Data) is a class of parallel computation where one instruction operates on multiple data elements simultaneously. Instead of processing one `float` at a time, a single AVX2 instruction can process **8 floats** in parallel.

```
Scalar addition:     a[0]+b[0], a[1]+b[1], a[2]+b[2], a[3]+b[3]  — 4 instructions
SIMD (AVX2) addition: [a0,a1,a2,a3,a4,a5,a6,a7] + [b0,b1,b2,b3,b4,b5,b6,b7] — 1 instruction
```

### Flynn's Taxonomy

| Class | Instructions | Data streams | Example |
|-------|-------------|--------------|---------|
| SISD  | Single      | Single       | Scalar CPU |
| SIMD  | Single      | Multiple     | AVX2, NEON |
| MISD  | Multiple    | Single       | (rare/theoretical) |
| MIMD  | Multiple    | Multiple     | Multi-core, GPU |

### SIMD Registers as Vectors

A 256-bit YMM register holds a *vector* — multiple *lanes* of the same scalar type:

```
256-bit YMM register layouts:
┌───────────────────────────────────────────────────────────────────┐
│   8 × float32   (32-bit each)                                     │
├───────────────────────────────────────────────────────────────────┤
│   4 × float64   (64-bit each)                                     │
├───────────────────────────────────────────────────────────────────┤
│  32 × int8      ( 8-bit each)                                     │
├───────────────────────────────────────────────────────────────────┤
│  16 × int16     (16-bit each)                                     │
├───────────────────────────────────────────────────────────────────┤
│   8 × int32     (32-bit each)                                     │
├───────────────────────────────────────────────────────────────────┤
│   4 × int64     (64-bit each)                                     │
└───────────────────────────────────────────────────────────────────┘
```

### Key Concepts

- **Vector width**: number of bits in the register (128-bit XMM, 256-bit YMM, 512-bit ZMM)
- **Lane**: a single element slot within a vector
- **Packed vs Scalar**: packed ops act on all lanes; scalar ops act on the lowest lane only
- **Alignment**: memory addresses should be aligned to the vector width for best performance (32-byte for AVX2)
- **Throughput vs Latency**: throughput = how many per cycle (with pipelining), latency = cycles until result is ready

---

## 2. x86 SIMD Evolution Timeline

| Extension | Year | Register width | Key additions |
|-----------|------|----------------|---------------|
| MMX       | 1997 | 64-bit MM       | Integer SIMD on FPU registers |
| SSE       | 1999 | 128-bit XMM     | 4× float32 packed |
| SSE2      | 2001 | 128-bit XMM     | 2× float64, integer ops |
| SSE3      | 2004 | 128-bit XMM     | Horizontal add, `lddqu` |
| SSSE3     | 2007 | 128-bit XMM     | `pshufb`, abs, sign |
| SSE4.1    | 2007 | 128-bit XMM     | `blend`, `insertps`, `dpps`, `mpsadbw` |
| SSE4.2    | 2008 | 128-bit XMM     | CRC32, string ops (`pcmpestri`) |
| AVX       | 2011 | 256-bit YMM     | 256-bit float32/64, 3-operand encoding |
| AVX2      | 2013 | 256-bit YMM     | **256-bit integer**, gather, `vperm2i128`, `vpsllvd` |
| FMA3      | 2013 | 256-bit YMM     | Fused multiply-add (3-operand) |
| AVX-512   | 2017 | 512-bit ZMM     | Masking, scatter, new ops |

> **AVX2 (Haswell, 2013) is the most widely deployed SIMD baseline** for performance-critical code. It runs on all major server and consumer Intel/AMD CPUs from ~2015 onward.

### VEX Encoding

AVX introduced **VEX encoding** — a new instruction encoding prefix that:
- Allows **3-operand, non-destructive** forms: `vaddps ymm0, ymm1, ymm2` (vs old `addps xmm0, xmm1` which overwrites)
- Zero-extends XMM writes to full 256-bit YMM (upper bits cleared on 128-bit writes)
- Required to use YMM registers at all

---

## 3. AVX2 Architecture & Registers

### YMM Registers

AVX2 provides **16 YMM registers** (256 bits each) on x86-64:

```
YMM0  – YMM15   (256-bit)
  └── lower 128 bits aliased as XMM0–XMM15
        └── lower 64 bits aliased as MM0–MM7 (MMX, largely obsolete)
```

Each YMM register is composed of two 128-bit *lanes*:

```
YMM0:  [  high 128-bit lane  |  low 128-bit lane  ]
            XMM0 upper half       XMM0 (lower 128)
```

**Critical AVX "crossing the lane" rule**: Many AVX2 instructions treat the two 128-bit lanes independently. Cross-lane shuffles (permutes) require explicit cross-lane instructions and are typically slower.

### Register File Layout

```
256-bit YMM registers (16 total on x86-64):
    ┌──────────────────────────────────────────────┐
    │  YMM0  [255:128][127:0]  ← YMM0 upper | XMM0│
    │  YMM1  [255:128][127:0]                      │
    │  ...                                         │
    │  YMM15 [255:128][127:0]                      │
    └──────────────────────────────────────────────┘

Calling conventions (System V AMD64 ABI):
    YMM0–YMM7:   Argument/return, caller-saved
    YMM8–YMM15:  Caller-saved (not preserved across calls)
    Note: on Windows x64, XMM6–XMM15 are callee-saved
```

### MXCSR — SSE/AVX Control and Status Register

A 32-bit register controlling floating-point behavior (see Section 18 for full details):

```
Bit  Meaning
 0   IE  Invalid Operation Flag
 1   DE  Denormalized Operand Flag
 2   ZE  Divide-by-Zero Flag
 3   OE  Overflow Flag
 4   UE  Underflow Flag
 5   PE  Precision Flag
 6   DAZ Denormals Are Zero
 7   IM  Invalid Operation Mask
 8   DM  Denormal Mask
 9   ZM  Divide-by-Zero Mask
10   OM  Overflow Mask
11   UM  Underflow Mask
12   PM  Precision Mask
13–14 RC Rounding Control: 00=nearest, 01=down, 10=up, 11=toward-zero
15   FZ  Flush to Zero
```

---

## 4. Data Types & Naming Conventions

### Intrinsic Type System

```c
// 128-bit types (SSE/AVX)
__m128    // 4 × float32
__m128d   // 2 × float64
__m128i   // 128-bit integer (8×i16, 4×i32, 2×i64, 16×i8, etc.)

// 256-bit types (AVX/AVX2)
__m256    // 8 × float32
__m256d   // 4 × float64
__m256i   // 256-bit integer (32×i8, 16×i16, 8×i32, 4×i64)

// 512-bit types (AVX-512 — not AVX2, but shown for context)
__m512    // 16 × float32
__m512d   // 8 × float64
__m512i   // 512-bit integer
```

### Intrinsic Naming Schema

Every intrinsic follows a consistent naming pattern:

```
_mm<width>_<operation>_<suffix>

Components:
  _mm      = 128-bit (SSE/legacy)
  _mm256   = 256-bit (AVX/AVX2)
  _mm512   = 512-bit (AVX-512)

  <operation> = functional name (add, mul, load, shuffle, etc.)

  <suffix> = element type:
    ps  = packed single (float32)
    pd  = packed double (float64)
    ss  = scalar single (float32, lowest lane only)
    sd  = scalar double (float64, lowest lane only)
    epi8  / epi16 / epi32 / epi64  = signed integer (8/16/32/64-bit)
    epu8  / epu16 / epu32 / epu64  = unsigned integer
    si256 = opaque 256-bit integer (e.g. cast operations)
```

**Examples:**

| Intrinsic | Meaning |
|-----------|---------|
| `_mm256_add_ps`   | Add 8 float32 values in YMM registers |
| `_mm256_mul_pd`   | Multiply 4 float64 values |
| `_mm256_add_epi32`| Add 8 signed int32 values |
| `_mm_add_ss`      | Add lowest float32 lane only (scalar) |
| `_mm256_loadu_ps` | Unaligned load of 8 floats |

### Required Header

```c
#include <immintrin.h>   // Includes all x86 intrinsics (SSE through AVX-512)

// Or include specific headers (older style):
// #include <xmmintrin.h>  // SSE
// #include <emmintrin.h>  // SSE2
// #include <pmmintrin.h>  // SSE3
// #include <tmmintrin.h>  // SSSE3
// #include <smmintrin.h>  // SSE4.1
// #include <nmmintrin.h>  // SSE4.2
// #include <immintrin.h>  // AVX, AVX2, FMA, AVX-512
```

---

## 5. Compiler Setup & Feature Detection

### Compiler Flags

```bash
# GCC / Clang
-mavx2          # Enable AVX2 instructions
-mfma           # Enable FMA instructions  
-mavx2 -mfma    # Both (typical)
-march=native   # Enable everything the current CPU supports (not portable)
-march=haswell  # Target Haswell (AVX2 + FMA + BMI2 + MOVBE + POPCNT + ...)
-O2 -mavx2 -mfma -ftree-vectorize  # Full optimization + vectorization hints

# MSVC
/arch:AVX2      # Enable AVX2 (also implies FMA on MSVC)

# ICC (Intel Compiler)
-xCORE-AVX2     # Enables AVX2 for Intel CPUs
```

### Runtime CPU Feature Detection

Never assume a CPU supports AVX2. Always detect at runtime for distributed code:

```c
#include <cpuid.h>   // GCC/Clang
// or #include <intrin.h> on MSVC

typedef struct {
    bool avx2;
    bool fma;
    bool avx512f;
    bool bmi2;
} cpu_features_t;

static inline cpu_features_t detect_cpu_features(void) {
    cpu_features_t f = {0};
    unsigned int eax, ebx, ecx, edx;

    // Check OSXSAVE (OS has enabled extended state saving for YMM registers)
    __cpuid(1, eax, ebx, ecx, edx);
    bool osxsave = (ecx >> 27) & 1;
    bool avx_supported = (ecx >> 28) & 1;
    if (!osxsave || !avx_supported) return f;

    // Verify OS support for YMM (XGETBV.XCR0 bits 1 and 2 set)
    unsigned long long xcr0 = _xgetbv(0);
    if ((xcr0 & 0x6) != 0x6) return f;  // YMM not saved by OS

    // CPUID leaf 7, sub-leaf 0
    __cpuid_count(7, 0, eax, ebx, ecx, edx);
    f.avx2     = (ebx >> 5)  & 1;   // AVX2
    f.bmi2     = (ebx >> 8)  & 1;   // BMI2
    f.avx512f  = (ebx >> 16) & 1;   // AVX-512 Foundation

    // FMA check: CPUID leaf 1 ECX bit 12
    __cpuid(1, eax, ebx, ecx, edx);
    f.fma = (ecx >> 12) & 1;

    return f;
}

// Usage:
cpu_features_t cpu = detect_cpu_features();
if (cpu.avx2 && cpu.fma) {
    process_avx2_fma(data, len);
} else {
    process_scalar(data, len);
}
```

### GCC Built-in CPU Detection

```c
// GCC 4.8+ built-in
if (__builtin_cpu_supports("avx2")) { ... }
if (__builtin_cpu_supports("fma"))  { ... }
```

### Target Attributes (Function-Level)

```c
// Compile a specific function with AVX2 even if the translation unit doesn't use -mavx2
__attribute__((target("avx2,fma")))
void process_avx2(float* data, int n) {
    // AVX2 + FMA intrinsics safe here
}

// GCC function multi-versioning (automatic dispatch)
__attribute__((target("avx2,fma")))
void compute(float* a, float* b, int n);  // AVX2 version

__attribute__((target("default")))
void compute(float* a, float* b, int n);  // Fallback version
// GCC automatically selects the right version at runtime
```

---

## 6. Memory Operations

Memory operations are the entry and exit points for SIMD — you load data into vectors, process them, and store back.

### Load Operations

```c
// ── Aligned loads (address must be 32-byte aligned for 256-bit) ──────────────
__m256  _mm256_load_ps(float const* mem_addr);        // 8 × float32, aligned
__m256d _mm256_load_pd(double const* mem_addr);       // 4 × float64, aligned
__m256i _mm256_load_si256(__m256i const* mem_addr);   // 256-bit integer, aligned

// ── Unaligned loads (no alignment requirement, slightly slower on older CPUs) ─
__m256  _mm256_loadu_ps(float const* mem_addr);       // 8 × float32, unaligned
__m256d _mm256_loadu_pd(double const* mem_addr);      // 4 × float64, unaligned
__m256i _mm256_loadu_si256(__m256i const* mem_addr);  // 256-bit integer, unaligned

// ── 128-bit loads into YMM (upper lane zero-extended) ─────────────────────────
__m256  _mm256_castps128_ps256(__m128 a);             // 128→256 bit, upper undefined
__m256  _mm256_zextps128_ps256(__m128 a);             // 128→256 bit, upper zero-cleared

// ── Broadcast: load one scalar and replicate to all lanes ─────────────────────
__m256  _mm256_broadcast_ss(float const* mem_addr);   // Broadcast float32 to 8 lanes
__m256d _mm256_broadcast_sd(double const* mem_addr);  // Broadcast float64 to 4 lanes
__m256i _mm256_broadcastb_epi8 (__m128i a);           // Broadcast  8-bit to 32 lanes
__m256i _mm256_broadcastw_epi16(__m128i a);           // Broadcast 16-bit to 16 lanes
__m256i _mm256_broadcastd_epi32(__m128i a);           // Broadcast 32-bit to  8 lanes
__m256i _mm256_broadcastq_epi64(__m128i a);           // Broadcast 64-bit to  4 lanes

// Broadcast from register (not memory):
__m256  _mm256_set1_ps(float a);                      // All 8 lanes = a
__m256d _mm256_set1_pd(double a);                     // All 4 lanes = a
__m256i _mm256_set1_epi32(int a);                     // All 8 lanes = a (int32)
__m256i _mm256_set1_epi64x(long long a);              // All 4 lanes = a (int64)
__m256i _mm256_set1_epi16(short a);                   // All 16 lanes = a
__m256i _mm256_set1_epi8(char a);                     // All 32 lanes = a

// ── Streaming (non-temporal) load — bypasses cache ────────────────────────────
__m256i _mm256_stream_load_si256(__m256i const* mem_addr);  // Non-temporal load
```

### Store Operations

```c
// ── Aligned stores ────────────────────────────────────────────────────────────
void _mm256_store_ps (float*    mem_addr, __m256  a);
void _mm256_store_pd (double*   mem_addr, __m256d a);
void _mm256_store_si256(__m256i* mem_addr, __m256i a);

// ── Unaligned stores ──────────────────────────────────────────────────────────
void _mm256_storeu_ps (float*    mem_addr, __m256  a);
void _mm256_storeu_pd (double*   mem_addr, __m256d a);
void _mm256_storeu_si256(__m256i* mem_addr, __m256i a);

// ── Non-temporal (streaming) stores — write to memory, bypassing cache ────────
// Use when writing large amounts of data you won't read back soon
void _mm256_stream_ps (float*    mem_addr, __m256  a);   // Must be 32-byte aligned
void _mm256_stream_pd (double*   mem_addr, __m256d a);
void _mm256_stream_si256(__m256i* mem_addr, __m256i a);

// ── Masked stores — only write lanes where mask bit is set ────────────────────
void _mm256_maskstore_ps (float*  mem_addr, __m256i mask, __m256  a);
void _mm256_maskstore_pd (double* mem_addr, __m256i mask, __m256d a);
void _mm256_maskstore_epi32(int*      mem_addr, __m256i mask, __m256i a);
void _mm256_maskstore_epi64(long long* mem_addr, __m256i mask, __m256i a);
// mask lane = active if the most significant bit of that lane is set (bit 31 or 63)
```

### Set / Initialize Operations

```c
// ── Set all elements explicitly (arguments listed from HIGHEST to LOWEST lane) ─
__m256  _mm256_set_ps(float e7,e6,e5,e4,e3,e2,e1,e0);
__m256d _mm256_set_pd(double e3,e2,e1,e0);
__m256i _mm256_set_epi32(int e7,e6,e5,e4,e3,e2,e1,e0);
__m256i _mm256_set_epi64x(long long e3,e2,e1,e0);
__m256i _mm256_set_epi16(short e15,...,e0);
__m256i _mm256_set_epi8(char e31,...,e0);

// ── Set zero ──────────────────────────────────────────────────────────────────
__m256  _mm256_setzero_ps(void);
__m256d _mm256_setzero_pd(void);
__m256i _mm256_setzero_si256(void);

// ── Set reversed (low lane first, matching array order — easier to reason about)
__m256  _mm256_setr_ps(float e0,e1,e2,e3,e4,e5,e6,e7);
__m256d _mm256_setr_pd(double e0,e1,e2,e3);
__m256i _mm256_setr_epi32(int e0,e1,e2,e3,e4,e5,e6,e7);

// ── Undefined (optimization hint: do not initialize) ─────────────────────────
__m256  _mm256_undefined_ps(void);
__m256d _mm256_undefined_pd(void);
__m256i _mm256_undefined_si256(void);
```

### Prefetch Operations

```c
#include <xmmintrin.h>

// Prefetch data into cache before it's needed
void _mm_prefetch(char const* p, int i);

// i = hint:
#define _MM_HINT_T0   1   // Prefetch into all cache levels (L1, L2, L3)
#define _MM_HINT_T1   2   // Prefetch into L2 and L3 only
#define _MM_HINT_T2   3   // Prefetch into L3 only
#define _MM_HINT_NTA  0   // Non-temporal: fetch to L1, do not pollute other levels

// Example: prefetch 64 bytes ahead in a streaming loop
for (int i = 0; i < n; i += 8) {
    _mm_prefetch((char*)&data[i + 64], _MM_HINT_T0);
    __m256 v = _mm256_loadu_ps(&data[i]);
    // ... process v
}
```

### Memory Fence / Ordering

```c
void _mm_sfence(void);   // Store fence: all prior stores complete before any later stores
void _mm_lfence(void);   // Load fence: all prior loads complete before any later loads
void _mm_mfence(void);   // Memory fence: all prior loads AND stores complete
// Use before/after non-temporal stores to ensure visibility
```

---

## 7. Arithmetic Intrinsics

### Floating-Point Arithmetic

#### Addition

```c
// 256-bit packed
__m256  _mm256_add_ps(__m256  a, __m256  b);  // a[i] + b[i], 8 × float32
__m256d _mm256_add_pd(__m256d a, __m256d b);  // a[i] + b[i], 4 × float64

// 128-bit packed
__m128  _mm_add_ps(__m128  a, __m128  b);     // 4 × float32
__m128d _mm_add_pd(__m128d a, __m128d b);     // 2 × float64

// Scalar (lowest lane only)
__m128  _mm_add_ss(__m128 a, __m128 b);       // a[0] + b[0]; upper lanes from a
__m128d _mm_add_sd(__m128d a, __m128d b);     // a[0] + b[0]; upper lane from a
```

#### Subtraction

```c
__m256  _mm256_sub_ps(__m256  a, __m256  b);  // a[i] - b[i], 8 × float32
__m256d _mm256_sub_pd(__m256d a, __m256d b);  // a[i] - b[i], 4 × float64
__m128  _mm_sub_ps(__m128  a, __m128  b);
__m128d _mm_sub_pd(__m128d a, __m128d b);
__m128  _mm_sub_ss(__m128 a, __m128 b);
__m128d _mm_sub_sd(__m128d a, __m128d b);
```

#### Multiplication

```c
__m256  _mm256_mul_ps(__m256  a, __m256  b);  // a[i] * b[i], 8 × float32
__m256d _mm256_mul_pd(__m256d a, __m256d b);  // a[i] * b[i], 4 × float64
__m128  _mm_mul_ps(__m128  a, __m128  b);
__m128d _mm_mul_pd(__m128d a, __m128d b);
__m128  _mm_mul_ss(__m128 a, __m128 b);
__m128d _mm_mul_sd(__m128d a, __m128d b);
```

#### Division

```c
__m256  _mm256_div_ps(__m256  a, __m256  b);  // a[i] / b[i], 8 × float32
__m256d _mm256_div_pd(__m256d a, __m256d b);  // a[i] / b[i], 4 × float64
__m128  _mm_div_ps(__m128  a, __m128  b);
__m128d _mm_div_pd(__m128d a, __m128d b);
// Note: division is EXPENSIVE (~20 cycles latency). Use reciprocal approximation when precision allows.
```

#### Square Root

```c
__m256  _mm256_sqrt_ps(__m256  a);   // sqrt(a[i]) for 8 × float32
__m256d _mm256_sqrt_pd(__m256d a);   // sqrt(a[i]) for 4 × float64
__m128  _mm_sqrt_ps(__m128  a);
__m128d _mm_sqrt_pd(__m128d a);
__m128  _mm_sqrt_ss(__m128 a);       // sqrt of lowest lane
__m128d _mm_sqrt_sd(__m128d a, __m128d b);
```

#### Approximate Reciprocal and Reciprocal Square Root

> **Note**: These have ~11-bit precision (~0.03% error). Use Newton-Raphson refinement for higher precision.

```c
// Reciprocal approximation: ~1/a  (14-bit precision, 1-2 ULP error)
__m256  _mm256_rcp_ps(__m256  a);    // ~1.0 / a[i], 8 × float32
__m128  _mm_rcp_ps(__m128  a);
__m128  _mm_rcp_ss(__m128 a);

// Reciprocal square root approximation: ~1/sqrt(a)
__m256  _mm256_rsqrt_ps(__m256  a);  // ~1.0 / sqrt(a[i]), 8 × float32
__m128  _mm_rsqrt_ps(__m128  a);
__m128  _mm_rsqrt_ss(__m128 a);

// Newton-Raphson refinement for rsqrt (reaches ~23-bit precision):
// result = 0.5 * x * (3.0 - a * x * x)   where x = rsqrt(a)
__m256 fast_rsqrt_refined(__m256 a) {
    __m256 x    = _mm256_rsqrt_ps(a);
    __m256 half = _mm256_set1_ps(0.5f);
    __m256 three = _mm256_set1_ps(3.0f);
    __m256 ax2  = _mm256_mul_ps(a, _mm256_mul_ps(x, x));
    __m256 term = _mm256_sub_ps(three, ax2);
    return _mm256_mul_ps(half, _mm256_mul_ps(x, term));
    // With FMA: return _mm256_mul_ps(half, _mm256_fmadd_ps(x, term, _mm256_setzero_ps()));
}
```

#### Min / Max

```c
__m256  _mm256_min_ps(__m256  a, __m256  b);   // min(a[i], b[i]), 8 × float32
__m256  _mm256_max_ps(__m256  a, __m256  b);   // max(a[i], b[i])
__m256d _mm256_min_pd(__m256d a, __m256d b);   // min(a[i], b[i]), 4 × float64
__m256d _mm256_max_pd(__m256d a, __m256d b);
__m128  _mm_min_ps(__m128  a, __m128  b);
__m128  _mm_max_ps(__m128  a, __m128  b);
__m128d _mm_min_pd(__m128d a, __m128d b);
__m128d _mm_max_pd(__m128d a, __m128d b);
__m128  _mm_min_ss(__m128 a, __m128 b);
__m128  _mm_max_ss(__m128 a, __m128 b);
// Note: NaN handling follows IEEE 754: if either operand is NaN, min/max returns second operand
```

#### Absolute Value and Negation (Floating-Point)

```c
// No dedicated AVX abs/neg — use bitwise ops on sign bit
__m256 _mm256_abs_ps(__m256 a) {
    // Clear sign bit (bit 31 of each float32)
    __m256i mask = _mm256_set1_epi32(0x7FFFFFFF);
    return _mm256_and_ps(a, _mm256_castsi256_ps(mask));
}

__m256 _mm256_negate_ps(__m256 a) {
    // Flip sign bit
    __m256 sign = _mm256_set1_ps(-0.0f);  // 0x80000000
    return _mm256_xor_ps(a, sign);
}
```

### Integer Arithmetic

#### Integer Addition

```c
// 8-bit elements
__m256i _mm256_add_epi8 (__m256i a, __m256i b);   // a[i]+b[i], 32 × int8, wrapping
__m256i _mm256_adds_epi8 (__m256i a, __m256i b);  // saturating add, signed 8-bit
__m256i _mm256_adds_epu8 (__m256i a, __m256i b);  // saturating add, unsigned 8-bit

// 16-bit elements
__m256i _mm256_add_epi16 (__m256i a, __m256i b);  // 16 × int16, wrapping
__m256i _mm256_adds_epi16(__m256i a, __m256i b);  // saturating add, signed 16-bit
__m256i _mm256_adds_epu16(__m256i a, __m256i b);  // saturating add, unsigned 16-bit

// 32-bit elements
__m256i _mm256_add_epi32(__m256i a, __m256i b);   // 8 × int32, wrapping

// 64-bit elements
__m256i _mm256_add_epi64(__m256i a, __m256i b);   // 4 × int64, wrapping
```

#### Integer Subtraction

```c
__m256i _mm256_sub_epi8 (__m256i a, __m256i b);
__m256i _mm256_sub_epi16(__m256i a, __m256i b);
__m256i _mm256_sub_epi32(__m256i a, __m256i b);
__m256i _mm256_sub_epi64(__m256i a, __m256i b);
__m256i _mm256_subs_epi8 (__m256i a, __m256i b);  // saturating
__m256i _mm256_subs_epi16(__m256i a, __m256i b);
__m256i _mm256_subs_epu8 (__m256i a, __m256i b);
__m256i _mm256_subs_epu16(__m256i a, __m256i b);
```

#### Integer Multiplication

```c
// 16-bit × 16-bit → 16-bit (low 16 bits of 32-bit result)
__m256i _mm256_mullo_epi16(__m256i a, __m256i b);

// 16-bit × 16-bit → 16-bit (high 16 bits of 32-bit result, signed)
__m256i _mm256_mulhi_epi16(__m256i a, __m256i b);

// 16-bit × 16-bit → 16-bit (high 16 bits, unsigned)
__m256i _mm256_mulhi_epu16(__m256i a, __m256i b);

// 32-bit × 32-bit → 32-bit (low 32 bits)
__m256i _mm256_mullo_epi32(__m256i a, __m256i b);

// 32-bit × 32-bit → 64-bit (even-indexed 32-bit elements, signed)
__m256i _mm256_mul_epi32(__m256i a, __m256i b);   // a[0]*b[0], a[2]*b[2], a[4]*b[4], a[6]*b[6]

// 32-bit × 32-bit → 64-bit (even-indexed 32-bit elements, unsigned)
__m256i _mm256_mul_epu32(__m256i a, __m256i b);

// Note: No 64×64→64 multiply in AVX2. Use _mm256_mullo_epi64 from AVX-512, or scalar.

// Multiply-Add (16-bit): a[i]*b[i] + a[i+1]*b[i+1] packed into 32-bit
__m256i _mm256_madd_epi16(__m256i a, __m256i b);
// e.g. {a0,a1,a2,a3,...} * {b0,b1,b2,b3,...} → {a0*b0+a1*b1, a2*b2+a3*b3, ...} (8 int32 results)
// CRITICAL operation for neural networks and DSP
```

#### Integer Min / Max

```c
__m256i _mm256_min_epi8 (__m256i a, __m256i b);   // min, 32 × signed int8
__m256i _mm256_max_epi8 (__m256i a, __m256i b);
__m256i _mm256_min_epu8 (__m256i a, __m256i b);   // min, 32 × unsigned int8
__m256i _mm256_max_epu8 (__m256i a, __m256i b);
__m256i _mm256_min_epi16(__m256i a, __m256i b);
__m256i _mm256_max_epi16(__m256i a, __m256i b);
__m256i _mm256_min_epu16(__m256i a, __m256i b);
__m256i _mm256_max_epu16(__m256i a, __m256i b);
__m256i _mm256_min_epi32(__m256i a, __m256i b);   // min, 8 × signed int32
__m256i _mm256_max_epi32(__m256i a, __m256i b);
__m256i _mm256_min_epu32(__m256i a, __m256i b);
__m256i _mm256_max_epu32(__m256i a, __m256i b);
// Note: No min/max for 64-bit integers in AVX2 (added in AVX-512)
```

#### Integer Absolute Value

```c
__m256i _mm256_abs_epi8 (__m256i a);   // |a[i]| for 32 × int8
__m256i _mm256_abs_epi16(__m256i a);   // |a[i]| for 16 × int16
__m256i _mm256_abs_epi32(__m256i a);   // |a[i]| for 8 × int32
```

#### Integer Average (Unsigned)

```c
__m256i _mm256_avg_epu8 (__m256i a, __m256i b);  // (a[i]+b[i]+1)>>1 for uint8 (rounded)
__m256i _mm256_avg_epu16(__m256i a, __m256i b);  // (a[i]+b[i]+1)>>1 for uint16
```

---

## 8. FMA — Fused Multiply-Add

FMA (Fused Multiply-Add) computes `a*b + c` in a single instruction with only one rounding step, giving **better accuracy** than separate multiply + add, **one fewer instruction**, and the **same latency** as a multiply.

### Why FMA Matters

```
Without FMA:
  result = a*b + c          → 2 instructions, 2 roundings, accumulates error

With FMA:
  result = fma(a, b, c)     → 1 instruction, 1 rounding, less error

Performance: ~2× throughput for multiply-accumulate heavy code (GEMM, convolutions, dot products)
```

### FMA Instruction Variants — Naming Logic

```
_mm256_fma[dir1][dir2]_[type]

dir1 = addend direction:
  add   → add (c is added)
  sub   → subtract (c is subtracted)

dir2 = product sign:
  (none) = normal multiply
  addsub = alternating add/subtract per lane (for complex arithmetic)

Full variant set:
  fmadd   → a*b + c
  fmsub   → a*b - c
  fnmadd  → -(a*b) + c   (negated multiply, then add)
  fnmsub  → -(a*b) - c   (negated multiply, then subtract)
  fmaddsub → a*b +- c    (alternating: even lanes subtract, odd lanes add)
  fmsubadd → a*b -+ c    (alternating: even lanes add, odd lanes subtract)
```

### All FMA Intrinsics

#### Float32 (256-bit)

```c
// a[i]*b[i] + c[i]  for 8 × float32
__m256 _mm256_fmadd_ps(__m256 a, __m256 b, __m256 c);

// a[i]*b[i] - c[i]
__m256 _mm256_fmsub_ps(__m256 a, __m256 b, __m256 c);

// -(a[i]*b[i]) + c[i]
__m256 _mm256_fnmadd_ps(__m256 a, __m256 b, __m256 c);

// -(a[i]*b[i]) - c[i]
__m256 _mm256_fnmsub_ps(__m256 a, __m256 b, __m256 c);

// even lanes: a[i]*b[i] - c[i], odd lanes: a[i]*b[i] + c[i]
__m256 _mm256_fmaddsub_ps(__m256 a, __m256 b, __m256 c);

// even lanes: a[i]*b[i] + c[i], odd lanes: a[i]*b[i] - c[i]
__m256 _mm256_fmsubadd_ps(__m256 a, __m256 b, __m256 c);
```

#### Float64 (256-bit)

```c
__m256d _mm256_fmadd_pd(__m256d a, __m256d b, __m256d c);
__m256d _mm256_fmsub_pd(__m256d a, __m256d b, __m256d c);
__m256d _mm256_fnmadd_pd(__m256d a, __m256d b, __m256d c);
__m256d _mm256_fnmsub_pd(__m256d a, __m256d b, __m256d c);
__m256d _mm256_fmaddsub_pd(__m256d a, __m256d b, __m256d c);
__m256d _mm256_fmsubadd_pd(__m256d a, __m256d b, __m256d c);
```

#### Float32 (128-bit)

```c
__m128 _mm_fmadd_ps(__m128 a, __m128 b, __m128 c);
__m128 _mm_fmsub_ps(__m128 a, __m128 b, __m128 c);
__m128 _mm_fnmadd_ps(__m128 a, __m128 b, __m128 c);
__m128 _mm_fnmsub_ps(__m128 a, __m128 b, __m128 c);
__m128 _mm_fmaddsub_ps(__m128 a, __m128 b, __m128 c);
__m128 _mm_fmsubadd_ps(__m128 a, __m128 b, __m128 c);
```

#### Float64 (128-bit)

```c
__m128d _mm_fmadd_pd(__m128d a, __m128d b, __m128d c);
__m128d _mm_fmsub_pd(__m128d a, __m128d b, __m128d c);
__m128d _mm_fnmadd_pd(__m128d a, __m128d b, __m128d c);
__m128d _mm_fnmsub_pd(__m128d a, __m128d b, __m128d c);
__m128d _mm_fmaddsub_pd(__m128d a, __m128d b, __m128d c);
__m128d _mm_fmsubadd_pd(__m128d a, __m128d b, __m128d c);
```

#### Scalar FMA (lowest lane only)

```c
__m128  _mm_fmadd_ss(__m128  a, __m128  b, __m128  c);
__m128d _mm_fmadd_sd(__m128d a, __m128d b, __m128d c);
__m128  _mm_fmsub_ss(__m128  a, __m128  b, __m128  c);
__m128d _mm_fmsub_sd(__m128d a, __m128d b, __m128d c);
__m128  _mm_fnmadd_ss(__m128  a, __m128  b, __m128  c);
__m128d _mm_fnmadd_sd(__m128d a, __m128d b, __m128d c);
__m128  _mm_fnmsub_ss(__m128  a, __m128  b, __m128  c);
__m128d _mm_fnmsub_sd(__m128d a, __m128d b, __m128d c);
```

### FMA Usage Patterns

```c
// ── Dot product of two float arrays ──────────────────────────────────────────
float dot_product_avx2_fma(const float* a, const float* b, int n) {
    __m256 sum = _mm256_setzero_ps();
    int i = 0;
    for (; i <= n - 8; i += 8) {
        __m256 va = _mm256_loadu_ps(&a[i]);
        __m256 vb = _mm256_loadu_ps(&b[i]);
        sum = _mm256_fmadd_ps(va, vb, sum);  // sum += va * vb
    }
    // Reduce 8-wide accumulator to scalar
    // (horizontal sum — see Section 13)
    __m128 hi  = _mm256_extractf128_ps(sum, 1);
    __m128 lo  = _mm256_castps256_ps128(sum);
    __m128 s   = _mm_add_ps(lo, hi);
    s = _mm_hadd_ps(s, s);
    s = _mm_hadd_ps(s, s);
    float result = _mm_cvtss_f32(s);
    // Handle remainder
    for (; i < n; i++) result += a[i] * b[i];
    return result;
}

// ── Polynomial evaluation (Horner's method with FMA) ─────────────────────────
// P(x) = c0 + c1*x + c2*x^2 + c3*x^3 = c0 + x*(c1 + x*(c2 + x*c3))
__m256 poly_eval_ps(__m256 x, float c0, float c1, float c2, float c3) {
    __m256 result = _mm256_set1_ps(c3);
    result = _mm256_fmadd_ps(result, x, _mm256_set1_ps(c2));
    result = _mm256_fmadd_ps(result, x, _mm256_set1_ps(c1));
    result = _mm256_fmadd_ps(result, x, _mm256_set1_ps(c0));
    return result;
}

// ── 4×4 matrix × vector (float) ──────────────────────────────────────────────
// Broadcast each element of the 4-element vector and FMA with matrix rows
void mat4_vec4_mul(const float* M, const float* v, float* out) {
    __m128 row0 = _mm_loadu_ps(&M[0]);
    __m128 row1 = _mm_loadu_ps(&M[4]);
    __m128 row2 = _mm_loadu_ps(&M[8]);
    __m128 row3 = _mm_loadu_ps(&M[12]);
    __m128 v0 = _mm_set1_ps(v[0]);
    __m128 v1 = _mm_set1_ps(v[1]);
    __m128 v2 = _mm_set1_ps(v[2]);
    __m128 v3 = _mm_set1_ps(v[3]);
    __m128 result = _mm_fmadd_ps(row3, v3,
                    _mm_fmadd_ps(row2, v2,
                    _mm_fmadd_ps(row1, v1,
                    _mm_mul_ps(row0, v0))));
    _mm_storeu_ps(out, result);
}

// ── Complex multiplication with fmaddsub ─────────────────────────────────────
// (a+bi)(c+di) = (ac-bd) + (ad+bc)i
// Data layout: [re0, im0, re1, im1, re2, im2, re3, im3]
__m256 complex_mul_ps(__m256 a, __m256 b) {
    // Duplicate: [re0,re0,re1,re1,...] and [im0,im0,im1,im1,...]
    __m256 a_re = _mm256_moveldup_ps(a);   // [re0,re0,re1,re1,re2,re2,re3,re3]
    __m256 a_im = _mm256_movehdup_ps(a);   // [im0,im0,im1,im1,im2,im2,im3,im3]
    // Swap re/im of b: [im0,re0,im1,re1,...]
    __m256 b_swapped = _mm256_permute_ps(b, 0xB1);  // 0xB1 = 10110001b
    // (a_re * b)  +-  (a_im * b_swapped)
    return _mm256_fmaddsub_ps(a_re, b, _mm256_mul_ps(a_im, b_swapped));
    // even lanes: a_re*b_re - a_im*b_im  → real part
    // odd  lanes: a_re*b_im + a_im*b_re  → imaginary part
}
```

### FMA Performance Notes

| CPU Microarchitecture | FMA Latency | FMA Throughput |
|-----------------------|-------------|----------------|
| Intel Haswell (2013)  | 5 cycles    | 0.5 CPI (2/cycle) |
| Intel Skylake (2015)  | 4 cycles    | 0.5 CPI (2/cycle) |
| Intel Ice Lake (2019) | 4 cycles    | 0.5 CPI (2/cycle) |
| AMD Zen 2 (2019)      | 5 cycles    | 0.5 CPI (2/cycle) |
| AMD Zen 3 (2020)      | 4 cycles    | 0.25 CPI (4/cycle) |

Theoretical peak FLOP/s with FMA: `2 × lanes × frequency × (FMA ops/cycle)`

For Zen 3 @ 4 GHz: `2 × 8 × 4×10^9 × 4 = 256 GFLOP/s` (per core, float32)

---

## 9. Logical & Bitwise Operations

### Floating-Point Bitwise Operations

```c
// AND
__m256  _mm256_and_ps (__m256  a, __m256  b);
__m256d _mm256_and_pd (__m256d a, __m256d b);

// AND NOT: (~a) & b
__m256  _mm256_andnot_ps (__m256  a, __m256  b);
__m256d _mm256_andnot_pd (__m256d a, __m256d b);

// OR
__m256  _mm256_or_ps  (__m256  a, __m256  b);
__m256d _mm256_or_pd  (__m256d a, __m256d b);

// XOR
__m256  _mm256_xor_ps (__m256  a, __m256  b);
__m256d _mm256_xor_pd (__m256d a, __m256d b);
```

### Integer Bitwise Operations

```c
__m256i _mm256_and_si256   (__m256i a, __m256i b);   // a & b
__m256i _mm256_andnot_si256(__m256i a, __m256i b);   // (~a) & b
__m256i _mm256_or_si256    (__m256i a, __m256i b);   // a | b
__m256i _mm256_xor_si256   (__m256i a, __m256i b);   // a ^ b
```

### Integer Shifts

```c
// ── Logical left shift (shift left, fill with zeros) ──────────────────────────
__m256i _mm256_slli_epi16(__m256i a, int imm8);  // a[i] << imm8, 16-bit lanes
__m256i _mm256_slli_epi32(__m256i a, int imm8);  // a[i] << imm8, 32-bit lanes
__m256i _mm256_slli_epi64(__m256i a, int imm8);  // a[i] << imm8, 64-bit lanes

// Variable shift (different count per lane — AVX2)
__m256i _mm256_sllv_epi32(__m256i a, __m256i count);  // a[i] << count[i], 32-bit
__m256i _mm256_sllv_epi64(__m256i a, __m256i count);  // a[i] << count[i], 64-bit

// Shift entire 256-bit register left by imm8 BYTES
__m256i _mm256_slli_si256(__m256i a, int imm8);       // Byte-granularity shift (within 128-bit lanes!)

// ── Logical right shift (shift right, fill with zeros) ────────────────────────
__m256i _mm256_srli_epi16(__m256i a, int imm8);
__m256i _mm256_srli_epi32(__m256i a, int imm8);
__m256i _mm256_srli_epi64(__m256i a, int imm8);
__m256i _mm256_srlv_epi32(__m256i a, __m256i count);
__m256i _mm256_srlv_epi64(__m256i a, __m256i count);
__m256i _mm256_srli_si256(__m256i a, int imm8);       // Byte shift within 128-bit lanes

// ── Arithmetic right shift (shift right, sign-extend) ─────────────────────────
__m256i _mm256_srai_epi16(__m256i a, int imm8);  // signed >>
__m256i _mm256_srai_epi32(__m256i a, int imm8);
__m256i _mm256_srav_epi32(__m256i a, __m256i count);  // variable arithmetic shift (AVX2)
// No 64-bit arithmetic shift in AVX2 (added in AVX-512)
```

> **Warning on `_mm256_slli_si256` / `_mm256_srli_si256`**: These operate *within each 128-bit lane independently*, NOT across the full 256-bit vector. This is a common source of bugs.

```c
// Example pitfall:
__m256i v = ...; // contains [7,6,5,4,3,2,1,0] as int32
__m256i shifted = _mm256_slli_si256(v, 4);  // 4-byte = 1 int32 shift
// Result: [6,5,4,0, 2,1,0,0]  ← NOT [6,5,4,3,2,1,0,0]
// The 128-bit lane boundary (between index 3 and 4) blocks carry-over!
```

### Bit Manipulation

```c
// Population count (number of set bits) — not AVX2 but widely used alongside:
int _mm_popcnt_u32(unsigned int a);    // popcnt for 32-bit integer
int _mm_popcnt_u64(unsigned long long a);  // popcnt for 64-bit integer

// There is no SIMD popcnt in AVX2. Use AVX-512 _mm256_popcnt_epi8 or
// a VPSHUFB-based lookup table approach (see patterns section).
```

---

## 10. Comparison Operations

### Floating-Point Comparisons

```c
// ── 256-bit packed comparisons: return bitmask (all 1s or all 0s per lane) ────
// imm8 selects comparison predicate (see table below)
__m256  _mm256_cmp_ps(__m256  a, __m256  b, const int imm8);  // 8 × float32
__m256d _mm256_cmp_pd(__m256d a, __m256d b, const int imm8);  // 4 × float64

// 128-bit equivalents
__m128  _mm_cmp_ps(__m128  a, __m128  b, const int imm8);
__m128d _mm_cmp_pd(__m128d a, __m128d b, const int imm8);
__m128  _mm_cmp_ss(__m128  a, __m128  b, const int imm8);
__m128d _mm_cmp_sd(__m128d a, __m128d b, const int imm8);
```

#### Comparison Predicates (imm8 values)

| imm8 | Mnemonic | Description | NaN behavior |
|------|----------|-------------|--------------|
| 0x00 | `_CMP_EQ_OQ`   | Equal (ordered, quiet)           | Returns false if NaN |
| 0x01 | `_CMP_LT_OS`   | Less-than (ordered, signaling)   | Signals on NaN |
| 0x02 | `_CMP_LE_OS`   | Less-equal (ordered, signaling)  | Signals on NaN |
| 0x03 | `_CMP_UNORD_Q` | Unordered (quiet)                | Returns true if NaN |
| 0x04 | `_CMP_NEQ_UQ`  | Not-equal (unordered, quiet)     | Returns true if NaN |
| 0x05 | `_CMP_NLT_US`  | Not-less-than (unordered, sig)   | Returns true if NaN |
| 0x06 | `_CMP_NLE_US`  | Not-less-equal (unordered, sig)  | Returns true if NaN |
| 0x07 | `_CMP_ORD_Q`   | Ordered (quiet)                  | Returns false if NaN |
| 0x08 | `_CMP_EQ_UQ`   | Equal (unordered, quiet)         | Returns true if NaN |
| 0x09 | `_CMP_NGE_US`  | Not-greater-equal                | Returns true if NaN |
| 0x0A | `_CMP_NGT_US`  | Not-greater-than                 | Returns true if NaN |
| 0x0B | `_CMP_FALSE_OQ`| Always false                     | — |
| 0x0C | `_CMP_NEQ_OQ`  | Not-equal (ordered, quiet)       | Returns false if NaN |
| 0x0D | `_CMP_GE_OS`   | Greater-equal (ordered, sig)     | Signals on NaN |
| 0x0E | `_CMP_GT_OS`   | Greater-than (ordered, sig)      | Signals on NaN |
| 0x0F | `_CMP_TRUE_UQ` | Always true                      | — |

```c
// Example: element-wise greater-than comparison
__m256 gt_mask = _mm256_cmp_ps(a, b, _CMP_GT_OS);
// Each lane of gt_mask = 0xFFFFFFFF where a[i] > b[i], else 0x00000000

// Use the mask to select: result[i] = a[i] > b[i] ? a[i] : b[i]
__m256 result = _mm256_blendv_ps(b, a, gt_mask);  // equivalent to max
```

### Integer Comparisons

```c
// ── Equal ─────────────────────────────────────────────────────────────────────
__m256i _mm256_cmpeq_epi8 (__m256i a, __m256i b);  // 32 × int8:  a[i]==b[i] → 0xFF or 0x00
__m256i _mm256_cmpeq_epi16(__m256i a, __m256i b);  // 16 × int16
__m256i _mm256_cmpeq_epi32(__m256i a, __m256i b);  //  8 × int32
__m256i _mm256_cmpeq_epi64(__m256i a, __m256i b);  //  4 × int64

// ── Greater-than (signed only — no unsigned compare in AVX2) ─────────────────
__m256i _mm256_cmpgt_epi8 (__m256i a, __m256i b);  // a[i] > b[i] (signed)
__m256i _mm256_cmpgt_epi16(__m256i a, __m256i b);
__m256i _mm256_cmpgt_epi32(__m256i a, __m256i b);
__m256i _mm256_cmpgt_epi64(__m256i a, __m256i b);

// ── Unsigned compare workaround ───────────────────────────────────────────────
// To compare unsigned: flip the sign bit, then use signed compare
__m256i unsigned_cmpgt_epi32(__m256i a, __m256i b) {
    __m256i bias = _mm256_set1_epi32(0x80000000);
    return _mm256_cmpgt_epi32(_mm256_xor_si256(a, bias),
                               _mm256_xor_si256(b, bias));
}
```

### Converting Comparison Masks

```c
// Convert floating-point compare mask to integer bitmask (one bit per lane)
int _mm256_movemask_ps(__m256 a);   // 8-bit mask, bit i set if lane i MSB is set
int _mm256_movemask_pd(__m256d a);  // 4-bit mask

// 128-bit equivalents
int _mm_movemask_ps(__m128 a);
int _mm_movemask_pd(__m128d a);
int _mm_movemask_epi8(__m128i a);  // 16-bit mask, one bit per byte lane

// Example: check if ANY lane satisfies condition
__m256 cmp = _mm256_cmp_ps(a, _mm256_setzero_ps(), _CMP_LT_OS);
int mask = _mm256_movemask_ps(cmp);
if (mask != 0) {
    // At least one element is negative
}

// Check if ALL lanes satisfy condition
if (mask == 0xFF) {
    // All 8 elements are negative
}
```

---

## 11. Shuffle, Permute & Blend

These operations rearrange data within vectors — critical for transposition, interleaving, and data layout transformations.

### Blend Operations

```c
// ── blendv: blend based on runtime mask (MSB of each lane) ───────────────────
__m256  _mm256_blendv_ps(__m256  a, __m256  b, __m256  mask);
// result[i] = (mask[i] MSB set) ? b[i] : a[i]
__m256d _mm256_blendv_pd(__m256d a, __m256d b, __m256d mask);
__m256i _mm256_blendv_epi8(__m256i a, __m256i b, __m256i mask);

// ── blend: blend based on immediate (compile-time constant) ───────────────────
__m256  _mm256_blend_ps(__m256  a, __m256  b, const int imm8);
// imm8 = 8-bit constant, bit i: 0=select from a, 1=select from b
__m256d _mm256_blend_pd(__m256d a, __m256d b, const int imm8);
__m256i _mm256_blend_epi16(__m256i a, __m256i b, const int imm8);
__m256i _mm256_blend_epi32(__m256i a, __m256i b, const int imm8);  // AVX2

// Example: select alternating elements (a0,b1,a2,b3,...):
__m256 result = _mm256_blend_ps(a, b, 0b10101010);  // 0xAA = 10101010
```

### Permute Within 128-bit Lanes (ps)

```c
// Permute float32 within each 128-bit lane (4 elements per lane)
__m256 _mm256_permute_ps(__m256 a, int imm8);
// imm8: 4 fields of 2 bits, each selecting a source lane WITHIN the 128-bit lane
// imm8 = [sel3:sel2:sel1:sel0], selN ∈ {0,1,2,3}
// Example: reverse order within each 128-bit lane:
__m256 rev = _mm256_permute_ps(a, 0x1B);  // 0x1B = 00011011 = [0,1,2,3]→[3,2,1,0]

// Permute float64 within each 128-bit lane (2 elements per lane)
__m256d _mm256_permute_pd(__m256d a, int imm8);
// imm8: 4 bits, each bit selects index 0 or 1 within each pair
```

### Permute Across Full 256-bit (Cross-lane)

```c
// permute2f128: combine 128-bit chunks from two 256-bit registers
__m256  _mm256_permute2f128_ps(__m256  a, __m256  b, int imm8);
__m256d _mm256_permute2f128_pd(__m256d a, __m256d b, int imm8);
__m256i _mm256_permute2x128_si256(__m256i a, __m256i b, int imm8);
// imm8 layout: [high_src_hi:high_src_lo | low_src_hi:low_src_lo]
//   bits [1:0]: src for low 128-bit output:  0=a_lo, 1=a_hi, 2=b_lo, 3=b_hi
//   bits [3:2]: same for high 128-bit output
//   bit  4:    zero low 128-bit output
//   bit  7:    zero high 128-bit output

// Example: swap low and high 128-bit halves
__m256 swapped = _mm256_permute2f128_ps(a, a, 0x01);  // b = a (unused)

// permutevar: integer-indexed permute (each lane selects an index)
__m256  _mm256_permutevar_ps(__m256  a, __m256i idx);   // idx[i] selects within 128-bit lane
__m256d _mm256_permutevar_pd(__m256d a, __m256i idx);

// permutevar8x32: permute 32-bit integers across FULL 256 bits (AVX2)
__m256i _mm256_permutevar8x32_epi32(__m256i a, __m256i idx);
__m256  _mm256_permutevar8x32_ps(__m256 a, __m256i idx);
// idx[i] ∈ {0..7}, selects from any of the 8 int32 lanes (true cross-lane permute!)
```

### Shuffle Operations

```c
// ── shuffle_ps: select float32 elements across two sources ───────────────────
__m256 _mm256_shuffle_ps(__m256 a, __m256 b, const int imm8);
// imm8 = [bits 7:6 = b_hi | bits 5:4 = b_lo | bits 3:2 = a_hi | bits 1:0 = a_lo]
// Each pair of bits selects index within the respective 128-bit lane of a or b
// Acts independently in each 128-bit lane

__m256d _mm256_shuffle_pd(__m256d a, __m256d b, const int imm8);

// ── pshufb: byte shuffle with per-lane control (SSSE3, extended to 256-bit in AVX2)
__m256i _mm256_shuffle_epi8(__m256i a, __m256i mask);
// For each byte lane i (0–31): result[i] = (mask[i] bit 7 set) ? 0 : a[mask[i] & 0x0F]
// IMPORTANT: operates independently within each 128-bit lane — mask index is relative to lane start!

// ── shuffle integer elements ──────────────────────────────────────────────────
__m256i _mm256_shuffle_epi32(__m256i a, const int imm8);
// imm8 selects 4 int32 elements within each 128-bit lane (same format as _mm256_permute_ps)

// Shuffle 16-bit elements within each 64-bit group (8 elements per half)
__m256i _mm256_shufflelo_epi16(__m256i a, const int imm8);  // shuffle low 4 of each lane
__m256i _mm256_shufflehi_epi16(__m256i a, const int imm8);  // shuffle high 4 of each lane
```

### Unpack / Interleave

```c
// ── Interleave low half of two vectors ───────────────────────────────────────
__m256i _mm256_unpacklo_epi8 (__m256i a, __m256i b);   // interleave low bytes
__m256i _mm256_unpacklo_epi16(__m256i a, __m256i b);
__m256i _mm256_unpacklo_epi32(__m256i a, __m256i b);
__m256i _mm256_unpacklo_epi64(__m256i a, __m256i b);
__m256  _mm256_unpacklo_ps(__m256 a, __m256 b);
__m256d _mm256_unpacklo_pd(__m256d a, __m256d b);

// ── Interleave high half of two vectors ──────────────────────────────────────
__m256i _mm256_unpackhi_epi8 (__m256i a, __m256i b);
__m256i _mm256_unpackhi_epi16(__m256i a, __m256i b);
__m256i _mm256_unpackhi_epi32(__m256i a, __m256i b);
__m256i _mm256_unpackhi_epi64(__m256i a, __m256i b);
__m256  _mm256_unpackhi_ps(__m256 a, __m256 b);
__m256d _mm256_unpackhi_pd(__m256d a, __m256d b);

// Example: _mm256_unpacklo_ps with a = [a7,a6,a5,a4,a3,a2,a1,a0]
//                                    b = [b7,b6,b5,b4,b3,b2,b1,b0]
// Result:  [b5,a5,b4,a4, b1,a1,b0,a0]  (interleaved low of each 128-bit lane)
```

### Move Duplicate / Broadcast within Vector

```c
__m256  _mm256_moveldup_ps(__m256 a);   // duplicate even (lower) lanes: [a6,a6,a4,a4,a2,a2,a0,a0]
__m256  _mm256_movehdup_ps(__m256 a);   // duplicate odd (upper) lanes:  [a7,a7,a5,a5,a3,a3,a1,a1]
__m256d _mm256_movedup_pd(__m256d a);   // duplicate even lanes: [a2,a2,a0,a0]
```

### Insert / Extract 128-bit Chunks

```c
// Insert 128-bit chunk into 256-bit register
__m256  _mm256_insertf128_ps (__m256  a, __m128  b, int imm8);  // imm8=0: low, imm8=1: high
__m256d _mm256_insertf128_pd (__m256d a, __m128d b, int imm8);
__m256i _mm256_inserti128_si256(__m256i a, __m128i b, int imm8);

// Extract 128-bit chunk from 256-bit register
__m128  _mm256_extractf128_ps (__m256  a, int imm8);
__m128d _mm256_extractf128_pd (__m256d a, int imm8);
__m128i _mm256_extracti128_si256(__m256i a, int imm8);

// Zero-cost casts (just reinterpret)
__m128  _mm256_castps256_ps128(__m256 a);  // Extract low 128 bits, no instruction
__m256  _mm256_castps128_ps256(__m128 a);  // Extend to 256 (upper bits undefined!)
```

### 4×4 Float Transpose Example

```c
// Transpose a 4×4 float matrix using unpack + shuffle
void transpose_4x4_ps(__m128 row0, __m128 row1, __m128 row2, __m128 row3,
                       __m128* out0, __m128* out1, __m128* out2, __m128* out3) {
    __m128 t0 = _mm_unpacklo_ps(row0, row1);  // [r01,r00,r11,r10] → actually [r10,r00,r11,r01]
    __m128 t1 = _mm_unpacklo_ps(row2, row3);
    __m128 t2 = _mm_unpackhi_ps(row0, row1);
    __m128 t3 = _mm_unpackhi_ps(row2, row3);
    *out0 = _mm_movelh_ps(t0, t1);  // [t1[1:0], t0[1:0]]
    *out1 = _mm_movehl_ps(t1, t0);  // [t0[3:2], t1[3:2]]
    *out2 = _mm_movelh_ps(t2, t3);
    *out3 = _mm_movehl_ps(t3, t2);
}
```

---

## 12. Conversion Intrinsics

### Float ↔ Integer Conversions

```c
// ── Float32 to Integer32 ──────────────────────────────────────────────────────
__m256i _mm256_cvtps_epi32(__m256 a);      // Round to nearest (current rounding mode)
__m256i _mm256_cvttps_epi32(__m256 a);     // Truncate toward zero (fast — no rounding)

// ── Integer32 to Float32 ──────────────────────────────────────────────────────
__m256  _mm256_cvtepi32_ps(__m256i a);     // int32 → float32

// ── Float64 to Integer32 (result in 128-bit) ──────────────────────────────────
__m128i _mm256_cvtpd_epi32(__m256d a);     // 4×double → 4×int32 (with rounding)
__m128i _mm256_cvttpd_epi32(__m256d a);    // Truncate

// ── Float64 to Float32 (256→128) ─────────────────────────────────────────────
__m128  _mm256_cvtpd_ps(__m256d a);        // 4×double → 4×float32

// ── Float32 to Float64 (128→256) ─────────────────────────────────────────────
__m256d _mm256_cvtps_pd(__m128 a);         // 4×float32 → 4×double

// ── 128-bit float conversions ─────────────────────────────────────────────────
__m128i _mm_cvtps_epi32(__m128 a);         // 4×float32 → 4×int32
__m128i _mm_cvttps_epi32(__m128 a);        // truncate
__m128  _mm_cvtepi32_ps(__m128i a);        // int32 → float32
float   _mm_cvtss_f32(__m128 a);           // Extract lowest float32 lane to scalar
double  _mm_cvtsd_f64(__m128d a);          // Extract lowest float64 lane to scalar
int     _mm_cvtsi128_si32(__m128i a);      // Extract lowest int32 lane
```

### Integer Width Conversions (Sign Extension and Zero Extension)

```c
// ── Sign-extend smaller integers to larger ones ───────────────────────────────
// All these are in SSE4.1 for 128-bit, extended to 256-bit with AVX2

// 8→16
__m256i _mm256_cvtepi8_epi16(__m128i a);   // 16×int8 → 16×int16 (sign extend)
// 8→32
__m256i _mm256_cvtepi8_epi32(__m128i a);   // 8×int8 → 8×int32 (sign extend)
// 8→64
__m256i _mm256_cvtepi8_epi64(__m128i a);   // 4×int8 → 4×int64 (sign extend)
// 16→32
__m256i _mm256_cvtepi16_epi32(__m128i a);  // 8×int16 → 8×int32 (sign extend)
// 16→64
__m256i _mm256_cvtepi16_epi64(__m128i a);  // 4×int16 → 4×int64 (sign extend)
// 32→64
__m256i _mm256_cvtepi32_epi64(__m128i a);  // 4×int32 → 4×int64 (sign extend)

// ── Zero-extend smaller integers ─────────────────────────────────────────────
__m256i _mm256_cvtepu8_epi16(__m128i a);   // 16×uint8 → 16×int16 (zero extend)
__m256i _mm256_cvtepu8_epi32(__m128i a);   // 8×uint8 → 8×int32 (zero extend)
__m256i _mm256_cvtepu8_epi64(__m128i a);   // 4×uint8 → 4×int64 (zero extend)
__m256i _mm256_cvtepu16_epi32(__m128i a);  // 8×uint16 → 8×int32 (zero extend)
__m256i _mm256_cvtepu16_epi64(__m128i a);  // 4×uint16 → 4×int64 (zero extend)
__m256i _mm256_cvtepu32_epi64(__m128i a);  // 4×uint32 → 4×int64 (zero extend)
```

### Pack — Reduce Width with Saturation

```c
// Pack 16-bit → 8-bit with signed saturation
__m256i _mm256_packs_epi16(__m256i a, __m256i b);   // 32×int16 → 32×int8, signed sat
// Pack 16-bit → 8-bit with unsigned saturation
__m256i _mm256_packus_epi16(__m256i a, __m256i b);  // → uint8, unsigned sat

// Pack 32-bit → 16-bit with signed saturation
__m256i _mm256_packs_epi32(__m256i a, __m256i b);   // 16×int32 → 16×int16, signed sat
// Pack 32-bit → 16-bit with unsigned saturation
__m256i _mm256_packus_epi32(__m256i a, __m256i b);  // → uint16, unsigned sat

// WARNING: packs operates within 128-bit lanes, with interleaving.
// Output for packs_epi16(a, b) = [a_hi_packed, b_hi_packed, a_lo_packed, b_lo_packed]
// (not simply concatenated — elements interleave at the lane boundary)
```

### Type Cast (Zero-Cost Reinterpret)

```c
// These generate NO instructions — just tell the compiler to treat bits differently
__m256i _mm256_castps_si256(__m256  a);
__m256  _mm256_castsi256_ps(__m256i a);
__m256i _mm256_castpd_si256(__m256d a);
__m256d _mm256_castsi256_pd(__m256i a);
__m256  _mm256_castpd_ps(__m256d a);
__m256d _mm256_castps_pd(__m256  a);
// And between 128/256-bit:
__m256  _mm256_castps128_ps256(__m128 a);   // Upper 128 bits UNDEFINED
__m128  _mm256_castps256_ps128(__m256 a);   // Extract lower 128 bits
```

---

## 13. Horizontal Operations

Horizontal operations reduce across lanes — e.g., summing all elements. They are generally *slower* than vertical operations (which operate lane-by-lane). Use sparingly, or restructure your algorithm to use vertical ops instead.

### Horizontal Add/Subtract

```c
// Horizontal add float32 (adjacent pairs within 128-bit lanes)
__m256  _mm256_hadd_ps(__m256  a, __m256  b);
// [a3+a2, a1+a0, b3+b2, b1+b0, a7+a6, a5+a4, b7+b6, b5+b4]

__m256d _mm256_hadd_pd(__m256d a, __m256d b);
// [a1+a0, b1+b0, a3+a2, b3+b2]

__m256  _mm256_hsub_ps(__m256  a, __m256  b);
__m256d _mm256_hsub_pd(__m256d a, __m256d b);

// 128-bit versions
__m128  _mm_hadd_ps(__m128  a, __m128  b);
__m128d _mm_hadd_pd(__m128d a, __m128d b);

// Integer horizontal add (16-bit → 32-bit)
__m256i _mm256_hadd_epi16(__m256i a, __m256i b);  // 16 pairs of int16 → 8 int32? No:
// Correct: 16×int16 adjacent pairs → 8×int16 sums (saturating for epi16)
// Actually: horizontal add of adjacent int16 pairs → result is int16 with wrapping
__m256i _mm256_hadd_epi32(__m256i a, __m256i b);  // adjacent int32 pairs → int32 sums
__m256i _mm256_hsub_epi16(__m256i a, __m256i b);
__m256i _mm256_hsub_epi32(__m256i a, __m256i b);

// Saturating integer horizontal add
__m256i _mm256_hadds_epi16(__m256i a, __m256i b);  // signed saturating hadd int16
```

### Dot Product (SSE4.1, 128-bit)

```c
// Compute partial dot products and store in selected output lanes
__m128  _mm_dp_ps(__m128 a, __m128 b, const int imm8);
// imm8 bits [7:4]: which input lanes participate in multiply
// imm8 bits [3:0]: which output lanes receive the sum
// Example: full 4-element dot product broadcast to all output lanes:
float dp = _mm_cvtss_f32(_mm_dp_ps(a, b, 0xFF));

__m128d _mm_dp_pd(__m128d a, __m128d b, const int imm8);
// Note: No 256-bit dp intrinsic. Split into two 128-bit dp ops or use hadd.
```

### Efficient Horizontal Sum (Float)

```c
// Best practice for 256-bit horizontal sum:
float hsum_ps(__m256 v) {
    __m128 lo = _mm256_castps256_ps128(v);      // low 128 bits
    __m128 hi = _mm256_extractf128_ps(v, 1);    // high 128 bits
    lo = _mm_add_ps(lo, hi);                    // add the two halves
    // Now do 128-bit horizontal sum
    __m128 shuf = _mm_movehdup_ps(lo);          // [3,3,1,1]
    __m128 sums = _mm_add_ps(lo, shuf);         // [3+2, 3+2, 1+0, 1+0]
    shuf = _mm_movehl_ps(shuf, sums);           // [3+2, 3+2, 3+2, 1+0]
    sums = _mm_add_ss(sums, shuf);              // total
    return _mm_cvtss_f32(sums);
}
```

---

## 14. Gather Operations

Gather loads data from non-contiguous memory addresses specified by a vector of indices.

> **Performance warning**: Gather instructions have high latency (~20–35 cycles) and are often slower than scalar loads for small vectors. Profile before committing to gather.

### Integer-Indexed Gathers

```c
// Gather int32 from base + index[i]*scale
__m128i _mm_i32gather_epi32 (int const* base, __m128i vindex, const int scale);
__m256i _mm256_i32gather_epi32(int const* base, __m256i vindex, const int scale);

// Gather int64 from base + index[i]*scale
__m128i _mm_i32gather_epi64 (long long const* base, __m128i vindex, const int scale);
__m256i _mm256_i32gather_epi64(long long const* base, __m128i vindex, const int scale);
__m128i _mm_i64gather_epi64 (long long const* base, __m128i vindex, const int scale);
__m256i _mm256_i64gather_epi64(long long const* base, __m256i vindex, const int scale);

// scale must be a compile-time constant: 1, 2, 4, or 8 (byte multiplier)
// Effective address = base + vindex[i] * scale
```

### Float Gathers

```c
__m128  _mm_i32gather_ps (float const* base, __m128i vindex, const int scale);
__m256  _mm256_i32gather_ps(float const* base, __m256i vindex, const int scale);
__m128d _mm_i32gather_pd (double const* base, __m128i vindex, const int scale);
__m256d _mm256_i32gather_pd(double const* base, __m128i vindex, const int scale);
__m128  _mm_i64gather_ps (float const* base, __m128i vindex, const int scale);
__m256  _mm256_i64gather_ps(float const* base, __m256i vindex, const int scale);
__m128d _mm_i64gather_pd (double const* base, __m128i vindex, const int scale);
__m256d _mm256_i64gather_pd(double const* base, __m256i vindex, const int scale);
```

### Masked Gathers

```c
// Masked gather: only load lanes where mask MSB is set; others get src value
__m128i _mm_mask_i32gather_epi32 (__m128i src, int const* base, __m128i vi, __m128i mask, const int scale);
__m256i _mm256_mask_i32gather_epi32(__m256i src, int const* base, __m256i vi, __m256i mask, const int scale);
__m128  _mm_mask_i32gather_ps     (__m128  src, float const* base, __m128i vi, __m128  mask, const int scale);
__m256  _mm256_mask_i32gather_ps  (__m256  src, float const* base, __m256i vi, __m256  mask, const int scale);
// (and corresponding _i64gather variants...)
```

### Gather Usage Example

```c
// Access a sparse array: y[i] = x[idx[i]]
void sparse_gather(const float* x, const int* idx, float* y, int n) {
    for (int i = 0; i <= n - 8; i += 8) {
        __m256i vindex = _mm256_loadu_si256((__m256i*)&idx[i]);
        __m256  gathered = _mm256_i32gather_ps(x, vindex, 4);  // scale=4 for float (4 bytes)
        _mm256_storeu_ps(&y[i], gathered);
    }
    for (int i = (n & ~7); i < n; i++) y[i] = x[idx[i]];
}
```

---

## 15. String & Mask Operations

### PCMPESTRI / PCMPESTRM — Packed Compare Explicit-Length Strings

Available in SSE4.2 and usable with 128-bit XMM:

```c
// Return index of first match / mask of matches
int   _mm_cmpestri(__m128i a, int la, __m128i b, int lb, const int imm8);
__m128i _mm_cmpestrm(__m128i a, int la, __m128i b, int lb, const int imm8);

// imm8 controls: element type (byte/word), operation (ranges/chars/substring/equal any), output format
// Key constants for imm8:
#define _SIDD_UBYTE_OPS   0x00  // Unsigned 8-bit characters
#define _SIDD_UWORD_OPS   0x01  // Unsigned 16-bit characters
#define _SIDD_SBYTE_OPS   0x02  // Signed 8-bit
#define _SIDD_SWORD_OPS   0x03  // Signed 16-bit
#define _SIDD_CMP_EQUAL_ANY  0x00  // String contains any char from set
#define _SIDD_CMP_RANGES     0x04  // Characters fall within ranges
#define _SIDD_CMP_EQUAL_EACH 0x08  // Element-wise equality (substring)
#define _SIDD_CMP_EQUAL_ORDERED 0x0C  // Ordered substring search
#define _SIDD_POSITIVE_POLARITY 0x00
#define _SIDD_NEGATIVE_POLARITY 0x10
#define _SIDD_LEAST_SIGNIFICANT 0x00  // Index: return index of first match
#define _SIDD_MOST_SIGNIFICANT  0x40  // Index: return index of last match
#define _SIDD_BIT_MASK          0x00  // Mask: return bitmask
#define _SIDD_UNIT_MASK         0x40  // Mask: return byte/word mask

// PCMPISTRI/PCMPISTRM — implicit-length (null-terminated strings)
int     _mm_cmpistri(__m128i a, __m128i b, const int imm8);
__m128i _mm_cmpistrm(__m128i a, __m128i b, const int imm8);
```

### Masked Load/Store (AVX2)

```c
// maskload: load from memory, but only for lanes where mask MSB is set
__m256i _mm256_maskload_epi32(int   const* mem_addr, __m256i mask);
__m256i _mm256_maskload_epi64(long long const* mem_addr, __m256i mask);
__m256  _mm256_maskload_ps   (float  const* mem_addr, __m256i mask);
__m256d _mm256_maskload_pd   (double const* mem_addr, __m256i mask);

// maskstore: only store lanes where mask MSB is set
void _mm256_maskstore_epi32(int*   mem_addr, __m256i mask, __m256i a);
void _mm256_maskstore_epi64(long long* mem_addr, __m256i mask, __m256i a);
void _mm256_maskstore_ps   (float*  mem_addr, __m256i mask, __m256  a);
void _mm256_maskstore_pd   (double* mem_addr, __m256i mask, __m256d a);
```

---

## 16. Integer Arithmetic Deep Dive

### Sum of Absolute Differences (SAD)

```c
// Compute SAD between 8-bit unsigned integers: used in video motion estimation
__m256i _mm256_sad_epu8(__m256i a, __m256i b);
// For each 64-bit group: sum of |a[i] - b[i]| for 8 bytes → single uint64 result per group
// Output: 4 × uint64, each holding sum of 8 byte-differences in its group

// 128-bit version:
__m128i _mm_sad_epu8(__m128i a, __m128i b);
// 2 × uint64 outputs

// Multi-sum SAD: sum of absolute differences for 4-byte sub-blocks
// (SSE4.1, not extended to 256-bit in AVX2)
__m128i _mm_mpsadbw_epu8(__m128i a, __m128i b, const int imm8);
```

### Sign and Signum

```c
// sign: multiply by sign of another vector
__m256i _mm256_sign_epi8 (__m256i a, __m256i b);   // a[i] * sign(b[i]): -a, 0, or +a
__m256i _mm256_sign_epi16(__m256i a, __m256i b);
__m256i _mm256_sign_epi32(__m256i a, __m256i b);
// b[i] > 0 → result = a[i]; b[i] == 0 → result = 0; b[i] < 0 → result = -a[i]
```

### Multiply-Add Saturating

```c
// Multiply signed 8-bit values and accumulate as signed 16-bit with saturation
__m256i _mm256_maddubs_epi16(__m256i a, __m256i b);
// a = unsigned int8, b = signed int8
// result[i] = sat16(a[2i]*b[2i] + a[2i+1]*b[2i+1])
// Extremely useful for INT8 neural network inference
```

### Integer Division (No Direct SIMD Divide)

```c
// There is NO integer divide intrinsic in AVX2.
// Options:
// 1. Convert to float, divide, convert back (loses precision for large integers)
// 2. Use reciprocal multiplication (for constant divisors)
// 3. Intel SVML (Short Vector Math Library): _mm256_div_epi32 (not standard)
// 4. Scalar fallback

// Divide by power of 2: use arithmetic shift right
__m256i div_by_4_epi32(__m256i a) {
    return _mm256_srai_epi32(a, 2);  // a >> 2 (arithmetic = correct for signed)
}

// Reciprocal multiply trick (e.g. divide by 7):
// precompute: magic = ceil(2^32 / 7) = 613566757
__m256i div_by_7_u32(__m256i v) {
    __m256i magic = _mm256_set1_epi32(613566757);
    // Multiply (taking high 32 bits of 64-bit product)
    __m256i lo = _mm256_srli_epi64(_mm256_mul_epu32(v, magic), 32);
    __m256i hi = _mm256_mul_epu32(_mm256_srli_epi64(v, 32), magic);
    // ... (Hacker's Delight division algorithm)
    // In practice: use a library or compiler auto-optimization
}
```

---

## 17. Floating-Point Special Operations

### Floor, Ceil, Round, Truncate

```c
// Round using rounding mode immediate
__m256  _mm256_round_ps(__m256 a, int rounding);
__m256d _mm256_round_pd(__m256d a, int rounding);
__m128  _mm_round_ps(__m128  a, int rounding);
__m128d _mm_round_pd(__m128d a, int rounding);

// rounding constants:
#define _MM_FROUND_TO_NEAREST_INT  0x00  // Round to nearest even
#define _MM_FROUND_TO_NEG_INF      0x01  // Floor
#define _MM_FROUND_TO_POS_INF      0x02  // Ceil
#define _MM_FROUND_TO_ZERO         0x03  // Truncate
#define _MM_FROUND_CUR_DIRECTION   0x04  // Use current MXCSR rounding mode
#define _MM_FROUND_NO_EXC          0x08  // Suppress precision exception
// Combine: e.g. _MM_FROUND_TO_NEG_INF | _MM_FROUND_NO_EXC for quiet floor

// Convenience macros (equal to round with appropriate imm8)
__m256  _mm256_floor_ps(__m256 a);   // = _mm256_round_ps(a, _MM_FROUND_TO_NEG_INF | _MM_FROUND_NO_EXC)
__m256  _mm256_ceil_ps(__m256 a);    // = _mm256_round_ps(a, _MM_FROUND_TO_POS_INF | _MM_FROUND_NO_EXC)
__m256d _mm256_floor_pd(__m256d a);
__m256d _mm256_ceil_pd(__m256d a);
```

### ADDSUB — Alternate Add/Subtract

```c
// Alternate between subtraction (even) and addition (odd) in each pair
__m256  _mm256_addsub_ps(__m256  a, __m256  b);
// result = [a0-b0, a1+b1, a2-b2, a3+b3, a4-b4, a5+b5, a6-b6, a7+b7]
__m256d _mm256_addsub_pd(__m256d a, __m256d b);
// Useful for: complex arithmetic
```

### Dot Product (SSE4.1 DPPS, 128-bit only)

```c
__m128 _mm_dp_ps(__m128 a, __m128 b, const int imm8);
// imm8[7:4] = participation mask (which elements multiply)
// imm8[3:0] = destination mask (which elements receive the sum)
// Example: full 4-element dot product into all lanes: imm8 = 0xFF
float dp = _mm_cvtss_f32(_mm_dp_ps(a, b, 0xFF));
```

---

## 18. SIMD Control: MXCSR Register

```c
// Get/set the full MXCSR register
unsigned int _mm_getcsr(void);
void         _mm_setcsr(unsigned int a);

// Rounding mode macros
#define _MM_ROUND_NEAREST     0x0000
#define _MM_ROUND_DOWN        0x2000
#define _MM_ROUND_UP          0x4000
#define _MM_ROUND_TOWARD_ZERO 0x6000
#define _MM_ROUNDING_MASK     0x6000

// Get/set rounding mode
#define _MM_GET_ROUNDING_MODE()    (_mm_getcsr() & _MM_ROUNDING_MASK)
#define _MM_SET_ROUNDING_MODE(x)   _mm_setcsr((_mm_getcsr() & ~_MM_ROUNDING_MASK) | (x))

// Flush-To-Zero (FTZ) — underflowing results become 0.0 instead of denormals
#define _MM_FLUSH_ZERO_MASK      0x8000
#define _MM_FLUSH_ZERO_OFF       0x0000
#define _MM_FLUSH_ZERO_ON        0x8000
#define _MM_GET_FLUSH_ZERO_MODE()   (_mm_getcsr() & _MM_FLUSH_ZERO_MASK)
#define _MM_SET_FLUSH_ZERO_MODE(x)  _mm_setcsr((_mm_getcsr() & ~_MM_FLUSH_ZERO_MASK) | (x))

// Denormals-Are-Zero (DAZ) — denormal inputs treated as 0.0
#define _MM_DENORMALS_ZERO_MASK  0x0040
#define _MM_DENORMALS_ZERO_OFF   0x0000
#define _MM_DENORMALS_ZERO_ON    0x0040
#define _MM_GET_DENORMALS_ZERO_MODE()   (_mm_getcsr() & _MM_DENORMALS_ZERO_MASK)
#define _MM_SET_DENORMALS_ZERO_MODE(x)  _mm_setcsr((_mm_getcsr() & ~_MM_DENORMALS_ZERO_MASK) | (x))

// Exception flags
#define _MM_EXCEPT_INVALID    0x0001
#define _MM_EXCEPT_DENORM     0x0002
#define _MM_EXCEPT_DIV_ZERO   0x0004
#define _MM_EXCEPT_OVERFLOW   0x0008
#define _MM_EXCEPT_UNDERFLOW  0x0010
#define _MM_EXCEPT_INEXACT    0x0020
#define _MM_EXCEPTION_MASK    0x003F

// Exception mask (1 = suppress exception, 0 = generate)
#define _MM_MASK_INVALID    0x0080
#define _MM_MASK_DENORM     0x0100
#define _MM_MASK_DIV_ZERO   0x0200
#define _MM_MASK_OVERFLOW   0x0400
#define _MM_MASK_UNDERFLOW  0x0800
#define _MM_MASK_INEXACT    0x1000
#define _MM_MASK_MASK       0x1F80

// High performance mode: enable FTZ+DAZ (avoids slow denormal processing)
void enable_fast_float_mode(void) {
    _MM_SET_FLUSH_ZERO_MODE(_MM_FLUSH_ZERO_ON);
    _MM_SET_DENORMALS_ZERO_MODE(_MM_DENORMALS_ZERO_ON);
    // This can give 100x speedup when encountering many subnormals!
}
```

---

## 19. Advanced Patterns & Idioms

### Pattern 1: Conditional Select (branchless)

```c
// Branchless select: result[i] = cond[i] ? a[i] : b[i]
__m256 select_ps(__m256 cond_mask, __m256 a, __m256 b) {
    return _mm256_blendv_ps(b, a, cond_mask);
}

// Example: clamp values to [lo, hi]
__m256 clamp_ps(__m256 v, float lo, float hi) {
    return _mm256_min_ps(_mm256_max_ps(v, _mm256_set1_ps(lo)), _mm256_set1_ps(hi));
}

// ReLU: max(0, x)
__m256 relu_ps(__m256 x) {
    return _mm256_max_ps(x, _mm256_setzero_ps());
}
```

### Pattern 2: Loop with Tail Handling

```c
void saxpy(float* y, const float* x, float a, int n) {
    __m256 va = _mm256_set1_ps(a);
    int i = 0;
    // Main loop: process 8 elements at a time
    for (; i <= n - 8; i += 8) {
        __m256 vx = _mm256_loadu_ps(&x[i]);
        __m256 vy = _mm256_loadu_ps(&y[i]);
        vy = _mm256_fmadd_ps(va, vx, vy);  // y += a*x
        _mm256_storeu_ps(&y[i], vy);
    }
    // Tail: process remaining elements
    for (; i < n; i++) y[i] += a * x[i];
}
```

### Pattern 3: Scalar Extraction

```c
// Extract a specific lane from a __m256 register
float extract_ps(__m256 v, int lane) {
    alignas(32) float tmp[8];
    _mm256_store_ps(tmp, v);
    return tmp[lane];
}

// Or more efficiently for specific lanes:
float extract_lane0(__m256 v) { return _mm_cvtss_f32(_mm256_castps256_ps128(v)); }
float extract_lane4(__m256 v) { return _mm_cvtss_f32(_mm256_extractf128_ps(v, 1)); }
```

### Pattern 4: SIMD Population Count (without AVX-512)

```c
// Count set bits in each byte using VPSHUFB lookup table
__m256i popcount_epi8(__m256i v) {
    __m256i lookup = _mm256_set_epi8(
        4,3,3,2,3,2,2,1, 3,2,2,1,2,1,1,0,
        4,3,3,2,3,2,2,1, 3,2,2,1,2,1,1,0
    );
    __m256i lo  = _mm256_and_si256(v, _mm256_set1_epi8(0x0F));
    __m256i hi  = _mm256_and_si256(_mm256_srli_epi16(v, 4), _mm256_set1_epi8(0x0F));
    __m256i cnt = _mm256_add_epi8(_mm256_shuffle_epi8(lookup, lo),
                                   _mm256_shuffle_epi8(lookup, hi));
    return cnt;  // popcount of each byte
}

// Sum all byte popcounts into 64-bit results (4 per 256-bit register)
__m256i popcount_epi64(__m256i v) {
    return _mm256_sad_epu8(popcount_epi8(v), _mm256_setzero_si256());
}
```

### Pattern 5: Prefix Sum (Scan)

```c
// Parallel prefix sum across 8 int32 elements
__m256i prefix_sum_epi32(__m256i v) {
    // Step 1: add each element to its left neighbor
    v = _mm256_add_epi32(v, _mm256_slli_si256(v, 4));    // +1 shift
    // Step 2: add pairs
    v = _mm256_add_epi32(v, _mm256_slli_si256(v, 8));    // +2 shift
    // Step 3: propagate across 128-bit lane boundary
    __m256i v_lo = _mm256_permute2x128_si256(v, v, 0x08); // broadcast low lane value
    __m256i carry = _mm256_shuffle_epi32(v_lo, 0xFF);      // broadcast element 3
    carry = _mm256_blend_epi32(_mm256_setzero_si256(), carry, 0xF0);
    v = _mm256_add_epi32(v, carry);
    return v;
}
```

### Pattern 6: 8×8 Byte Matrix Transpose

```c
// Transpose an 8×8 block of uint8 values stored row-major
void transpose8x8_epi8(uint8_t* src, uint8_t* dst, int src_stride, int dst_stride) {
    __m128i rows[8];
    // Load 8 rows
    for (int i = 0; i < 8; i++)
        rows[i] = _mm_loadl_epi64((__m128i*)&src[i * src_stride]);

    // Interleave bytes
    __m128i t[8];
    t[0] = _mm_unpacklo_epi8(rows[0], rows[1]);
    t[1] = _mm_unpacklo_epi8(rows[2], rows[3]);
    t[2] = _mm_unpacklo_epi8(rows[4], rows[5]);
    t[3] = _mm_unpacklo_epi8(rows[6], rows[7]);
    __m128i u[4];
    u[0] = _mm_unpacklo_epi16(t[0], t[1]);
    u[1] = _mm_unpackhi_epi16(t[0], t[1]);
    u[2] = _mm_unpacklo_epi16(t[2], t[3]);
    u[3] = _mm_unpackhi_epi16(t[2], t[3]);
    __m128i v[8];
    v[0] = _mm_unpacklo_epi32(u[0], u[2]);
    v[1] = _mm_unpackhi_epi32(u[0], u[2]);
    v[2] = _mm_unpacklo_epi32(u[1], u[3]);
    v[3] = _mm_unpackhi_epi32(u[1], u[3]);

    for (int i = 0; i < 4; i++) {
        _mm_storel_epi64((__m128i*)&dst[(2*i)   * dst_stride], v[i]);
        _mm_storel_epi64((__m128i*)&dst[(2*i+1) * dst_stride], _mm_srli_si128(v[i], 8));
    }
}
```

### Pattern 7: Fast Memset with Non-Temporal Stores

```c
void fast_memset_nt(void* dst, int val_byte, size_t bytes) {
    uint8_t byte = (uint8_t)val_byte;
    __m256i vec = _mm256_set1_epi8((char)byte);
    // Must be 32-byte aligned for stream stores
    uintptr_t ptr = (uintptr_t)dst;
    uintptr_t aligned = (ptr + 31) & ~31UL;
    // Scalar prefix
    memset(dst, byte, aligned - ptr);
    // Vectorized non-temporal loop
    uint8_t* p = (uint8_t*)aligned;
    uint8_t* end = (uint8_t*)((uintptr_t)dst + bytes);
    while (p + 32 <= end) {
        _mm256_stream_si256((__m256i*)p, vec);
        p += 32;
    }
    _mm_sfence();  // Ensure all stores complete before continuing
    // Scalar suffix
    while (p < end) *p++ = byte;
}
```

---

## 20. Performance Optimization Guide

### Throughput vs. Latency

For pipelined code: **throughput** (inverse of 1/throughput = instructions per cycle) matters more than latency. Modern CPUs can have 2–4 FP execution units.

**Key operations on Intel Skylake (per core, 256-bit):**

| Operation | Latency | Throughput |
|-----------|---------|------------|
| `vaddps ymm`    | 4 cycles  | 2/cycle |
| `vmulps ymm`    | 4 cycles  | 2/cycle |
| `vfmadd231ps`   | 4 cycles  | 2/cycle |
| `vdivps ymm`    | 11-14 cycles | 0.2/cycle |
| `vsqrtps ymm`   | 12-15 cycles | 0.15/cycle |
| `vrcpps ymm`    | 4 cycles  | 1/cycle |
| `vrsqrtps ymm`  | 4 cycles  | 1/cycle |
| `vmovaps ymm`   | — (rename) | 2/cycle |
| `vpshufb ymm`   | 1 cycle   | 1/cycle |
| `vpermps ymm`   | 3 cycles  | 1/cycle |
| `vgatherdps`    | ~20 cycles | — |

### Loop Unrolling

```c
// Without unroll: one outstanding FMA per cycle (limited by latency chain)
for (int i = 0; i < n; i += 8)
    acc0 = _mm256_fmadd_ps(a[i], b[i], acc0);  // acc0 depends on previous acc0 (4-cycle chain)

// With unroll×4: 4 independent accumulators → hides 4-cycle latency
__m256 acc0 = _mm256_setzero_ps(), acc1 = _mm256_setzero_ps(),
       acc2 = _mm256_setzero_ps(), acc3 = _mm256_setzero_ps();
for (int i = 0; i < n; i += 32) {
    acc0 = _mm256_fmadd_ps(av[0], bv[0], acc0);
    acc1 = _mm256_fmadd_ps(av[1], bv[1], acc1);
    acc2 = _mm256_fmadd_ps(av[2], bv[2], acc2);
    acc3 = _mm256_fmadd_ps(av[3], bv[3], acc3);
}
acc0 = _mm256_add_ps(_mm256_add_ps(acc0, acc1), _mm256_add_ps(acc2, acc3));
```

### Memory Alignment

```c
// Aligned allocation
float* buf = (float*)aligned_alloc(32, n * sizeof(float));
// Or: posix_memalign((void**)&buf, 32, n * sizeof(float));
// MSVC: _aligned_malloc(n * sizeof(float), 32);

// Alignment annotation (helps compiler generate aligned loads)
float* __restrict__ a __attribute__((aligned(32)));
__builtin_assume_aligned(a, 32);  // GCC/Clang hint

// Detect alignment at runtime
bool is_aligned_32(const void* p) { return ((uintptr_t)p & 31) == 0; }
```

### Avoid False Dependencies

```c
// BAD: writing xmm0 leaves upper ymm0 in undefined state in old code
// (this "dirty upper" issue existed in Sandy Bridge era)
// GOOD: use VEX-encoded instructions exclusively (all intrinsics do this)

// BAD: mixing non-VEX SSE and VEX AVX causes state transitions and penalties
// Don't call legacy SSE intrinsics between AVX operations in hot paths

// After AVX code, call _mm256_zeroupper() before calling non-AVX functions
// to clear upper 128 bits of YMM registers (avoids false dependency stall)
_mm256_zeroupper();
// Or use _mm256_zeroall() to zero all YMM registers
```

### Data Layout: AoS vs SoA

```c
// Array of Structures (AoS) — bad for SIMD
struct Particle { float x, y, z, mass; };
Particle particles[N];
// SIMD access: particles[0].x, particles[1].x, etc. are 16 bytes apart → gather required

// Structure of Arrays (SoA) — good for SIMD
struct Particles {
    float x[N], y[N], z[N], mass[N];
};
// SIMD access: x[0..7] are contiguous → simple vector load
__m256 vx = _mm256_loadu_ps(&particles.x[i]);  // 8 x-coordinates in one instruction

// AoSoA (Array of Structures of Arrays) — balanced
// 8 particles packed together, then next 8...
struct Particle8 { float x[8], y[8], z[8], mass[8]; };
```

### Instruction Latency Hiding with Software Pipelining

```c
// Software pipelining example: load N iterations ahead
void sum_stream(float* a, float* b, float* out, int n) {
    // Prefetch ahead
    _mm_prefetch((char*)&a[16], _MM_HINT_T0);
    _mm_prefetch((char*)&b[16], _MM_HINT_T0);
    for (int i = 0; i < n; i += 8) {
        _mm_prefetch((char*)&a[i + 32], _MM_HINT_T0);
        _mm_prefetch((char*)&b[i + 32], _MM_HINT_T0);
        __m256 va = _mm256_loadu_ps(&a[i]);
        __m256 vb = _mm256_loadu_ps(&b[i]);
        _mm256_storeu_ps(&out[i], _mm256_add_ps(va, vb));
    }
}
```

---

## 21. Compiler Auto-Vectorization

### Helping the Compiler

```c
// Restrict pointers (no aliasing) — lets compiler vectorize freely
void add_arrays(float* __restrict__ a, float* __restrict__ b,
                float* __restrict__ c, int n) {
    for (int i = 0; i < n; i++) c[i] = a[i] + b[i];
}

// Aligned hint
void add_aligned(float* a, float* b, float* c, int n) {
    a = (float*)__builtin_assume_aligned(a, 32);
    b = (float*)__builtin_assume_aligned(b, 32);
    c = (float*)__builtin_assume_aligned(c, 32);
    for (int i = 0; i < n; i++) c[i] = a[i] + b[i];
}

// Loop vectorization pragmas
#pragma GCC ivdep          // GCC: no data dependencies (ignore alias analysis)
#pragma clang loop vectorize(enable)  // Clang: force vectorization
#pragma clang loop unroll(4)          // Clang: unroll 4x
for (int i = 0; i < n; i++) { ... }
```

### Vectorization Report

```bash
# GCC: see vectorization decisions
gcc -O2 -mavx2 -fopt-info-vec-optimized myfile.c   # Show what was vectorized
gcc -O2 -mavx2 -fopt-info-vec-missed myfile.c      # Show what was NOT vectorized

# Clang: emit vectorization report
clang -O2 -mavx2 -Rpass=loop-vectorize myfile.c
clang -O2 -mavx2 -Rpass-missed=loop-vectorize myfile.c

# MSVC
cl /O2 /arch:AVX2 /Qvec-report:2 myfile.c
```

### Common Vectorization Killers

```c
// 1. Non-unit stride access
for (int i = 0; i < n; i++) out[i] = a[i*2];  // stride-2 — hard to vectorize

// 2. Function calls inside loop (unless inlined)
for (int i = 0; i < n; i++) out[i] = sin(a[i]);  // scalar libm — blocks vectorization

// 3. Data-dependent exit conditions
for (int i = 0; i < n; i++) { if (a[i] < 0) break; }  // unknown loop count

// 4. Aliased pointers
void f(float* a, float* b) {
    for (int i = 0; i < n; i++) a[i] = a[i-1] + b[i];  // carried dependency
}

// 5. Complex conditional branches within the loop body
for (int i = 0; i < n; i++) {
    if (a[i] > threshold)
        out[i] = sqrt(a[i]);  // non-uniform: some lanes compute, others don't
    else
        out[i] = 0;
    // Fix: use SIMD blend/mask instead
}
```

---

## 22. Debugging & Profiling SIMD Code

### Debug Helper: Print Vector Contents

```c
#include <stdio.h>

void print_m256_ps(const char* name, __m256 v) {
    alignas(32) float f[8];
    _mm256_store_ps(f, v);
    printf("%s: [%f, %f, %f, %f, %f, %f, %f, %f]\n",
           name, f[0], f[1], f[2], f[3], f[4], f[5], f[6], f[7]);
}

void print_m256i_epi32(const char* name, __m256i v) {
    alignas(32) int32_t i[8];
    _mm256_store_si256((__m256i*)i, v);
    printf("%s: [%d, %d, %d, %d, %d, %d, %d, %d]\n",
           name, i[0], i[1], i[2], i[3], i[4], i[5], i[6], i[7]);
}

void print_m256i_hex(const char* name, __m256i v) {
    alignas(32) uint32_t u[8];
    _mm256_store_si256((__m256i*)u, v);
    printf("%s: [%08X, %08X, %08X, %08X, %08X, %08X, %08X, %08X]\n",
           name, u[0], u[1], u[2], u[3], u[4], u[5], u[6], u[7]);
}
```

### Profiling Tools

| Tool | Purpose |
|------|---------|
| **Intel VTune Profiler** | SIMD utilization, hotspot analysis, memory access patterns |
| **IACA (Intel Architecture Code Analyzer)** | Static throughput/latency analysis of code blocks |
| **llvm-mca** | LLVM Machine Code Analyzer — latency and throughput estimation |
| **Perf (Linux)** | Hardware counter-based profiling (FP ops, cache misses, etc.) |
| **Valgrind/Callgrind** | Instruction counting (slower, no hardware counters) |
| **Intel SDE** | Software Development Emulator — run AVX-512 on older hardware |
| **LIKWID** | Performance monitoring on Linux for SIMD and memory bandwidth |

### Key Performance Counters

```bash
# Linux perf: count FP SIMD instructions
perf stat -e fp_arith_inst_retired.256b_packed_single \
           -e fp_arith_inst_retired.256b_packed_double \
           -e fp_arith_inst_retired.scalar_single \
           ./my_program

# Count cache misses alongside SIMD ops
perf stat -e cache-misses,cache-references,fp_arith_inst_retired.256b_packed_single ./prog
```

### Common Bugs and Checks

```c
// 1. Alignment fault: store to unaligned address with aligned instruction
// Always use loadu/storeu unless you KNOW the address is 32-byte aligned

// 2. Lane ordering confusion
// _mm256_set_ps(e7,e6,e5,e4,e3,e2,e1,e0) — HIGH to LOW order!
// _mm256_setr_ps(e0,e1,e2,e3,e4,e5,e6,e7) — LOW to HIGH (matches array order)

// 3. Mixing VEX and non-VEX causes AVX-SSE transition penalty
// Ensure all SSE intrinsics are VEX-encoded (compiler does this with -mavx2)

// 4. Forgetting _mm256_zeroupper() when calling non-AVX code
_mm256_zeroupper();  // Add before calls to external functions, and at function exit

// 5. Integer vs float type confusion
// _mm256_and_ps operates on the same bits as _mm256_and_si256 but wrong type tag
// Use _mm256_castsi256_ps / _mm256_castps_si256 to switch without copying

// 6. pshufb/packs operating per-128-bit-lane (not across full 256 bits)
// Always check whether your shuffle/pack instruction is lane-limited

// 7. Denormal slowdown — enable FTZ+DAZ in performance-critical code
_MM_SET_FLUSH_ZERO_MODE(_MM_FLUSH_ZERO_ON);
_MM_SET_DENORMALS_ZERO_MODE(_MM_DENORMALS_ZERO_ON);
```

---

## 23. Complete Worked Examples

### Example 1: SGEMM Inner Kernel (Matrix Multiply)

```c
// Single-precision GEMM kernel: C += A * B
// A is M×K, B is K×N, C is M×N
// This implements a small 8×1 × 1×8 rank-1 update kernel
void sgemm_kernel_8x8(const float* A, const float* B, float* C,
                       int M, int N, int K,
                       int lda, int ldb, int ldc) {
    // Load 8 columns of C as 8 float32 vectors
    __m256 c0 = _mm256_loadu_ps(&C[0*ldc]);
    __m256 c1 = _mm256_loadu_ps(&C[1*ldc]);
    // ... (load all 8 rows)

    for (int k = 0; k < K; k++) {
        // Broadcast A[i][k] to all 8 lanes
        __m256 a0 = _mm256_broadcast_ss(&A[0*lda + k]);
        // Load B[k][0..7]
        __m256 b  = _mm256_loadu_ps(&B[k*ldb]);
        // FMA: C[i][0..7] += A[i][k] * B[k][0..7]
        c0 = _mm256_fmadd_ps(a0, b, c0);
        // repeat for all 8 rows...
    }
    _mm256_storeu_ps(&C[0*ldc], c0);
    // ... store all 8 rows
}
```

### Example 2: AVX2 FMA Sigmoid Activation

```c
// Fast sigmoid using FMA and polynomial approximation
// Approximation: sigmoid(x) ≈ 0.5 + 0.25*x - 0.0625*x^3 (for |x| < 2)
// More accurate: use rational approximation

// Accurate sigmoid via exp approximation
__m256 sigmoid_ps(__m256 x) {
    // Clamp to avoid overflow in exp
    x = _mm256_max_ps(x, _mm256_set1_ps(-88.0f));
    x = _mm256_min_ps(x, _mm256_set1_ps(88.0f));

    // Compute e^x using fast_exp (Cody-Waite range reduction + polynomial)
    // 1. Decompose: x = n*ln2 + r, |r| <= 0.5*ln2
    __m256 log2e = _mm256_set1_ps(1.44269504f);
    __m256 ln2   = _mm256_set1_ps(0.693147180f);
    __m256 half  = _mm256_set1_ps(0.5f);
    __m256 n_f   = _mm256_floor_ps(_mm256_fmadd_ps(x, log2e, half));
    __m256 r     = _mm256_fnmadd_ps(n_f, ln2, x);  // r = x - n*ln2

    // 2. Polynomial: e^r ≈ 1 + r + r^2/2 + r^3/6 + ...
    __m256 poly = _mm256_set1_ps(1.0f/720.0f);   // 1/6!
    poly = _mm256_fmadd_ps(poly, r, _mm256_set1_ps(1.0f/120.0f));
    poly = _mm256_fmadd_ps(poly, r, _mm256_set1_ps(1.0f/24.0f));
    poly = _mm256_fmadd_ps(poly, r, _mm256_set1_ps(1.0f/6.0f));
    poly = _mm256_fmadd_ps(poly, r, _mm256_set1_ps(1.0f/2.0f));
    poly = _mm256_fmadd_ps(poly, r, _mm256_set1_ps(1.0f));
    poly = _mm256_fmadd_ps(poly, r, _mm256_set1_ps(1.0f));

    // 3. Scale by 2^n using integer exponent trick
    __m256i n_i = _mm256_cvtps_epi32(n_f);
    n_i = _mm256_add_epi32(n_i, _mm256_set1_epi32(127));
    n_i = _mm256_slli_epi32(n_i, 23);
    __m256 exp_x = _mm256_mul_ps(poly, _mm256_castsi256_ps(n_i));

    // sigmoid = 1 / (1 + e^(-x)) = e^x / (1 + e^x)
    __m256 one = _mm256_set1_ps(1.0f);
    return _mm256_div_ps(exp_x, _mm256_add_ps(one, exp_x));
}
```

### Example 3: INT8 Quantized Dot Product (Neural Network)

```c
// INT8 dot product: typical in quantized neural networks
// Computes dot(a, b) where a,b are int8 vectors of length n
// Result accumulated in int32

int32_t int8_dot_avx2(const int8_t* a, const int8_t* b, int n) {
    __m256i acc = _mm256_setzero_si256();
    int i = 0;

    for (; i <= n - 32; i += 32) {
        __m256i va = _mm256_loadu_si256((__m256i*)&a[i]);
        __m256i vb = _mm256_loadu_si256((__m256i*)&b[i]);

        // Widen to int16 and multiply (using PMADDUBSW trick)
        // PMADDUBSW: treat a as uint8, b as int8 → 16-bit saturated results
        // Re-sign: if a can be signed, adjust or handle saturation
        __m256i prod16 = _mm256_maddubs_epi16(
            _mm256_add_epi8(va, _mm256_set1_epi8(128)),  // shift a to uint8
            vb
        );
        // Widen to int32 and accumulate
        __m256i prod32 = _mm256_madd_epi16(prod16, _mm256_set1_epi16(1));
        acc = _mm256_add_epi32(acc, prod32);
    }

    // Reduce acc (8 int32 → scalar)
    __m128i lo = _mm256_castsi256_si128(acc);
    __m128i hi = _mm256_extracti128_si256(acc, 1);
    lo = _mm_add_epi32(lo, hi);
    lo = _mm_hadd_epi32(lo, lo);
    lo = _mm_hadd_epi32(lo, lo);
    int32_t result = _mm_cvtsi128_si32(lo);

    // Scalar tail
    for (; i < n; i++) result += (int32_t)a[i] * b[i];
    return result;
}
```

### Example 4: RGB Image Processing — Grayscale Conversion

```c
// Convert RGB (interleaved: R0,G0,B0,R1,G1,B1,...) to grayscale
// gray = 0.299*R + 0.587*G + 0.114*B
void rgb_to_gray_avx2(const uint8_t* rgb, uint8_t* gray, int n_pixels) {
    // Weights scaled to uint8 (sum ≈ 255): R=77, G=150, B=29  (77+150+29=256≈255)
    // Process 8 pixels at a time = 24 bytes of RGB input

    const __m256i wr = _mm256_set1_epi16(77);
    const __m256i wg = _mm256_set1_epi16(150);
    const __m256i wb = _mm256_set1_epi16(29);

    int i = 0;
    for (; i <= n_pixels - 8; i += 8) {
        // Load 24 bytes: 8 pixels × 3 channels
        __m128i raw = _mm_loadu_si128((__m128i*)&rgb[i*3]);
        // Deinterleave R, G, B channels...
        // (Use pshufb to extract each channel — specific indices depend on pixel layout)
        // Simplified (requires full deinterleave which uses ~6 shuffles):
        // Here we show the final MAC step assuming r8, g8, b8 are loaded:

        // Promote uint8 to uint16
        __m256i r16 = _mm256_cvtepu8_epi16(/*r8*/raw);  // simplified
        __m256i g16 = _mm256_cvtepu8_epi16(/*g8*/raw);
        __m256i b16 = _mm256_cvtepu8_epi16(/*b8*/raw);

        // Weighted sum
        __m256i gray16 = _mm256_add_epi16(
                            _mm256_add_epi16(
                                _mm256_mullo_epi16(r16, wr),
                                _mm256_mullo_epi16(g16, wg)),
                            _mm256_mullo_epi16(b16, wb));

        // Divide by 256 (right shift 8)
        gray16 = _mm256_srli_epi16(gray16, 8);

        // Pack back to uint8
        __m128i result = _mm256_castsi256_si128(
                            _mm256_packs_epi16(gray16,
                                _mm256_setzero_si256()));
        _mm_storel_epi64((__m128i*)&gray[i], result);
    }
    // Scalar tail...
}
```

---

## 24. Quick-Reference Tables

### AVX2 Intrinsic Summary by Category

| Category | Float32 | Float64 | Int8/16 | Int32 | Int64 |
|----------|---------|---------|---------|-------|-------|
| Load aligned | `_mm256_load_ps` | `_mm256_load_pd` | — | — | `_mm256_load_si256` |
| Load unaligned | `_mm256_loadu_ps` | `_mm256_loadu_pd` | `_mm256_loadu_si256` | ← | ← |
| Broadcast scalar | `_mm256_set1_ps` | `_mm256_set1_pd` | `_mm256_set1_epi8/16` | `_mm256_set1_epi32` | `_mm256_set1_epi64x` |
| Add | `_mm256_add_ps` | `_mm256_add_pd` | `_mm256_add_epi8/16` | `_mm256_add_epi32` | `_mm256_add_epi64` |
| Subtract | `_mm256_sub_ps` | `_mm256_sub_pd` | `_mm256_sub_epi8/16` | `_mm256_sub_epi32` | `_mm256_sub_epi64` |
| Multiply | `_mm256_mul_ps` | `_mm256_mul_pd` | — | `_mm256_mullo_epi32` | — |
| FMA | `_mm256_fmadd_ps` | `_mm256_fmadd_pd` | — | — | — |
| Min | `_mm256_min_ps` | `_mm256_min_pd` | `_mm256_min_epi8` | `_mm256_min_epi32` | — |
| Max | `_mm256_max_ps` | `_mm256_max_pd` | `_mm256_max_epi8` | `_mm256_max_epi32` | — |
| Compare | `_mm256_cmp_ps` | `_mm256_cmp_pd` | `_mm256_cmpeq_epi8` | `_mm256_cmpgt_epi32` | `_mm256_cmpeq_epi64` |
| Blend | `_mm256_blendv_ps` | `_mm256_blendv_pd` | `_mm256_blendv_epi8` | ← | ← |
| Shift left | — | — | — | `_mm256_slli_epi32` | `_mm256_slli_epi64` |
| Shift right | — | — | — | `_mm256_srli_epi32` | `_mm256_srli_epi64` |
| Store aligned | `_mm256_store_ps` | `_mm256_store_pd` | — | — | `_mm256_store_si256` |
| Store unaligned | `_mm256_storeu_ps` | `_mm256_storeu_pd` | `_mm256_storeu_si256` | ← | ← |

### Data Widths Per Register

| Type | Elements per __m128 | Elements per __m256 |
|------|--------------------|--------------------|
| int8 / uint8  | 16 | 32 |
| int16 / uint16 | 8 | 16 |
| int32 / uint32 | 4 | 8 |
| int64 / uint64 | 2 | 4 |
| float32 | 4 | 8 |
| float64 | 2 | 4 |

### FMA Variant Summary

| Intrinsic | Operation |
|-----------|-----------|
| `fmadd`  | `a*b + c` |
| `fmsub`  | `a*b - c` |
| `fnmadd` | `-(a*b) + c` |
| `fnmsub` | `-(a*b) - c` |
| `fmaddsub` | `a*b ± c` (even lanes `-`, odd lanes `+`) |
| `fmsubadd` | `a*b ∓ c` (even lanes `+`, odd lanes `-`) |

### Alignment Requirements

| Register size | Aligned load/store | Unaligned |
|---------------|-------------------|-----------|
| 128-bit XMM | 16-byte aligned | `loadu`/`storeu` |
| 256-bit YMM | 32-byte aligned | `loadu`/`storeu` |
| Non-temporal | Must be aligned | N/A |

### Common imm8 Shuffles for `_mm256_permute_ps`

| imm8 | Effect |
|------|--------|
| `0x00` | Broadcast lane 0 to all: [0,0,0,0,...] |
| `0x55` | Broadcast lane 1 to all: [1,1,1,1,...] |
| `0xAA` | Broadcast lane 2 |
| `0xFF` | Broadcast lane 3 |
| `0x1B` | Reverse within each 128-bit lane: [3,2,1,0,...] |
| `0xB1` | Swap pairs: [2,3,0,1,...] |
| `0x4E` | Rotate by 2: [1,0,3,2,...] |

### Useful Compiler Explorer Resources

- **Godbolt** (compiler-explorer.com): See SIMD assembly output, compare compilers
- **Intel Intrinsics Guide**: software.intel.com/sites/landingpage/IntrinsicsGuide/
- **Agner Fog's Tables**: agner.org/optimize — instruction latency/throughput tables for all Intel/AMD microarchitectures
- **uops.info**: Detailed per-instruction port binding data

---

## Appendix A: AVX2 vs. AVX Differences

| Feature | AVX (2011) | AVX2 (2013) |
|---------|-----------|------------|
| Float32/Float64 | 256-bit ✓ | 256-bit ✓ |
| Integer ops (256-bit) | Only 128-bit in YMM | Full 256-bit ✓ |
| Gather | ✗ | ✓ |
| Variable shift | ✗ | ✓ (`sllv`, `srlv`, `srav`) |
| Broadcast from memory | Only `_ps`/`_pd` | All integer sizes too ✓ |
| `vperm2i128` | ✗ (only f128) | ✓ |
| `permutevar8x32` | ✗ | ✓ |

## Appendix B: FMA3 vs FMA4

| | FMA3 | FMA4 |
|---|------|------|
| Operands | 3 (destructive source) | 4 (non-destructive) |
| Intel support | Haswell+ | Never on Intel |
| AMD support | Piledriver+ | Bulldozer only |
| Recommendation | **Use FMA3** — FMA4 is dead | Avoid |

## Appendix C: AVX-512 Preview (Successor to AVX2)

AVX-512 (Skylake-X, Ice Lake, Zen 4) extends further:
- 512-bit ZMM registers (16 elements float32, 8 double)
- 8 mask registers k0–k7 for per-lane predicated execution
- Scatter (complement to gather)
- Embedded rounding and broadcast in instruction encoding
- New operations: ternary logic, conflict detection, bit manipulation

```c
// AVX-512 masked add example (requires -mavx512f):
__m512 _mm512_mask_add_ps(__m512 src, __mmask16 k, __m512 a, __m512 b);
// only updates lanes where k bit is set; other lanes get src
```

---

*Reference compiled from Intel Intrinsics Guide, Intel Software Developer's Manual Vol. 1-3, Agner Fog's Optimization Manuals, and real-world SIMD engineering practice. All intrinsics are for GCC/Clang/MSVC with `<immintrin.h>`. Performance data from Intel Architecture Instruction Latency tables and Agner Fog's microarchitecture tables.*