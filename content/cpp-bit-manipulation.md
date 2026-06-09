---
title: "Complete Bit Manipulation in C++ Reference"
description: "Notes bit manipulation."
---

## The Definitive Guide — Beginner to Advanced

---

# PART 1: FOUNDATIONS

---

## 1.1 Number Systems and Binary Representation

### How Integers Are Stored

Every integer in a computer is stored as a sequence of bits (binary digits), each being 0 or 1. A byte is 8 bits. A 32-bit integer occupies 4 bytes, a 64-bit integer occupies 8 bytes.

```
Decimal 42  =  Binary 00101010
               Bit positions (0-indexed from right):
               Bit 5 = 1  (2^5 = 32)
               Bit 3 = 1  (2^3 = 8)
               Bit 1 = 1  (2^1 = 2)
               32 + 8 + 2 = 42
```

### Unsigned Integers

An n-bit unsigned integer can represent values from 0 to 2^n - 1.

```
uint8_t:  0 to 255
uint16_t: 0 to 65535
uint32_t: 0 to 4294967295
uint64_t: 0 to 18446744073709551615
```

### Signed Integers — Two's Complement

C++ uses two's complement for signed integers. The most significant bit (MSB) is the sign bit.

```
For int8_t (8-bit signed):
 0 1 1 1 1 1 1 1 = +127
 0 0 0 0 0 0 0 1 = +1
 0 0 0 0 0 0 0 0 =  0
 1 1 1 1 1 1 1 1 = -1
 1 0 0 0 0 0 0 1 = -127
 1 0 0 0 0 0 0 0 = -128
```

To negate a number in two's complement: flip all bits, then add 1.

```cpp
int x = 42;    // 00101010
// Negate:
// Flip:       11010101
// Add 1:      11010110  = -42
// Verify: -42 in two's complement is indeed 11010110
```

Why two's complement? Because addition works uniformly for both positive and negative numbers without special hardware.

### Integer Literal Notations in C++

```cpp
int a = 42;          // Decimal
int b = 0b00101010;  // Binary (C++14)
int c = 052;         // Octal
int d = 0x2A;        // Hexadecimal

// All four are equal to 42

// Long and unsigned suffixes
unsigned int  u = 42U;
long          l = 42L;
unsigned long ul = 42UL;
long long     ll = 42LL;
unsigned long long ull = 42ULL;
```

### Printing in Different Bases

```cpp
#include <iostream>
#include <bitset>
#include <iomanip>

int x = 42;
std::cout << std::dec << x << "\n";       // 42
std::cout << std::oct << x << "\n";       // 52
std::cout << std::hex << x << "\n";       // 2a
std::cout << std::hex << std::uppercase << x << "\n"; // 2A
std::cout << std::bitset<32>(x) << "\n";  // 00000000000000000000000000101010

// Custom binary printer for learning
void printBinary(int n, int bits = 32) {
    for (int i = bits - 1; i >= 0; i--) {
        std::cout << ((n >> i) & 1);
        if (i % 4 == 0 && i != 0) std::cout << ' '; // group by 4
    }
    std::cout << '\n';
}
```

---

## 1.2 The Six Bitwise Operators

### AND (&)

Output is 1 only when BOTH inputs are 1.

```
Truth table:
0 & 0 = 0
0 & 1 = 0
1 & 0 = 0
1 & 1 = 1

Example:
  1 0 1 1 0 1 1 0   (182)
& 0 1 1 0 1 1 0 1   (109)
= 0 0 1 0 0 1 0 0   (36)
```

Use cases: masking, checking bits, clearing bits.

### OR (|)

Output is 1 when AT LEAST ONE input is 1.

```
Truth table:
0 | 0 = 0
0 | 1 = 1
1 | 0 = 1
1 | 1 = 1

Example:
  1 0 1 1 0 1 1 0   (182)
| 0 1 1 0 1 1 0 1   (109)
= 1 1 1 1 1 1 1 1   (255)
```

Use cases: setting bits, combining flags.

### XOR (^)

Output is 1 when inputs DIFFER.

```
Truth table:
0 ^ 0 = 0
0 ^ 1 = 1
1 ^ 0 = 1
1 ^ 1 = 0

Example:
  1 0 1 1 0 1 1 0   (182)
^ 0 1 1 0 1 1 0 1   (109)
= 1 1 0 1 1 0 1 1   (219)
```

Key properties of XOR:
- x ^ x = 0       (self-cancellation)
- x ^ 0 = x       (identity)
- x ^ ~0 = ~x     (flip all bits)
- commutative: a ^ b = b ^ a
- associative: (a ^ b) ^ c = a ^ (b ^ c)

Use cases: toggling bits, finding unique elements, in-place swap, cryptography.

### NOT (~)

Flips all bits (bitwise complement).

```
~0 0 1 0 1 0 1 0   (42)
= 1 1 0 1 0 1 0 1  (-43 in two's complement)

General rule: ~x = -(x + 1)
~0 = -1
~1 = -2
~42 = -43
```

Be careful: ~ on unsigned types wraps around, giving large positive values.

### Left Shift (<<)

Shifts bits toward the MSB (left). Zeros fill from the right.

```
42 = 0 0 1 0 1 0 1 0
42 << 1 = 0 1 0 1 0 1 0 0 = 84
42 << 2 = 1 0 1 0 1 0 0 0 = 168

Left shift by n = multiply by 2^n (when no overflow)
x << n  is equivalent to  x * (1 << n)
```

Undefined behavior when:
- Shifting by a negative amount
- Shifting by >= width of the type
- Left-shifting a negative value (in C++17 and earlier)
- Result overflows the type

### Right Shift (>>)

Shifts bits toward the LSB (right).

For unsigned types: logical right shift — zeros fill from the left.
For signed types: arithmetic right shift (implementation-defined, but virtually always arithmetic on modern systems) — sign bit fills from the left.

```
// Unsigned (logical shift):
uint8_t u = 0b10101000;  // 168
u >> 1 =    0b01010100   // 84  (zero fills from left)

// Signed (arithmetic shift):
int8_t s = -40;          // 11011000
s >> 1   = -20;          // 11101100  (sign bit fills from left)
```

Right shift by n = divide by 2^n (for positive, rounds toward negative infinity for negative).

---

## 1.3 Operator Precedence Pitfalls

Bitwise operators have lower precedence than comparison and arithmetic operators. This is the single most common source of bugs in bit manipulation code.

```cpp
// WRONG — evaluates as: x & (y == 1)
if (x & y == 1) { ... }

// CORRECT
if ((x & y) == 1) { ... }

// WRONG — evaluates as: (a + b) & (c + d) is correct here by accident,
// but this is wrong:
int result = a & b + c;    // parsed as a & (b + c)

// WRONG
if (x | y && z) { ... }    // parsed as x | (y && z)

// CORRECT
if ((x | y) && z) { ... }
```

Precedence order (highest to lowest, relevant operators):
```
~          (unary NOT — very high)
<< >>      (shifts)
&          (AND)
^          (XOR)
|          (OR)
&&         (logical AND)
||         (logical OR)
```

Always use parentheses liberally in bit manipulation code.

---

# PART 2: CORE BIT TRICKS

---

## 2.1 Reading, Setting, Clearing, and Toggling Bits

These are the fundamental operations every bit manipulation programmer must know cold.

```cpp
// Check if bit i is set (returns nonzero if set, 0 if not)
bool isBitSet(int n, int i) {
    return (n >> i) & 1;
    // Alternatively: n & (1 << i)
}

// Set bit i (force to 1)
int setBit(int n, int i) {
    return n | (1 << i);
}

// Clear bit i (force to 0)
int clearBit(int n, int i) {
    return n & ~(1 << i);
}

// Toggle bit i (flip 0 to 1 or 1 to 0)
int toggleBit(int n, int i) {
    return n ^ (1 << i);
}

// Change bit i to value v (0 or 1)
int changeBit(int n, int i, int v) {
    return (n & ~(1 << i)) | (v << i);
    // Clear the bit, then OR in the new value
}

// Get the value of bit i (0 or 1)
int getBit(int n, int i) {
    return (n >> i) & 1;
}
```

### 64-bit versions

When working with 64-bit integers, use `1LL` or `1ULL` instead of `1`, because `1` is a 32-bit int and `1 << 32` is undefined behavior.

```cpp
bool isBitSet64(long long n, int i) {
    return (n >> i) & 1LL;
}

long long setBit64(long long n, int i) {
    return n | (1LL << i);
}

long long clearBit64(long long n, int i) {
    return n & ~(1LL << i);
}

long long toggleBit64(long long n, int i) {
    return n ^ (1LL << i);
}
```

---

## 2.2 The Most Important Single-Expression Tricks

### Turn off the lowest set bit

```cpp
n & (n - 1)

// Example: n = 10110100
// n - 1   = 10110011
// n & n-1 = 10110000  (lowest set bit cleared)

// Applications:
// 1. Check if n is a power of 2: n > 0 && (n & (n-1)) == 0
// 2. Count set bits (Brian Kernighan's algorithm)
// 3. Iterate over all set bits
```

### Isolate the lowest set bit

```cpp
n & (-n)
// Equivalent to n & (~n + 1)

// Example: n = 10110100
// -n      = 01001100  (two's complement negation)
// n & -n  = 00000100  (only the lowest set bit remains)

// Also written as: n & (~n + 1)
// Application: find the rightmost set bit position
// Position = __builtin_ctz(n)  (count trailing zeros)
```

### Turn on the lowest unset bit

```cpp
n | (n + 1)
// Example: n = 10110011
// n + 1    = 10110100
// n | n+1  = 10110111  (lowest 0 bit turned to 1)
```

### Isolate the lowest unset bit

```cpp
~n & (n + 1)
// Example: n = 10110011
// ~n       = 01001100
// n + 1    = 10110100
// result   = 00000100  (lowest 0 bit isolated)
```

### Turn off the trailing 1s (lowest block of 1s)

```cpp
n & (n + 1)
// Example: n = 10110111
// n + 1    = 10111000
// n & n+1  = 10110000  (all trailing 1s cleared)
```

### Turn on the trailing 0s (lowest block of 0s)

```cpp
n | (n - 1)
// Example: n = 10110100
// n - 1    = 10110011
// n | n-1  = 10110111  (all trailing 0s set)
```

---

## 2.3 Checking Powers of Two and Related

```cpp
// Is n a power of 2?
bool isPow2(int n) {
    return n > 0 && (n & (n - 1)) == 0;
}

// Is n a power of 2 or zero?
bool isPow2OrZero(int n) {
    return (n & (n - 1)) == 0;
}

// Next power of 2 >= n
uint32_t nextPow2(uint32_t n) {
    if (n == 0) return 1;
    n--;
    n |= n >> 1;
    n |= n >> 2;
    n |= n >> 4;
    n |= n >> 8;
    n |= n >> 16;
    return n + 1;
}
// Explanation: spread the highest set bit across all lower bits,
// then add 1. Works for 32-bit integers.

// Previous power of 2 <= n
uint32_t prevPow2(uint32_t n) {
    n |= n >> 1;
    n |= n >> 2;
    n |= n >> 4;
    n |= n >> 8;
    n |= n >> 16;
    return n - (n >> 1);
}

// Log base 2 (floor) — position of highest set bit
int log2floor(uint32_t n) {
    return 31 - __builtin_clz(n);
    // __builtin_clz = count leading zeros
}
```

---

## 2.4 Swap Without Temporary

```cpp
// XOR swap — works correctly only when a and b are different variables
// (not different locations with the same address)
void xorSwap(int &a, int &b) {
    a ^= b;
    b ^= a;
    a ^= b;
}

// Why it works:
// After a ^= b: a = A^B, b = B
// After b ^= a: a = A^B, b = B^(A^B) = A
// After a ^= b: a = (A^B)^A = B, b = A
// Done: a = B, b = A

// CAUTION: if a and b are the same variable (xorSwap(x, x)),
// the result is 0. Always check before using in practice.

// Practical: just use std::swap — the compiler optimizes it perfectly.
```

---

## 2.5 Absolute Value Without Branching

```cpp
int absNoBranch(int n) {
    int mask = n >> 31;  // arithmetic right shift: -1 if negative, 0 if positive
    return (n ^ mask) - mask;
    // If n >= 0: mask = 0, (n ^ 0) - 0 = n
    // If n < 0:  mask = -1 = 0xFFFFFFFF, (n ^ 0xFFFFFFFF) - (-1) = ~n + 1 = -n
}
```

---

## 2.6 Minimum and Maximum Without Branching

```cpp
int minNoBranch(int x, int y) {
    return y ^ ((x ^ y) & -(x < y));
    // x < y evaluates to 1 or 0
    // -(1) = -1 = 0xFFFFFFFF, -(0) = 0
    // When x < y: y ^ (x ^ y) = x
    // When x >= y: y ^ 0 = y
}

int maxNoBranch(int x, int y) {
    return x ^ ((x ^ y) & -(x < y));
}
```

---

## 2.7 Arithmetic Tricks

```cpp
// Multiply by 2^k
int x = n << k;

// Divide by 2^k (for non-negative n)
int x = n >> k;

// Modulo by power of 2 (unsigned or positive n)
int x = n & ((1 << k) - 1);
// E.g., n % 8 == n & 7, n % 16 == n & 15

// Round up to multiple of power of 2
// Round up n to next multiple of (1 << k)
int rounded = (n + (1 << k) - 1) & ~((1 << k) - 1);

// Average of two integers without overflow
int avg = (x & y) + ((x ^ y) >> 1);
// (x & y) gets the bits both have, (x ^ y) >> 1 distributes the difference

// Multiply by 3
int times3 = (n << 1) + n;

// Multiply by 5
int times5 = (n << 2) + n;

// Multiply by 7
int times7 = (n << 3) - n;

// Check if two integers have opposite signs
bool oppositeSigns(int x, int y) {
    return (x ^ y) < 0;
    // If sign bits differ, XOR produces a negative number
}
```

---

# PART 3: COUNTING AND FINDING BITS

---

## 3.1 Population Count (Count Set Bits)

### Method 1: Brian Kernighan's Algorithm

Repeatedly clears the lowest set bit and counts iterations. O(k) where k = number of set bits.

```cpp
int popcount(int n) {
    int count = 0;
    while (n) {
        n &= (n - 1);  // clear lowest set bit
        count++;
    }
    return count;
}
```

### Method 2: Lookup Table

Precompute answers for all 8-bit values, then process 8 bits at a time.

```cpp
int lookup[256];  // precomputed for 0..255

void buildLookup() {
    lookup[0] = 0;
    for (int i = 1; i < 256; i++) {
        lookup[i] = (i & 1) + lookup[i >> 1];
    }
}

int popcount_lookup(uint32_t n) {
    return lookup[n & 0xFF]
         + lookup[(n >> 8)  & 0xFF]
         + lookup[(n >> 16) & 0xFF]
         + lookup[(n >> 24) & 0xFF];
}
```

### Method 3: Parallel Bit Counting (SWAR — SIMD Within A Register)

Works on 32-bit integers. This is the classic Hacker's Delight approach.

```cpp
int popcount32(uint32_t n) {
    n = n - ((n >> 1) & 0x55555555);         // 2-bit sums
    n = (n & 0x33333333) + ((n >> 2) & 0x33333333); // 4-bit sums
    n = (n + (n >> 4)) & 0x0F0F0F0F;         // 8-bit sums
    return (n * 0x01010101) >> 24;            // sum all bytes
}

// 64-bit version
int popcount64(uint64_t n) {
    n = n - ((n >> 1) & 0x5555555555555555ULL);
    n = (n & 0x3333333333333333ULL) + ((n >> 2) & 0x3333333333333333ULL);
    n = (n + (n >> 4)) & 0x0F0F0F0F0F0F0F0FULL;
    return (n * 0x0101010101010101ULL) >> 56;
}
```

### Method 4: Built-in (Fastest in Practice)

```cpp
#include <bit>  // C++20

// C++20
int c1 = std::popcount(42u);  // must be unsigned

// GCC/Clang built-ins (also works in competitive programming)
int c2 = __builtin_popcount(n);    // 32-bit
int c3 = __builtin_popcountl(n);   // long
int c4 = __builtin_popcountll(n);  // long long

// MSVC
#include <intrin.h>
int c5 = __popcnt(n);       // 32-bit
int c6 = __popcnt64(n);     // 64-bit
```

---

## 3.2 Count Leading Zeros (CLZ) and Trailing Zeros (CTZ)

```cpp
// Count leading zeros (undefined for n == 0)
int clz = __builtin_clz(n);    // 32-bit unsigned
int clzl = __builtin_clzl(n);  // long
int clzll = __builtin_clzll(n);// long long

// Count trailing zeros (undefined for n == 0)
int ctz = __builtin_ctz(n);    // 32-bit unsigned
int ctzl = __builtin_ctzl(n);
int ctzll = __builtin_ctzll(n);

// C++20 equivalents (well-defined for 0)
#include <bit>
int l = std::countl_zero(42u);   // leading zeros
int t = std::countr_zero(42u);   // trailing zeros
int lo = std::countl_one(42u);   // leading ones
int to = std::countr_one(42u);   // trailing ones

// Position of highest set bit (= floor(log2(n)) for n > 0)
int highestBit = 31 - __builtin_clz(n);

// Position of lowest set bit (0-indexed)
int lowestBit = __builtin_ctz(n);

// Manual CLZ for portability
int clzManual(uint32_t n) {
    if (n == 0) return 32;
    int count = 0;
    if (!(n & 0xFFFF0000)) { count += 16; n <<= 16; }
    if (!(n & 0xFF000000)) { count += 8;  n <<= 8;  }
    if (!(n & 0xF0000000)) { count += 4;  n <<= 4;  }
    if (!(n & 0xC0000000)) { count += 2;  n <<= 2;  }
    if (!(n & 0x80000000)) { count += 1;             }
    return count;
}
```

---

## 3.3 Iterating Over Set Bits

Extremely common in competitive programming and graph algorithms (bitmask DP).

```cpp
// Iterate over all set bits of a mask
void iterateSetBits(int mask) {
    while (mask) {
        int bit = mask & (-mask);          // isolate lowest set bit
        int pos = __builtin_ctz(mask);     // position of that bit
        
        // process bit at position pos
        
        mask &= (mask - 1);                // clear lowest set bit
    }
}

// Iterate over all subsets of a mask (VERY important for bitmask DP)
void iterateSubsets(int mask) {
    for (int sub = mask; sub > 0; sub = (sub - 1) & mask) {
        // process subset 'sub' of 'mask'
        // This visits all 2^popcount(mask) subsets, excluding empty set
    }
    // If you need the empty set too, handle sub = 0 separately
}

// Total complexity: O(3^n) to iterate all subsets of all subsets of {0..n-1}
// Because each element is either: not in mask, in mask but not in sub, in both
```

---

## 3.4 Parity

```cpp
// Parity: is the number of set bits odd (1) or even (0)?

// Method 1: using popcount
int parity = __builtin_popcount(n) & 1;

// Method 2: XOR-fold (fast)
uint32_t x = n;
x ^= x >> 16;
x ^= x >> 8;
x ^= x >> 4;
x ^= x >> 2;
x ^= x >> 1;
int parity = x & 1;

// Built-in parity
int p = __builtin_parity(n);  // GCC: returns parity of popcount
```

---

## 3.5 Bit Reversal

```cpp
// Reverse all 32 bits of n
uint32_t reverseBits(uint32_t n) {
    n = ((n & 0xAAAAAAAA) >> 1)  | ((n & 0x55555555) << 1);  // swap odd/even bits
    n = ((n & 0xCCCCCCCC) >> 2)  | ((n & 0x33333333) << 2);  // swap 2-bit pairs
    n = ((n & 0xF0F0F0F0) >> 4)  | ((n & 0x0F0F0F0F) << 4);  // swap nibbles
    n = ((n & 0xFF00FF00) >> 8)  | ((n & 0x00FF00FF) << 8);  // swap bytes
    n = (n >> 16) | (n << 16);                                  // swap halves
    return n;
}

// C++20
#include <bit>
uint32_t r = std::byteswap(n); // reverses bytes (not bits)
```

---

# PART 4: COMPETITIVE PROGRAMMING PATTERNS

---

## 4.1 Bitmask Representation of Sets

Represent a subset of {0, 1, ..., n-1} as an integer where bit i = 1 means element i is in the set.

```cpp
// n elements, subsets are integers in [0, 2^n - 1]

int n = 5;           // 5 elements: {0, 1, 2, 3, 4}
int S = 0b10110;     // subset {1, 2, 4}

// Add element i to set S
S |= (1 << i);

// Remove element i from set S
S &= ~(1 << i);

// Check if element i is in S
bool has = (S >> i) & 1;

// Check if S is empty
bool empty = (S == 0);

// Size of S (number of elements)
int size = __builtin_popcount(S);

// Complement of S within universe U = (1<<n)-1
int comp = S ^ ((1 << n) - 1);

// Union of S and T
int unionST = S | T;

// Intersection of S and T
int intersect = S & T;

// Difference S \ T (elements in S but not T)
int diff = S & ~T;

// Check if S is a subset of T
bool isSubset = (S & T) == S;

// Full set (all n elements)
int full = (1 << n) - 1;
```

---

## 4.2 Bitmask DP

### Traveling Salesman Problem (TSP)

Classic O(2^n * n^2) bitmask DP.

```cpp
// dp[mask][i] = min cost to visit exactly the cities in mask,
//               ending at city i
// mask is a bitmask of visited cities

int n;  // number of cities
int dist[20][20];
int dp[1 << 20][20];
const int INF = 1e9;

int tsp() {
    // Initialize
    for (int mask = 0; mask < (1 << n); mask++)
        fill(dp[mask], dp[mask] + n, INF);
    
    dp[1][0] = 0;  // Start at city 0 (bit 0 set)
    
    for (int mask = 1; mask < (1 << n); mask++) {
        for (int u = 0; u < n; u++) {
            if (!(mask & (1 << u))) continue;  // u not in mask
            if (dp[mask][u] == INF) continue;
            
            for (int v = 0; v < n; v++) {
                if (mask & (1 << v)) continue;  // v already visited
                int newMask = mask | (1 << v);
                dp[newMask][v] = min(dp[newMask][v], dp[mask][u] + dist[u][v]);
            }
        }
    }
    
    int full = (1 << n) - 1;
    int ans = INF;
    for (int u = 1; u < n; u++) {
        if (dp[full][u] != INF)
            ans = min(ans, dp[full][u] + dist[u][0]);
    }
    return ans;
}
```

### Subset Sum DP

```cpp
// Can we partition numbers into two equal-sum subsets?
// dp is a bitmask of achievable sums

bool canPartition(vector<int>& nums) {
    int total = 0;
    for (int x : nums) total += x;
    if (total % 2 != 0) return false;
    int target = total / 2;
    
    // dp[s] = true if sum s is achievable
    // Use a bitset for O(n * W / 64) time
    bitset<100001> dp;
    dp[0] = 1;
    for (int x : nums) {
        dp |= dp << x;  // try adding x to every existing sum
    }
    return dp[target];
}
```

### Profile DP (Grid Tiling)

Common in tiling problems. The state is a bitmask representing the "profile" of the boundary between placed and unplaced pieces.

### Assignment Problem / Matching

```cpp
// Minimum cost perfect matching using bitmask DP
// dp[mask] = min cost to assign the first popcount(mask) workers
//            to the workers indicated by mask

int n;  // n workers, n tasks
int cost[20][20];
int dp[1 << 20];

int minCostMatching() {
    fill(dp, dp + (1 << n), INT_MAX);
    dp[0] = 0;
    
    for (int mask = 0; mask < (1 << n); mask++) {
        if (dp[mask] == INT_MAX) continue;
        int worker = __builtin_popcount(mask);  // which worker we're assigning
        if (worker == n) continue;
        
        for (int task = 0; task < n; task++) {
            if (mask & (1 << task)) continue;  // task already assigned
            int newMask = mask | (1 << task);
            dp[newMask] = min(dp[newMask], dp[mask] + cost[worker][task]);
        }
    }
    return dp[(1 << n) - 1];
}
```

---

## 4.3 Sum Over Subsets (SOS DP) / Zeta Transform

Compute f[mask] = sum of g[sub] for all subsets sub of mask. Naive O(3^n), SOS DP O(n * 2^n).

```cpp
// SOS DP — after this, dp[mask] = sum of g[sub] for all subsets sub of mask
void sos(vector<int>& dp, int n) {
    for (int i = 0; i < n; i++) {
        for (int mask = 0; mask < (1 << n); mask++) {
            if (mask & (1 << i)) {
                dp[mask] += dp[mask ^ (1 << i)];
            }
        }
    }
}

// This is equivalent to the "subset sum" or "zeta transform" in the GF(2)^n lattice.
// The inverse is the Mobius transform (inclusion-exclusion):
void mobius(vector<int>& dp, int n) {
    for (int i = 0; i < n; i++) {
        for (int mask = 0; mask < (1 << n); mask++) {
            if (mask & (1 << i)) {
                dp[mask] -= dp[mask ^ (1 << i)];
            }
        }
    }
}

// Application: AND convolution, OR convolution
// OR convolution: c[k] = sum of a[i]*b[j] for all i|j == k
// Step 1: zeta transform a and b
// Step 2: pointwise multiply
// Step 3: inverse zeta (mobius) transform
```

---

## 4.4 XOR Basis / Linear Basis (Gaussian Elimination over GF(2))

Find the linear basis of a set of integers under XOR. Used to find the maximum XOR of a subset, check if a value is achievable as XOR of a subset, etc.

```cpp
struct LinearBasis {
    static const int MAXLOG = 60;
    long long basis[MAXLOG];
    int sz;
    
    LinearBasis() : sz(0) { fill(basis, basis + MAXLOG, 0); }
    
    // Insert x into the basis
    bool insert(long long x) {
        for (int i = MAXLOG - 1; i >= 0; i--) {
            if (!((x >> i) & 1)) continue;
            if (!basis[i]) {
                basis[i] = x;
                sz++;
                return true;  // inserted (increases rank)
            }
            x ^= basis[i];
        }
        return false;  // x is in the span of existing basis
    }
    
    // Max XOR of any subset (can also XOR with some starting value)
    long long maxXor(long long start = 0) {
        long long result = start;
        for (int i = MAXLOG - 1; i >= 0; i--) {
            result = max(result, result ^ basis[i]);
        }
        return result;
    }
    
    // Min nonzero XOR of any subset
    long long minXor() {
        for (int i = 0; i < MAXLOG; i++) {
            if (basis[i]) return basis[i];
        }
        return 0;
    }
    
    // Can x be represented as XOR of some subset?
    bool canRepresent(long long x) {
        for (int i = MAXLOG - 1; i >= 0; i--) {
            if (!((x >> i) & 1)) continue;
            if (!basis[i]) return false;
            x ^= basis[i];
        }
        return true;  // x == 0 now means it's representable
    }
    
    // k-th smallest XOR value achievable (0-indexed, 0 is always achievable)
    // First reduce basis to row echelon form
    long long kthSmallest(long long k) {
        // Reduce to reduced row echelon form
        vector<long long> b;
        for (int i = 0; i < MAXLOG; i++) {
            if (!basis[i]) continue;
            long long x = basis[i];
            for (int j = i - 1; j >= 0; j--) {
                if ((x >> j) & 1) x ^= basis[j];
            }
            b.push_back(x);
        }
        // Now kth smallest (sorted) XOR
        // b.size() == rank, total achievable values == 2^rank
        if (k >= (1LL << b.size())) return -1;  // k out of range
        long long result = 0;
        for (int i = 0; i < (int)b.size(); i++) {
            if ((k >> i) & 1) result ^= b[i];
        }
        return result;
    }
};
```

---

## 4.5 Finding Unique Elements Using XOR

### Single Number — One element appears once, rest appear twice

```cpp
int singleNumber(vector<int>& nums) {
    int result = 0;
    for (int x : nums) result ^= x;
    return result;
    // All pairs cancel (a ^ a = 0), leaving the unique element
}
```

### Two Unique Elements — Two elements appear once, rest appear twice

```cpp
pair<int,int> twoUniqueNumbers(vector<int>& nums) {
    int xorAll = 0;
    for (int x : nums) xorAll ^= x;
    // xorAll = a ^ b where a, b are the two unique numbers
    
    // Find any bit where a and b differ (use lowest set bit)
    int diffBit = xorAll & (-xorAll);
    
    // Divide numbers into two groups based on that bit
    // a is in one group, b in the other
    int a = 0, b = 0;
    for (int x : nums) {
        if (x & diffBit) a ^= x;
        else b ^= x;
    }
    return {a, b};
}
```

### One element appears once, rest appear three times

```cpp
int singleNumberThree(vector<int>& nums) {
    int ones = 0, twos = 0;
    for (int x : nums) {
        ones = (ones ^ x) & ~twos;
        twos = (twos ^ x) & ~ones;
    }
    return ones;
    // ones: bits that have appeared 1 mod 3 times
    // twos: bits that have appeared 2 mod 3 times
    // When a bit appears 3 times, it's cleared from both
}
```

---

## 4.6 Gray Code

Gray code: a sequence of n-bit strings where consecutive entries differ in exactly one bit. Used in error correction, analog-to-digital converters, puzzle solutions (Tower of Hanoi).

```cpp
// Convert binary to Gray code
int toGray(int n) {
    return n ^ (n >> 1);
}

// Convert Gray code back to binary
int fromGray(int g) {
    int n = 0;
    for (; g; g >>= 1) n ^= g;
    return n;
}

// Generate all n-bit Gray codes
vector<int> grayCode(int n) {
    vector<int> result;
    for (int i = 0; i < (1 << n); i++) {
        result.push_back(i ^ (i >> 1));
    }
    return result;
}
```

---

## 4.7 Next Permutation of Bits (Gosper's Hack)

Given a bitmask with k set bits, find the next larger integer with the same number of set bits. Essential for enumerating all subsets of size k.

```cpp
int nextWithSameBitCount(int n) {
    int c = n & (-n);               // lowest set bit
    int r = n + c;                  // turn off rightmost block, set next bit
    return (((r ^ n) >> 2) / c) | r; // restore the right bits
}

// Enumerate all k-element subsets of {0..n-1}
void enumerateSubsetsOfSizeK(int n, int k) {
    int mask = (1 << k) - 1;  // k ones in lowest positions
    int limit = 1 << n;
    while (mask < limit) {
        // process mask
        
        // Gosper's Hack
        int c = mask & (-mask);
        int r = mask + c;
        mask = (((r ^ mask) >> 2) / c) | r;
    }
}
```

---

# PART 5: ADVANCED BIT MANIPULATION

---

## 5.1 Bit Fields in Structs

C++ allows packing multiple values into a single integer using bit fields.

```cpp
struct PackedFlags {
    uint32_t r : 8;   // 8 bits for red
    uint32_t g : 8;   // 8 bits for green
    uint32_t b : 8;   // 8 bits for blue
    uint32_t a : 8;   // 8 bits for alpha
};

struct GameState {
    uint32_t playerHP  : 10;  // 0-1023
    uint32_t playerMP  : 10;  // 0-1023
    uint32_t level     : 6;   // 0-63
    uint32_t flags     : 6;   // 6 boolean flags
};

// Bit fields are compiler/platform-dependent in layout.
// For portable serialization, use manual bit manipulation.
```

---

## 5.2 Manual Bit Packing for Portable Serialization

```cpp
// Pack two 16-bit values into a 32-bit integer
uint32_t pack(uint16_t hi, uint16_t lo) {
    return ((uint32_t)hi << 16) | lo;
}
void unpack(uint32_t packed, uint16_t &hi, uint16_t &lo) {
    hi = (packed >> 16) & 0xFFFF;
    lo = packed & 0xFFFF;
}

// Pack four 8-bit values into a 32-bit integer (e.g., RGBA color)
uint32_t packRGBA(uint8_t r, uint8_t g, uint8_t b, uint8_t a) {
    return ((uint32_t)r << 24) | ((uint32_t)g << 16) | ((uint32_t)b << 8) | a;
}

// Extract a field of 'len' bits starting at position 'pos'
uint32_t extractField(uint32_t n, int pos, int len) {
    return (n >> pos) & ((1u << len) - 1);
}

// Set a field of 'len' bits starting at position 'pos' to value 'val'
uint32_t setField(uint32_t n, int pos, int len, uint32_t val) {
    uint32_t mask = ((1u << len) - 1) << pos;
    return (n & ~mask) | ((val << pos) & mask);
}
```

---

## 5.3 Byte Swapping (Endianness)

```cpp
#include <cstdint>

// Swap bytes of a 16-bit integer (little-endian <-> big-endian)
uint16_t byteSwap16(uint16_t n) {
    return (n >> 8) | (n << 8);
}

// Swap bytes of a 32-bit integer
uint32_t byteSwap32(uint32_t n) {
    return ((n & 0xFF000000) >> 24)
         | ((n & 0x00FF0000) >> 8)
         | ((n & 0x0000FF00) << 8)
         | ((n & 0x000000FF) << 24);
}

// Using built-ins
uint32_t bs32 = __builtin_bswap32(n);  // GCC/Clang
uint64_t bs64 = __builtin_bswap64(n);

// C++23
#include <bit>
uint32_t bs = std::byteswap(n);

// Check system endianness at compile time (C++20)
#include <bit>
if constexpr (std::endian::native == std::endian::little) {
    // little-endian system
}
```

---

## 5.4 Rotate Bits

```cpp
// Rotate left by k positions (32-bit)
uint32_t rotateLeft(uint32_t n, int k) {
    k &= 31;  // k mod 32
    return (n << k) | (n >> (32 - k));
}

// Rotate right by k positions (32-bit)
uint32_t rotateRight(uint32_t n, int k) {
    k &= 31;
    return (n >> k) | (n << (32 - k));
}

// C++20
#include <bit>
uint32_t r = std::rotl(n, k);   // rotate left
uint32_t l = std::rotr(n, k);   // rotate right
```

---

## 5.5 Saturating Arithmetic

Clamp instead of wrapping on overflow.

```cpp
// Saturating add for uint32_t
uint32_t satAdd(uint32_t a, uint32_t b) {
    uint32_t result = a + b;
    result |= -(result < a);  // if overflow, all bits become 1 = UINT32_MAX
    return result;
}

// Saturating subtract for uint32_t
uint32_t satSub(uint32_t a, uint32_t b) {
    uint32_t result = a - b;
    result &= -(result <= a);  // if underflow, all bits become 0
    return result;
}
```

---

## 5.6 Sign Extension

```cpp
// Sign-extend an n-bit value to 32 bits
int signExtend(uint32_t x, int n) {
    int shift = 32 - n;
    return (int)(x << shift) >> shift;
    // Left shift puts the sign bit at bit 31
    // Arithmetic right shift fills with the sign bit
}

// Example: sign-extend 5-bit value 10111 (-9 in 5-bit two's complement)
// x = 0b10111 = 23
// signExtend(23, 5) = (23 << 27) >> 27 = -9
```

---

## 5.7 Finding Contiguous Bit Sequences

```cpp
// Does n have k consecutive set bits?
bool hasKConsecutiveSetBits(int n, int k) {
    for (int i = 0; i < k - 1; i++) n &= (n >> 1);
    return n != 0;
}

// Longest run of 1s in binary representation of n
int longestRun(int n) {
    int longest = 0;
    while (n) {
        n &= (n >> 1);   // AND with shifted version: only keep runs >= 2, then >= 3, etc.
        longest++;
    }
    return longest;
}
// Actually the above counts iterations, which equals the longest run length. Correct.

// Count runs of 1s (number of maximal groups of consecutive 1s)
int countRuns(int n) {
    return __builtin_popcount(n & ~(n >> 1));
    // n & ~(n>>1): left end of each run of 1s
    // Popcount that = number of runs
}
```

---

## 5.8 The std::bitset Class

For fixed-size bitsets with convenient operations.

```cpp
#include <bitset>

std::bitset<32> b1(42);          // from integer
std::bitset<32> b2("10101010");  // from string

b1.set(5);        // set bit 5
b1.reset(3);      // clear bit 3
b1.flip(2);       // toggle bit 2
b1.flip();        // toggle all bits

bool v = b1.test(5);   // check bit 5
int count = b1.count();// number of set bits
int sz = b1.size();    // total bits (32)
bool any = b1.any();   // any bit set?
bool none = b1.none(); // no bits set?
bool all = b1.all();   // all bits set?

unsigned long ul = b1.to_ulong();
unsigned long long ull = b1.to_ullong();
std::string s = b1.to_string();

// Bitwise operations on bitsets
std::bitset<32> b3 = b1 & b2;
std::bitset<32> b4 = b1 | b2;
std::bitset<32> b5 = b1 ^ b2;
std::bitset<32> b6 = ~b1;
std::bitset<32> b7 = b1 << 3;
std::bitset<32> b8 = b1 >> 3;

// Dynamic size: use std::vector<bool> or boost::dynamic_bitset
```

---

## 5.9 Compile-Time Bit Tricks with constexpr

```cpp
constexpr bool isPow2(int n) {
    return n > 0 && (n & (n - 1)) == 0;
}

constexpr int log2floor(uint32_t n) {
    return n <= 1 ? 0 : 1 + log2floor(n >> 1);
}

constexpr uint32_t nextPow2(uint32_t n) {
    n--;
    n |= n >> 1; n |= n >> 2; n |= n >> 4; n |= n >> 8; n |= n >> 16;
    return n + 1;
}

// Use in static arrays
static_assert(isPow2(64), "Must be power of 2");
int arr[nextPow2(100)];  // array of size 128
```

---

# PART 6: REAL-WORLD APPLICATIONS

---

## 6.1 Flags and Permissions Systems

The canonical real-world use of bitwise OR and AND.

```cpp
// File permissions (Unix-style)
enum Permission : uint32_t {
    NONE    = 0,
    READ    = 1 << 0,   // 0001
    WRITE   = 1 << 1,   // 0010
    EXECUTE = 1 << 2,   // 0100
    SETUID  = 1 << 3,   // 1000
};

// Application flags
enum AppFlags : uint32_t {
    FLAG_NONE       = 0,
    FLAG_VERBOSE    = 1 << 0,
    FLAG_DEBUG      = 1 << 1,
    FLAG_READONLY   = 1 << 2,
    FLAG_COMPRESSED = 1 << 3,
    FLAG_ENCRYPTED  = 1 << 4,
};

class FileDescriptor {
    uint32_t perms;
public:
    void grant(Permission p) { perms |= p; }
    void revoke(Permission p) { perms &= ~p; }
    bool can(Permission p) const { return (perms & p) == p; }
    void toggle(Permission p) { perms ^= p; }
};

// Usage
FileDescriptor fd;
fd.grant(READ | WRITE);
fd.revoke(WRITE);
if (fd.can(READ)) { /* allowed */ }

// Combining flags with OR, checking with AND is O(1) regardless of number of flags.
// vs. std::vector<bool>: bitflags use 1 bit per flag instead of 1 byte.
```

---

## 6.2 Memory Allocator — Free Bitmap

Large memory allocators track which blocks are free using a bitmap.

```cpp
class BitmapAllocator {
    static const int TOTAL_BLOCKS = 1024;
    uint64_t bitmap[TOTAL_BLOCKS / 64] = {};  // 1 = free, 0 = used
    
public:
    // Allocate one block, return its index (-1 if full)
    int allocate() {
        for (int i = 0; i < TOTAL_BLOCKS / 64; i++) {
            if (bitmap[i] == 0) continue;
            int bit = __builtin_ctzll(bitmap[i]);  // find lowest free block
            bitmap[i] &= bitmap[i] - 1;            // mark as used
            return i * 64 + bit;
        }
        return -1;  // no free block
    }
    
    // Free block at index
    void free(int idx) {
        bitmap[idx / 64] |= 1ULL << (idx % 64);
    }
    
    // Check if block is free
    bool isFree(int idx) {
        return (bitmap[idx / 64] >> (idx % 64)) & 1;
    }
    
    // Allocate n contiguous blocks (more complex, omitted for brevity)
};
```

---

## 6.3 Hash Table — Open Addressing with Bitmask

Power-of-2 sized hash tables use bitmasking instead of modulo for index calculation.

```cpp
class HashMap {
    static const int CAPACITY = 1024;    // must be power of 2
    static const int MASK = CAPACITY - 1;
    
    int keys[CAPACITY];
    int values[CAPACITY];
    bool occupied[CAPACITY];
    
    int hash(int key) {
        return key & MASK;               // equivalent to key % CAPACITY, but faster
    }
    
public:
    void insert(int key, int value) {
        int h = hash(key);
        while (occupied[h] && keys[h] != key) {
            h = (h + 1) & MASK;         // linear probing with bitmask wrap-around
        }
        keys[h] = key;
        values[h] = value;
        occupied[h] = true;
    }
    
    int get(int key) {
        int h = hash(key);
        while (occupied[h] && keys[h] != key) {
            h = (h + 1) & MASK;
        }
        return occupied[h] ? values[h] : -1;
    }
};
```

---

## 6.4 Bloom Filter

A probabilistic data structure using multiple hash functions and a bitset to test set membership. Space-efficient, allows false positives, no false negatives.

```cpp
#include <bitset>
#include <functional>

class BloomFilter {
    static const int M = 1 << 20;  // 1M bits
    std::bitset<M> bits;
    
    // k different hash functions
    size_t hash1(int x) { return std::hash<int>{}(x) & (M - 1); }
    size_t hash2(int x) { return std::hash<int>{}(x * 2654435761u) & (M - 1); }
    size_t hash3(int x) { return std::hash<int>{}(x ^ (x >> 16)) & (M - 1); }
    
public:
    void insert(int x) {
        bits.set(hash1(x));
        bits.set(hash2(x));
        bits.set(hash3(x));
    }
    
    bool mayContain(int x) {
        return bits.test(hash1(x)) && bits.test(hash2(x)) && bits.test(hash3(x));
        // All three bits set: probably in set
        // Any bit unset: definitely NOT in set
    }
};
```

---

## 6.5 Network Programming — IP Address Manipulation

```cpp
#include <cstdint>
#include <string>
#include <sstream>

// Represent IPv4 address as uint32_t
uint32_t ipToInt(int a, int b, int c, int d) {
    return ((uint32_t)a << 24) | ((uint32_t)b << 16) | ((uint32_t)c << 8) | d;
}

std::string intToIp(uint32_t ip) {
    return std::to_string((ip >> 24) & 0xFF) + "." +
           std::to_string((ip >> 16) & 0xFF) + "." +
           std::to_string((ip >>  8) & 0xFF) + "." +
           std::to_string(ip & 0xFF);
}

// Subnet masking
uint32_t subnetMask(int prefixLen) {
    return prefixLen == 0 ? 0 : (~0u) << (32 - prefixLen);
}

uint32_t networkAddress(uint32_t ip, int prefixLen) {
    return ip & subnetMask(prefixLen);
}

uint32_t broadcastAddress(uint32_t ip, int prefixLen) {
    return networkAddress(ip, prefixLen) | ~subnetMask(prefixLen);
}

bool inSameSubnet(uint32_t ip1, uint32_t ip2, int prefixLen) {
    return networkAddress(ip1, prefixLen) == networkAddress(ip2, prefixLen);
}

// Example
uint32_t ip = ipToInt(192, 168, 1, 100);
uint32_t net = networkAddress(ip, 24);  // 192.168.1.0
uint32_t bcast = broadcastAddress(ip, 24); // 192.168.1.255
```

---

## 6.6 Checksums and Hashing

### Simple XOR Checksum

```cpp
uint8_t xorChecksum(const uint8_t* data, size_t len) {
    uint8_t checksum = 0;
    for (size_t i = 0; i < len; i++) checksum ^= data[i];
    return checksum;
}
// Detects single-bit errors (any odd number of errors in same position across bytes).
// Sends corrupted bytes of different values cancel out — limitation.
```

### FNV-1a Hash (uses XOR)

```cpp
uint32_t fnv1a(const uint8_t* data, size_t len) {
    uint32_t hash = 2166136261u;  // FNV offset basis
    for (size_t i = 0; i < len; i++) {
        hash ^= data[i];
        hash *= 16777619u;  // FNV prime
    }
    return hash;
}
```

---

## 6.7 Graphics — Pixel Manipulation

```cpp
// ARGB32 pixel format
using Pixel = uint32_t;

uint8_t getAlpha(Pixel p) { return (p >> 24) & 0xFF; }
uint8_t getRed  (Pixel p) { return (p >> 16) & 0xFF; }
uint8_t getGreen(Pixel p) { return (p >>  8) & 0xFF; }
uint8_t getBlue (Pixel p) { return  p        & 0xFF; }

Pixel makePixel(uint8_t a, uint8_t r, uint8_t g, uint8_t b) {
    return ((Pixel)a << 24) | ((Pixel)r << 16) | ((Pixel)g << 8) | b;
}

// Alpha blending using bit manipulation (approximation)
Pixel alphaBlend(Pixel fg, Pixel bg) {
    uint8_t a = getAlpha(fg);
    // For each channel: result = fg * a/255 + bg * (255-a)/255
    // Fast approximation: use >> 8 instead of / 255
    uint8_t r = (getRed(fg) * a + getRed(bg) * (255 - a)) >> 8;
    uint8_t g = (getGreen(fg) * a + getGreen(bg) * (255 - a)) >> 8;
    uint8_t b = (getBlue(fg) * a + getBlue(bg) * (255 - a)) >> 8;
    return makePixel(255, r, g, b);
}

// Grayscale: luminance approximation
uint8_t toGrayscale(Pixel p) {
    // 0.299*R + 0.587*G + 0.114*B
    // Approximation: (77*R + 150*G + 29*B) >> 8
    return (77 * getRed(p) + 150 * getGreen(p) + 29 * getBlue(p)) >> 8;
}

// Bitwise image operations
// XOR two images: highlights differences
// AND with a mask: apply a stencil
// OR with a pattern: add watermark pixels
```

---

## 6.8 Data Compression — Run-Length Encoding Helpers

```cpp
// Count identical consecutive bits
int runLength(uint32_t n, int startBit) {
    int bit = (n >> startBit) & 1;
    int len = 0;
    while (startBit < 32 && ((n >> startBit) & 1) == bit) {
        len++;
        startBit++;
    }
    return len;
}

// Simple bit-level RLE encoder (conceptual)
// Group runs of 0s and 1s, encode as (bit, length) pairs
```

---

## 6.9 Cryptography Foundations

Many crypto primitives are built on bit operations.

```cpp
// XOR cipher (Vernam cipher) — theoretically perfect if key is truly random (OTP)
void xorCipher(uint8_t* data, const uint8_t* key, size_t len) {
    for (size_t i = 0; i < len; i++) {
        data[i] ^= key[i % len];
    }
}

// Detecting whether two buffers are equal in constant time (timing-attack safe)
bool constTimeEqual(const uint8_t* a, const uint8_t* b, size_t len) {
    uint8_t diff = 0;
    for (size_t i = 0; i < len; i++) diff |= (a[i] ^ b[i]);
    return diff == 0;
    // XOR is 0 for identical bytes, OR accumulates any difference
    // Single branch at the end, independent of content -> constant time
}

// Left rotation — core operation in SHA, MD5, etc.
uint32_t rotl32(uint32_t x, int n) {
    return (x << n) | (x >> (32 - n));
}

// S-box lookup with XOR (AES-style confusion)
// In real AES: SubBytes uses an 8-bit S-box, then XOR in AddRoundKey
// ShiftRows: rotate rows (bit rearrangement)
// MixColumns: GF(2^8) polynomial multiplication
```

---

## 6.10 SIMD-Style Operations Using Integers (SWAR)

Process multiple small values packed into a single wide integer simultaneously, without using actual SIMD hardware.

```cpp
// Compare 4 bytes simultaneously for equality with a target byte
uint32_t hasByteValue(uint32_t x, uint8_t target) {
    // Trick from Hacker's Delight
    uint32_t t = x ^ (0x01010101u * target);  // XOR each byte with target; match -> 00
    // Now check if any byte is zero
    return (t - 0x01010101u) & ~t & 0x80808080u;
    // Nonzero result means at least one byte equals target
}

// Find first occurrence of a null byte (strlen-style optimization)
bool hasNullByte(uint64_t x) {
    return (x - 0x0101010101010101ULL) & ~x & 0x8080808080808080ULL;
}

// Add two vectors of 4 packed 8-bit values with saturation
uint32_t addPacked8(uint32_t a, uint32_t b) {
    uint32_t mask = 0x80808080u;
    uint32_t lo   = (a & ~mask) + (b & ~mask);  // add without high bits
    uint32_t carries = (a ^ b ^ lo) & mask;
    // Saturate: if carry out, set to 0xFF in that byte
    // (Simplified; full saturation needs more logic)
    return lo | carries;
}
```

---

# PART 7: PERFORMANCE AND OPTIMIZATION

---

## 7.1 When Bit Manipulation Is and Isn't Faster

Bit manipulation is faster than alternatives when:
- Avoiding a branch (CPU branch misprediction can cost 10-20 cycles)
- Replacing a division or modulo with a shift or mask (division is 20-90 cycles vs. 1 for shift)
- Processing multiple values in a single register (SWAR)
- Reducing memory footprint (cache is king)

Bit manipulation is NOT always faster when:
- Modern compilers already optimize the readable version
- Obscured code prevents vectorization by the compiler
- Lookup tables would fit in L1 cache and be faster
- The bottleneck is memory bandwidth, not compute

Always profile. A well-optimized readable version often beats a clever bit-hack.

---

## 7.2 Branch-Free Techniques for High-Performance Code

```cpp
// Branch-free clamping to [lo, hi]
int clamp(int x, int lo, int hi) {
    x = x < lo ? lo : x;   // compiler often generates cmov (conditional move)
    x = x > hi ? hi : x;
    return x;
}

// Branchless sign function: returns -1, 0, or +1
int sign(int x) {
    return (x > 0) - (x < 0);
}

// Branchless selection
int select(bool cond, int a, int b) {
    int mask = -(int)cond;  // 0xFFFFFFFF if true, 0 if false
    return (a & mask) | (b & ~mask);
}

// Branchless modulo by 2
int mod2(int x) { return x & 1; }

// Ceiling division by power of 2
int ceilDiv(int x, int k) {
    return (x + (1 << k) - 1) >> k;
}

// Alignment (round up to multiple of 2^k)
int alignUp(int x, int k) {
    int m = (1 << k) - 1;
    return (x + m) & ~m;
}

// Check alignment
bool isAligned(int x, int k) {
    return (x & ((1 << k) - 1)) == 0;
}
```

---

## 7.3 Compiler Intrinsics Reference

```cpp
// GCC/Clang built-ins
__builtin_popcount(x)      // popcount 32-bit
__builtin_popcountl(x)     // popcount long
__builtin_popcountll(x)    // popcount long long
__builtin_clz(x)           // count leading zeros (undefined for 0)
__builtin_clzl(x)
__builtin_clzll(x)
__builtin_ctz(x)           // count trailing zeros (undefined for 0)
__builtin_ctzl(x)
__builtin_ctzll(x)
__builtin_parity(x)        // parity (1 if odd number of 1-bits)
__builtin_bswap32(x)       // byte swap 32-bit
__builtin_bswap64(x)       // byte swap 64-bit
__builtin_expect(expr, v)  // branch prediction hint

// C++20 <bit> header (portable)
std::popcount(x)           // requires unsigned type
std::countl_zero(x)        // count leading zeros (0 if x == 0)
std::countl_one(x)         // count leading ones
std::countr_zero(x)        // count trailing zeros (0 if x == 0)
std::countr_one(x)         // count trailing ones
std::has_single_bit(x)     // true if x is power of 2
std::bit_ceil(x)           // smallest power of 2 >= x
std::bit_floor(x)          // largest power of 2 <= x
std::bit_width(x)          // number of bits needed to represent x
std::rotl(x, s)            // rotate left
std::rotr(x, s)            // rotate right
std::byteswap(x)           // reverse bytes (C++23)
```

---

## 7.4 Avoiding Undefined Behavior

Undefined behavior in bit manipulation is easy to trigger and causes subtle, hard-to-find bugs.

```cpp
// UB: shifting by negative amount
int x = 1 << -1;          // UNDEFINED

// UB: shifting by >= width of type
int x = 1 << 32;          // UNDEFINED (int is 32 bits)
int x = 1 << 31;          // UNDEFINED for signed int (overflow)
unsigned x = 1u << 31;    // OK — unsigned, no overflow

// UB: left-shifting a negative value (C++17 and earlier)
int x = -1 << 2;          // UNDEFINED in C++17, well-defined in C++20

// UB: signed overflow
int x = INT_MAX + 1;       // UNDEFINED (use unsigned arithmetic)

// Safe shifting: always check
int safeLShift(int n, int k) {
    if (k < 0 || k >= 32) return 0;  // or assert
    return (unsigned)n << k;  // cast to unsigned first
}

// Use unsigned types when doing bit manipulation to avoid signed UB
// Use uint32_t, uint64_t from <cstdint> for portability

// In competitive programming, the most common UB:
// 1 << 31 when the answer requires bit 31
// Fix: 1LL << 31 or 1u << 31
```

---

# PART 8: PROBLEM PATTERNS AND SOLUTIONS

---

## 8.1 Common Leetcode/Codeforces Patterns

### Check if two integers have the same sign

```cpp
bool sameSign(int a, int b) { return (a ^ b) >= 0; }
```

### Number of bits to convert A to B

```cpp
int bitsToConvert(int a, int b) { return __builtin_popcount(a ^ b); }
// XOR gives differing bits, popcount counts them
```

### Reverse bits

```cpp
// See reverseBits() in Section 3.5
```

### Add without arithmetic operators

```cpp
int addNoBitwise(int a, int b) {
    while (b) {
        int carry = a & b;
        a = a ^ b;         // sum without carry
        b = carry << 1;    // carry shifted left
    }
    return a;
}
```

### Multiply without * operator

```cpp
int multiply(int a, int b) {
    int result = 0;
    bool negative = (a < 0) ^ (b < 0);
    long la = abs((long)a), lb = abs((long)b);
    while (lb) {
        if (lb & 1) result += la;
        la <<= 1;
        lb >>= 1;
    }
    return negative ? -result : result;
}
```

### Power of four

```cpp
bool isPow4(int n) {
    // Is power of 2 AND the set bit is at an even position (0, 2, 4, ...)
    return n > 0 && (n & (n-1)) == 0 && (n & 0x55555555) != 0;
    // 0x55555555 = 01010101... (even-position bits set)
}
```

### Bitwise AND of a range [m, n]

```cpp
int rangeBitwiseAnd(int m, int n) {
    int shift = 0;
    while (m != n) {
        m >>= 1;
        n >>= 1;
        shift++;
    }
    return m << shift;
    // Common prefix of m and n — bits that change within [m,n] become 0
}
```

### Decode XOR-encoded array

```cpp
// Given encoded[i] = arr[i] ^ arr[i+1] and arr[0], recover arr
vector<int> decode(vector<int>& encoded, int first) {
    vector<int> arr = {first};
    for (int e : encoded) arr.push_back(arr.back() ^ e);
    return arr;
}
```

---

## 8.2 XOR-Based Tricks Collection

```cpp
// 1. Find the element that appears an odd number of times
int oddOne(vector<int>& v) {
    int r = 0; for (int x : v) r ^= x; return r;
}

// 2. Find the missing number in [0..n]
int missingNumber(vector<int>& nums) {
    int n = nums.size(), result = n;
    for (int i = 0; i < n; i++) result ^= i ^ nums[i];
    return result;
}

// 3. Check if a string has all unique characters (first 26 letters only)
bool allUnique(const string& s) {
    int mask = 0;
    for (char c : s) {
        int bit = c - 'a';
        if (mask & (1 << bit)) return false;
        mask |= (1 << bit);
    }
    return true;
}

// 4. Find two non-repeating elements in array where others repeat twice
// (See Section 4.5 — two unique numbers)

// 5. In-place matrix transpose bit trick for bitset representation
// (Flipping bits across diagonal — complex, used in DB column stores)
```

---

## 8.3 Bitmask Enumeration Patterns for Competitive Programming

```cpp
// Pattern: try all 2^n subsets
for (int mask = 0; mask < (1 << n); mask++) {
    // mask represents a subset of {0..n-1}
    for (int i = 0; i < n; i++) {
        if (mask & (1 << i)) {
            // element i is in this subset
        }
    }
}

// Pattern: iterate subsets of a given mask
for (int sub = mask; sub > 0; sub = (sub - 1) & mask) {
    // process sub
}

// Pattern: iterate all masks of exactly k bits (use Gosper's Hack)
// (see Section 4.7)

// Pattern: complement of sub within mask
int comp = sub ^ mask;

// Pattern: check if mask1 is subset of mask2
bool isSubset = (mask1 & mask2) == mask1;

// Pattern: number of elements in mask
int cnt = __builtin_popcount(mask);

// Pattern: highest element in mask
int highest = 31 - __builtin_clz(mask);

// Pattern: lowest element in mask
int lowest = __builtin_ctz(mask);

// Pattern: remove lowest element from mask
mask &= mask - 1;

// Pattern: DP on subsets with transitions
// dp[new_mask] updated from dp[mask] by adding one element
for (int mask = 0; mask < (1 << n); mask++) {
    for (int i = 0; i < n; i++) {
        if (!(mask & (1 << i))) {
            int newMask = mask | (1 << i);
            dp[newMask] = min(dp[newMask], dp[mask] + cost[i]);
        }
    }
}
```

---

## 8.4 Integer Square Root Using Bit Tricks

```cpp
uint32_t isqrt(uint32_t n) {
    if (n == 0) return 0;
    uint32_t x = 0;
    uint32_t bit = 1u << 30;  // 2^30 is the highest even power of 2 <= 2^31
    while (bit > n) bit >>= 2;
    while (bit) {
        if (n >= x + bit) {
            n -= x + bit;
            x = (x >> 1) + bit;
        } else {
            x >>= 1;
        }
        bit >>= 2;
    }
    return x;
    // Computes floor(sqrt(n)) using binary long division
}
```

---

## 8.5 Hamming Distance and Error Correction Concepts

```cpp
// Hamming distance between two integers
int hammingDistance(int x, int y) {
    return __builtin_popcount(x ^ y);
    // XOR marks positions where bits differ; popcount counts them
}

// Total Hamming distance of an array
long long totalHamming(vector<int>& nums) {
    long long result = 0;
    int n = nums.size();
    for (int i = 0; i < 32; i++) {
        long long ones = 0;
        for (int x : nums) ones += (x >> i) & 1;
        result += ones * (n - ones);
        // ones * zeros = number of pairs differing at bit i
    }
    return result;
}

// Hamming(7,4) code — encode 4 data bits with 3 parity bits
// Parity bits at positions 1, 2, 4 (1-indexed, powers of 2)
// Detects and corrects 1-bit errors

uint8_t hammingEncode(uint8_t data) {  // data: 4 bits (bits 0-3)
    // Place data bits at positions 3, 5, 6, 7 (0-indexed: 2, 4, 5, 6)
    uint8_t d = ((data & 0b1000) << 3) | ((data & 0b0111) << 2) | (data & 0b0001);
    // d at bits 6,5,4 = data[3,2,1], bit 2 = data[0]... (complex mapping)
    // For brevity, see a full Hamming code reference for exact bit positioning
    // Parity: p1 covers bits 1,3,5,7; p2 covers 2,3,6,7; p4 covers 4,5,6,7
    return d;
}
```

---

# PART 9: MAGIC NUMBERS AND CONSTANTS REFERENCE

---

## 9.1 Commonly Used Masks

```cpp
// 32-bit masks
0x55555555  // 01010101... — odd bit positions
0xAAAAAAAA  // 10101010... — even bit positions
0x33333333  // 00110011... — 2-bit groups
0xCCCCCCCC  // 11001100... — 2-bit groups (complement)
0x0F0F0F0F  // 00001111... — nibbles
0xF0F0F0F0  // 11110000... — nibbles (complement)
0x00FF00FF  // even bytes
0xFF00FF00  // odd bytes
0x0000FFFF  // lower 16 bits
0xFFFF0000  // upper 16 bits
0x80808080  // MSB of each byte
0x01010101  // LSB of each byte
0x7F7F7F7F  // all bits except MSB of each byte

// 64-bit equivalents (append ULL)
0x5555555555555555ULL
0xAAAAAAAAAAAAAAAAULL
// etc.

// All zeros
0x00000000

// All ones
0xFFFFFFFF
~0u             // ~0 for unsigned — portable all-ones

// Power of 2 - 1 (n lower bits set)
(1 << n) - 1

// Highest bit of int
1u << 31        // 0x80000000
INT_MIN         // same value, signed
```

---

## 9.2 Integer Limits

```cpp
#include <climits>
#include <cstdint>

INT_MIN     // -2147483648    = 0x80000000
INT_MAX     //  2147483647    = 0x7FFFFFFF
UINT_MAX    //  4294967295    = 0xFFFFFFFF
LLONG_MIN   // -9223372036854775808
LLONG_MAX   //  9223372036854775807
ULLONG_MAX  //  18446744073709551615

// From <cstdint>
INT8_MIN, INT8_MAX
INT16_MIN, INT16_MAX
INT32_MIN, INT32_MAX
INT64_MIN, INT64_MAX
UINT8_MAX  = 255
UINT16_MAX = 65535
UINT32_MAX = 4294967295
UINT64_MAX = 18446744073709551615
```

---

# PART 10: CHECKLIST FOR PRACTICE

---

## Problems to Master Bit Manipulation

Beginner tier:
- Count set bits in an integer (multiple methods)
- Find if a number is a power of 2
- Find the only non-repeating element in an array
- Swap two numbers using XOR
- Reverse bits of a 32-bit integer
- Count bits to flip to convert A to B
- Find the missing number in [1..n]

Intermediate tier:
- Find two non-repeating elements in array (others repeat twice)
- Subsets of a set using bitmask
- Gray code sequence
- Number of 1 bits in all integers from 1 to n
- Total Hamming distance of an array
- Bitwise AND of a number range
- Maximum XOR of two numbers in an array (Trie approach)

Advanced tier:
- TSP with bitmask DP
- Assignment problem with bitmask DP
- Maximum AND of a subset (greedy + linear basis)
- Maximum XOR of a subset (linear basis over GF(2))
- SOS DP (sum over subsets)
- Count number of subsets with XOR equal to k
- Gosper's Hack — enumerate subsets of given size
- Counting set bits in a range [0..n] efficiently

Real-world / project tier:
- Implement a bitmap allocator
- Implement a Bloom filter
- IP subnet calculations
- Pixel manipulation (ARGB operations)
- XOR cipher implementation
- Hamming code encoder/decoder
- Compress/decompress run-length encoded binary data
- Implement a compact set for small universes

---

## 10.1 Debugging Bit Manipulation

```cpp
// Print binary with labels — invaluable for debugging
void debugBits(const char* label, uint32_t n, int bits = 32) {
    printf("%-20s = ", label);
    for (int i = bits - 1; i >= 0; i--) {
        printf("%d", (n >> i) & 1);
        if (i % 4 == 0 && i != 0) printf(" ");
    }
    printf(" (0x%X = %d)\n", n, (int)n);
}

// Assert bit patterns
#define CHECK_BIT(n, i, expected) \
    assert(((n >> i) & 1) == expected && "bit " #i " should be " #expected)

// Step through bit operations manually on paper for small examples
// Draw the bit grid and apply operations by hand
// Use online tools like: godbolt.org to see assembly output
```

---

## 10.2 Summary of the Most Important Tricks

```
n & (n-1)      — clear lowest set bit
n & (-n)       — isolate lowest set bit
n | (n-1)      — fill trailing zeros with ones
~n & (n+1)     — isolate lowest zero bit
n ^ (n>>1)     — binary to Gray code
a ^ b ^ a = b  — XOR self-cancellation
x ^ x = 0      — duplicate elimination
(x ^ y) < 0   — different signs
!(n & (n-1))   — n is power of 2 (n > 0)
n % (2^k) = n & (2^k - 1) — fast modulo
n * (2^k) = n << k         — fast multiply
n / (2^k) = n >> k         — fast divide (positive)
__builtin_popcount(n)      — count set bits
__builtin_ctz(n)           — lowest set bit position
__builtin_clz(n)           — highest set bit (= 31 - clz)
```