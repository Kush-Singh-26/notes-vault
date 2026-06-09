---
title: "Python Reference: Basics to Intermediate Mastery"
description: "python reference"
---

## PART 1: THE LANGUAGE CORE

---

### 1.1 The Python Object Model

Everything in Python is an object. An integer, a function, a class, a module — all are first-class objects with identity (`id()`), type (`type()`), and value. This is not a metaphor; it has concrete consequences.

```python
x = 42
print(type(x))        # <class 'int'>
print(id(x))          # memory address
print(isinstance(x, int))  # True
```

Variables are names bound to objects, not boxes containing values. Assignment is binding, not copying.

```python
a = [1, 2, 3]
b = a           # b is bound to the same object
b.append(4)
print(a)        # [1, 2, 3, 4] — same object
```

To copy: use `a.copy()` or `list(a)` for a shallow copy, `copy.deepcopy(a)` for a deep copy.

---

### 1.2 Types and Literals

**Numeric Types**

```python
x = 10          # int (arbitrary precision)
y = 3.14        # float (C double, 64-bit)
z = 2 + 3j      # complex
b = 0b1010      # binary literal → 10
o = 0o17        # octal literal → 15
h = 0xFF        # hex literal → 255
big = 1_000_000 # underscore separator for readability

from decimal import Decimal
d = Decimal("0.1")  # exact decimal arithmetic

from fractions import Fraction
f = Fraction(1, 3)  # exact rational arithmetic
```

**Boolean**

`bool` is a subclass of `int`. `True == 1` and `False == 0`.

```python
print(True + True)   # 2
print(bool(0))       # False
print(bool([]))      # False
print(bool(""))      # False
print(bool(None))    # False
# Everything else is truthy
```

**Strings**

```python
s = "hello"
s = 'hello'
s = """multiline
string"""
s = r"\n is literal backslash n"     # raw string
s = b"bytes"                          # bytes literal
s = f"value is {1 + 1}"              # f-string

# Strings are immutable sequences of Unicode code points
print(s[0])           # indexing
print(s[1:4])         # slicing
print(s[::-1])        # reverse
print(len(s))
print(s.upper())
print(s.lower())
print(s.strip())      # removes leading/trailing whitespace
print(s.lstrip("xy")) # strips chars from left
print(s.split(","))   # returns list
print(",".join(["a","b","c"]))  # "a,b,c"
print(s.replace("l", "r"))
print(s.startswith("he"))
print(s.endswith("lo"))
print(s.find("ll"))   # index of first occurrence, -1 if not found
print(s.index("ll"))  # same but raises ValueError if not found
print(s.count("l"))
print(s.zfill(10))    # pad with zeros on left
print(s.center(20, "-"))
print(s.isdigit())
print(s.isalpha())
print(s.isalnum())
print(s.encode("utf-8"))  # str → bytes
```

**f-strings (PEP 498, extended in 3.12)**

```python
name = "Kush"
val = 3.14159

print(f"{name!r}")          # repr
print(f"{name!s}")          # str
print(f"{val:.2f}")         # 2 decimal places
print(f"{val:10.3f}")       # width 10, 3 decimal places
print(f"{1000000:,}")       # 1,000,000
print(f"{255:#010x}")       # 0x000000ff
print(f"{val = }")          # Python 3.8+ debugging: val = 3.14159
```

**None**

The null object. Singleton. Test with `is None`, never `== None`.

---

### 1.3 Collections

**list** — mutable, ordered, heterogeneous

```python
lst = [1, 2, 3]
lst.append(4)
lst.extend([5, 6])
lst.insert(0, 0)       # insert at index
lst.remove(3)          # removes first occurrence
lst.pop()              # removes and returns last
lst.pop(0)             # removes and returns at index
lst.index(2)           # first index of value
lst.count(2)
lst.reverse()
lst.sort()
lst.sort(key=lambda x: -x)  # custom key
lst.sort(reverse=True)
sorted_copy = sorted(lst)    # returns new list, doesn't mutate
lst.clear()
lst2 = lst.copy()
lst3 = lst[:]         # also a shallow copy
```

**Slicing** — works on all sequences

```python
lst = list(range(10))
lst[2:5]       # [2, 3, 4]
lst[::2]       # every other element
lst[::-1]      # reversed
lst[1:8:2]     # start:stop:step
```

**tuple** — immutable, ordered. Use for heterogeneous data, returning multiple values, dict keys.

```python
t = (1, 2, 3)
t = 1, 2, 3       # parentheses optional
single = (1,)     # trailing comma required for single-element tuple
a, b, c = t       # unpacking
a, *rest = t      # star unpacking → rest = [2, 3]
*init, last = t
```

**dict** — mutable, ordered (Python 3.7+), key-value mapping

```python
d = {"a": 1, "b": 2}
d = dict(a=1, b=2)
d["c"] = 3
d.get("x")          # None if missing
d.get("x", 0)       # default if missing
d.setdefault("y", []).append(1)  # init if missing
del d["a"]
d.pop("b")          # removes and returns
d.pop("z", None)    # with default
d.update({"e": 5})
d.update(f=6)

for k, v in d.items(): ...
for k in d.keys(): ...
for v in d.values(): ...

# dict comprehension
sq = {x: x**2 for x in range(5)}

# merging (Python 3.9+)
merged = d1 | d2      # new dict
d1 |= d2              # update in place
```

**set** — mutable, unordered, unique elements

```python
s = {1, 2, 3}
s = set([1, 2, 2, 3])  # {1, 2, 3}
s.add(4)
s.remove(4)       # raises KeyError if missing
s.discard(99)     # no error if missing
s.pop()           # arbitrary element

a | b             # union
a & b             # intersection
a - b             # difference
a ^ b             # symmetric difference
a <= b            # a is subset of b
a >= b            # a is superset of b
frozenset([1,2])  # immutable set, hashable, usable as dict key
```

**collections module — extended containers**

```python
from collections import defaultdict, Counter, OrderedDict, deque, namedtuple, ChainMap

# defaultdict — never raises KeyError
dd = defaultdict(list)
dd["key"].append(1)

dd = defaultdict(int)
dd["count"] += 1

# Counter — multiset / frequency map
c = Counter("abracadabra")
c.most_common(3)
c["a"]
c + Counter("aab")

# deque — O(1) append/pop from both ends
dq = deque([1, 2, 3], maxlen=5)
dq.appendleft(0)
dq.popleft()
dq.rotate(2)

# namedtuple — immutable struct
Point = namedtuple("Point", ["x", "y"])
p = Point(1, 2)
print(p.x, p.y)
print(p._asdict())
p2 = p._replace(x=10)

# ChainMap — unified view of multiple dicts
cm = ChainMap({"a": 1}, {"b": 2})
cm["a"]  # 1
```

---

### 1.4 Control Flow

**Conditionals**

```python
if x > 0:
    print("positive")
elif x == 0:
    print("zero")
else:
    print("negative")

# ternary
result = "yes" if condition else "no"

# match-case (Python 3.10+)
match command:
    case "quit":
        quit()
    case "help":
        show_help()
    case str(s) if s.startswith("go"):
        move(s)
    case _:
        print("unknown")
```

**Loops**

```python
for i in range(10): ...
for i in range(2, 10, 2): ...  # start, stop, step
for i, v in enumerate(lst, start=1): ...
for a, b in zip(lst1, lst2): ...
for a, b, c in zip(l1, l2, l3): ...

# zip behavior: stops at shortest. Use itertools.zip_longest for padding.

while condition:
    if x: break
    if y: continue
    ...
else:
    # runs if loop completed without break
    print("no break")

for i in range(10):
    ...
else:
    # also runs if for loop didn't break
    ...
```

**Exception Handling**

```python
try:
    result = 10 / 0
except ZeroDivisionError:
    print("division by zero")
except (TypeError, ValueError) as e:
    print(f"type or value error: {e}")
except Exception as e:
    print(f"unexpected: {e}")
else:
    # runs only if no exception
    print("success")
finally:
    # always runs
    print("cleanup")

# Raising
raise ValueError("bad input")
raise RuntimeError("failed") from original_exception  # chained

# Custom exceptions
class AppError(Exception):
    def __init__(self, message, code=None):
        super().__init__(message)
        self.code = code

try:
    ...
except AppError as e:
    print(e.code)
```

**Exception hierarchy (important subset)**

```
BaseException
 ├── SystemExit
 ├── KeyboardInterrupt
 └── Exception
      ├── ArithmeticError
      │    └── ZeroDivisionError
      ├── LookupError
      │    ├── IndexError
      │    └── KeyError
      ├── TypeError
      ├── ValueError
      ├── AttributeError
      ├── NameError
      ├── OSError (IOError, FileNotFoundError, PermissionError, etc.)
      ├── RuntimeError
      ├── StopIteration
      ├── NotImplementedError
      └── ImportError
```

Catch specific exceptions, not bare `except:` or `except Exception:` unless you truly need it. Never catch `BaseException` unless writing a top-level error handler.

---

### 1.5 Functions

```python
def greet(name, greeting="Hello"):
    """Docstring: describe what the function does."""
    return f"{greeting}, {name}!"

# *args and **kwargs
def func(*args, **kwargs):
    for arg in args: print(arg)
    for k, v in kwargs.items(): print(k, v)

# keyword-only arguments (after *)
def func(a, b, *, key=None):
    ...

# positional-only arguments (before /)
def func(a, b, /, c, d):
    ...

# full example
def full(pos_only, /, normal, *, kw_only):
    ...

# unpacking at call site
args = [1, 2, 3]
func(*args)
kwargs = {"a": 1, "b": 2}
func(**kwargs)
```

**Scope (LEGB rule)**

Python resolves names in this order: Local → Enclosing → Global → Built-in.

```python
x = "global"

def outer():
    x = "enclosing"
    def inner():
        nonlocal x        # refers to enclosing x
        x = "modified"
    inner()
    print(x)              # "modified"

def modify_global():
    global x
    x = "changed"
```

**Lambda**

```python
square = lambda x: x ** 2
add = lambda x, y: x + y

# useful inline but prefer def for anything non-trivial
lst.sort(key=lambda item: item[1])
```

**First-class functions**

```python
def apply(func, value):
    return func(value)

apply(str.upper, "hello")   # "HELLO"

funcs = [str.upper, str.lower, str.strip]
for f in funcs:
    print(f("  Hello  "))
```

**Closures**

```python
def make_adder(n):
    def adder(x):
        return x + n   # captures n from enclosing scope
    return adder

add5 = make_adder(5)
add5(3)  # 8
```

**Decorators**

```python
import functools

def my_decorator(func):
    @functools.wraps(func)   # preserves __name__, __doc__
    def wrapper(*args, **kwargs):
        print("before")
        result = func(*args, **kwargs)
        print("after")
        return result
    return wrapper

@my_decorator
def say_hello():
    """Says hello."""
    print("hello")

# Decorator with arguments
def repeat(n):
    def decorator(func):
        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            for _ in range(n):
                result = func(*args, **kwargs)
            return result
        return wrapper
    return decorator

@repeat(3)
def greet():
    print("hi")
```

**Useful built-in decorators**

- `@staticmethod` — no self/cls, just a regular function in a class namespace
- `@classmethod` — receives cls, factory pattern
- `@property` — getter/setter as attribute access

---

### 1.6 Comprehensions

**List comprehension**

```python
squares = [x**2 for x in range(10)]
evens = [x for x in range(20) if x % 2 == 0]
flat = [x for row in matrix for x in row]  # nested, order matters
```

**Dict comprehension**

```python
inv = {v: k for k, v in d.items()}
```

**Set comprehension**

```python
unique_lengths = {len(word) for word in words}
```

**Generator expression**

```python
gen = (x**2 for x in range(10))  # lazy, not materialized
sum(x**2 for x in range(10))     # no extra brackets needed
```

When to use which: list comp when you need the full list in memory; generator when you just iterate once; set/dict comp for those structures.

---

### 1.7 Iterators and Generators

**The iterator protocol**

Any object that implements `__iter__` (returns self) and `__next__` (returns next value, raises `StopIteration` when exhausted) is an iterator.

```python
class Counter:
    def __init__(self, low, high):
        self.current = low
        self.high = high

    def __iter__(self):
        return self

    def __next__(self):
        if self.current > self.high:
            raise StopIteration
        val = self.current
        self.current += 1
        return val

for n in Counter(1, 5):
    print(n)
```

**Generator functions**

```python
def fibonacci():
    a, b = 0, 1
    while True:
        yield a
        a, b = b, a + b

gen = fibonacci()
next(gen)   # 0
next(gen)   # 1
next(gen)   # 1

# generator with send()
def accumulator():
    total = 0
    while True:
        value = yield total
        if value is None:
            break
        total += value

acc = accumulator()
next(acc)        # prime it
acc.send(10)     # 10
acc.send(20)     # 30
```

**yield from**

```python
def chain(*iterables):
    for it in iterables:
        yield from it

list(chain([1,2], [3,4], [5]))  # [1, 2, 3, 4, 5]
```

**itertools — essential**

```python
import itertools

itertools.count(start=0, step=1)     # infinite counter
itertools.cycle([1, 2, 3])           # infinite cycle
itertools.repeat(x, n)               # repeat x n times

itertools.chain([1,2], [3,4])        # chain iterables
itertools.chain.from_iterable(lst_of_lsts)

itertools.islice(gen, 10)            # take first 10 from infinite gen
itertools.takewhile(pred, it)
itertools.dropwhile(pred, it)
itertools.filterfalse(pred, it)

itertools.product([1,2], [3,4])      # Cartesian product
itertools.permutations([1,2,3], 2)
itertools.combinations([1,2,3], 2)
itertools.combinations_with_replacement([1,2,3], 2)

itertools.groupby(sorted_data, key=lambda x: x["dept"])
itertools.accumulate([1,2,3,4], lambda a,b: a*b)  # running product
itertools.starmap(func, [(1,2),(3,4)])

itertools.zip_longest([1,2,3], [4,5], fillvalue=0)
itertools.batched(range(10), 3)  # Python 3.12+
```

**functools**

```python
import functools

functools.reduce(lambda a, b: a + b, [1,2,3,4])  # 10

functools.partial(int, base=2)  # new callable with base=2 pre-filled
bin_to_int = functools.partial(int, base=2)
bin_to_int("1010")  # 10

functools.lru_cache(maxsize=128)   # memoization decorator
functools.cache                    # unbounded lru_cache (3.9+)

@functools.lru_cache(maxsize=None)
def fib(n):
    if n < 2: return n
    return fib(n-1) + fib(n-2)

functools.total_ordering  # fill in missing comparison methods
```

---

## PART 2: OBJECT-ORIENTED PYTHON

---

### 2.1 Classes

```python
class Animal:
    species_count = 0           # class variable

    def __init__(self, name, sound):
        self.name = name        # instance variable
        self.sound = sound
        Animal.species_count += 1

    def speak(self):
        return f"{self.name} says {self.sound}"

    def __repr__(self):
        return f"Animal(name={self.name!r}, sound={self.sound!r})"

    def __str__(self):
        return self.name

    @classmethod
    def from_dict(cls, data):
        return cls(data["name"], data["sound"])

    @staticmethod
    def is_valid_name(name):
        return isinstance(name, str) and len(name) > 0
```

`__repr__` should be unambiguous and ideally eval-able. `__str__` is the human-readable form. When `__str__` is missing, Python falls back to `__repr__`.

**Inheritance**

```python
class Dog(Animal):
    def __init__(self, name, breed):
        super().__init__(name, "woof")
        self.breed = breed

    def speak(self):                     # override
        return super().speak() + "!"    # extend

    def fetch(self):
        return f"{self.name} fetches!"
```

**Multiple inheritance and MRO**

```python
class A:
    def method(self): return "A"

class B(A):
    def method(self): return "B"

class C(A):
    def method(self): return "C"

class D(B, C):
    pass

D().method()   # "B" — MRO: D → B → C → A
print(D.__mro__)
```

Python uses C3 linearization. Always use `super()` in cooperative multiple inheritance so the MRO chain is maintained.

---

### 2.2 Special (Dunder) Methods

```python
class Vector:
    def __init__(self, x, y):
        self.x, self.y = x, y

    # Representation
    def __repr__(self): return f"Vector({self.x}, {self.y})"
    def __str__(self): return f"({self.x}, {self.y})"

    # Arithmetic
    def __add__(self, other): return Vector(self.x + other.x, self.y + other.y)
    def __sub__(self, other): return Vector(self.x - other.x, self.y - other.y)
    def __mul__(self, scalar): return Vector(self.x * scalar, self.y * scalar)
    def __rmul__(self, scalar): return self.__mul__(scalar)  # 3 * v
    def __neg__(self): return Vector(-self.x, -self.y)
    def __abs__(self): return (self.x**2 + self.y**2) ** 0.5

    # Comparison
    def __eq__(self, other): return self.x == other.x and self.y == other.y
    def __lt__(self, other): return abs(self) < abs(other)
    def __hash__(self): return hash((self.x, self.y))  # required if __eq__ defined

    # Container
    def __len__(self): return 2
    def __getitem__(self, idx):
        if idx == 0: return self.x
        if idx == 1: return self.y
        raise IndexError(idx)
    def __iter__(self): yield self.x; yield self.y
    def __contains__(self, val): return val in (self.x, self.y)

    # Callable
    def __call__(self, *args): ...

    # Context manager
    def __enter__(self): return self
    def __exit__(self, exc_type, exc_val, exc_tb): return False  # don't suppress

    # Boolean
    def __bool__(self): return self.x != 0 or self.y != 0
```

---

### 2.3 Properties and Descriptors

```python
class Circle:
    def __init__(self, radius):
        self._radius = radius

    @property
    def radius(self):
        return self._radius

    @radius.setter
    def radius(self, value):
        if value < 0:
            raise ValueError("Radius must be non-negative")
        self._radius = value

    @radius.deleter
    def radius(self):
        del self._radius

    @property
    def area(self):
        import math
        return math.pi * self._radius ** 2
```

**Descriptor protocol**

```python
class Validated:
    def __set_name__(self, owner, name):
        self.name = name

    def __get__(self, obj, objtype=None):
        if obj is None: return self   # class-level access
        return obj.__dict__.get(self.name)

    def __set__(self, obj, value):
        if not isinstance(value, int):
            raise TypeError(f"{self.name} must be int")
        obj.__dict__[self.name] = value

class MyClass:
    x = Validated()
    y = Validated()
```

---

### 2.4 dataclasses (Python 3.7+)

```python
from dataclasses import dataclass, field, asdict, astuple

@dataclass
class Point:
    x: float
    y: float
    z: float = 0.0
    label: str = field(default="origin", repr=True)
    tags: list = field(default_factory=list)  # mutable defaults need factory

    def distance(self):
        return (self.x**2 + self.y**2 + self.z**2) ** 0.5

@dataclass(frozen=True)   # immutable, hashable
class ImmutablePoint:
    x: float
    y: float

@dataclass(order=True)    # auto-generates __lt__, __le__, etc.
class Sortable:
    priority: int
    name: str

p = Point(1.0, 2.0)
asdict(p)    # {'x': 1.0, 'y': 2.0, 'z': 0.0, 'label': 'origin', 'tags': []}
astuple(p)   # (1.0, 2.0, 0.0, 'origin', [])
```

---

### 2.5 Abstract Base Classes

```python
from abc import ABC, abstractmethod

class Shape(ABC):
    @abstractmethod
    def area(self) -> float: ...

    @abstractmethod
    def perimeter(self) -> float: ...

    def describe(self):
        return f"Area: {self.area()}, Perimeter: {self.perimeter()}"

class Rectangle(Shape):
    def __init__(self, w, h):
        self.w, self.h = w, h

    def area(self): return self.w * self.h
    def perimeter(self): return 2 * (self.w + self.h)

# Shape()  → TypeError: can't instantiate abstract class
```

---

## PART 3: TYPE SYSTEM AND ANNOTATIONS

---

### 3.1 Type Hints

Python's type hints are fully optional and runtime-ignored (unless you use a validator). They serve documentation and static analysis tools (mypy, pyright).

```python
from typing import Optional, Union, List, Dict, Tuple, Set, Any
from typing import Callable, Iterator, Generator, Sequence, Mapping
from typing import TypeVar, Generic, Protocol
from collections.abc import Iterable   # preferred over typing.Iterable in 3.9+

# Basic annotations
def greet(name: str) -> str:
    return f"Hello, {name}"

def process(data: list[int]) -> dict[str, int]:  # lowercase generics (3.9+)
    return {"sum": sum(data)}

# Optional and Union
def find(lst: list[str], key: str) -> str | None:   # 3.10+ union syntax
    ...

def coerce(x: int | float | str) -> float:
    ...

# Callable
def apply(func: Callable[[int, int], int], a: int, b: int) -> int:
    return func(a, b)

# TypeVar — generic functions
T = TypeVar("T")
def first(lst: list[T]) -> T:
    return lst[0]
```

**Protocol — structural subtyping (duck typing with types)**

```python
from typing import Protocol

class Drawable(Protocol):
    def draw(self) -> None: ...

def render(obj: Drawable) -> None:
    obj.draw()

class Circle:
    def draw(self): print("drawing circle")  # no explicit inheritance needed
```

**TypedDict**

```python
from typing import TypedDict

class Config(TypedDict):
    host: str
    port: int
    debug: bool

def start(config: Config): ...
```

---

## PART 4: THE STANDARD LIBRARY

---

### 4.1 os and os.path

```python
import os

os.getcwd()
os.chdir("/tmp")
os.listdir(".")
os.makedirs("a/b/c", exist_ok=True)
os.remove("file.txt")
os.rmdir("dir")
os.rename("old", "new")
os.environ["HOME"]
os.environ.get("MY_VAR", "default")
os.getenv("MY_VAR", "default")
os.getpid()
os.getuid()    # Unix only
os.cpu_count()

os.path.join("/home", "user", "file.txt")
os.path.exists("/etc/hosts")
os.path.isfile("x.py")
os.path.isdir("/tmp")
os.path.abspath("relative/path")
os.path.dirname("/home/user/file.txt")   # "/home/user"
os.path.basename("/home/user/file.txt")  # "file.txt"
os.path.splitext("file.tar.gz")          # ("file.tar", ".gz")
os.path.expanduser("~/documents")
os.path.getsize("file.txt")
os.stat("file.txt").st_mtime             # modification time
```

**os.walk**

```python
for dirpath, dirnames, filenames in os.walk("/some/dir"):
    for fname in filenames:
        full_path = os.path.join(dirpath, fname)
        print(full_path)

# topdown=False walks bottom-up
# followlinks=True follows symlinks
```

---

### 4.2 pathlib (preferred over os.path)

```python
from pathlib import Path

p = Path("/home/user/documents")
p = Path.home() / "documents" / "file.txt"
p = Path.cwd()

p.exists()
p.is_file()
p.is_dir()
p.name          # "file.txt"
p.stem          # "file"
p.suffix        # ".txt"
p.suffixes      # [".tar", ".gz"] for file.tar.gz
p.parent        # Path("/home/user/documents")
p.parents[1]    # Path("/home/user")

p.mkdir(parents=True, exist_ok=True)
p.unlink()              # delete file
p.rmdir()               # delete empty dir
p.rename(target)
p.replace(target)       # like rename but overwrites

p.read_text(encoding="utf-8")
p.write_text("content", encoding="utf-8")
p.read_bytes()
p.write_bytes(b"data")

# Globbing
list(p.glob("*.py"))
list(p.rglob("*.py"))       # recursive
list(p.glob("**/*.py"))     # same

for child in p.iterdir():   # immediate children
    print(child)

p.stat().st_size
p.stat().st_mtime
p.resolve()                 # absolute path, resolves symlinks
p.relative_to(base)         # relative path
str(p)                      # convert to string
```

---

### 4.3 sys

```python
import sys

sys.argv             # command-line arguments list; argv[0] is script name
sys.argv[1:]         # actual arguments

sys.exit(0)          # exit with code (0 = success)
sys.exit("message")  # exit with message to stderr

sys.path             # module search path (list of strings)
sys.path.insert(0, "/custom/path")

sys.stdin
sys.stdout
sys.stderr
sys.stdout.write("no newline")
sys.stderr.write("error\n")

sys.version
sys.version_info     # named tuple: major, minor, micro
sys.platform         # "linux", "darwin", "win32"

sys.maxsize          # max int size for platform
sys.getsizeof(obj)   # size in bytes
sys.getrecursionlimit()
sys.setrecursionlimit(10000)

sys.modules          # dict of loaded modules
sys.builtin_module_names
```

---

### 4.4 io and File I/O

**Text files**

```python
# Always use context managers (with)
with open("file.txt", "r", encoding="utf-8") as f:
    content = f.read()

with open("file.txt", "r", encoding="utf-8") as f:
    for line in f:              # memory efficient
        print(line.rstrip("\n"))

with open("file.txt", "r", encoding="utf-8") as f:
    lines = f.readlines()       # list of lines

with open("file.txt", "w", encoding="utf-8") as f:
    f.write("hello\n")
    f.writelines(["line1\n", "line2\n"])

with open("file.txt", "a", encoding="utf-8") as f:
    f.write("appended\n")
```

Mode flags: `r` (read), `w` (write/truncate), `a` (append), `x` (exclusive create), `r+` (read+write), `b` (binary mode modifier).

**Binary files**

```python
with open("data.bin", "rb") as f:
    header = f.read(4)
    f.seek(100)
    block = f.read(512)
    pos = f.tell()
```

**io.StringIO and io.BytesIO** — in-memory file objects

```python
import io

buffer = io.StringIO()
buffer.write("line 1\n")
buffer.write("line 2\n")
buffer.getvalue()        # "line 1\nline 2\n"
buffer.seek(0)
buffer.read()

# useful for testing code that writes to files
buf = io.BytesIO(b"\x00\x01\x02")
buf.read(1)   # b'\x00'
```

---

### 4.5 json

```python
import json

# Serialization
data = {"name": "Kush", "scores": [95, 87, 92], "active": True}
s = json.dumps(data)                          # to string
s = json.dumps(data, indent=2)               # pretty print
s = json.dumps(data, sort_keys=True)
s = json.dumps(data, ensure_ascii=False)     # allow non-ASCII

with open("data.json", "w") as f:
    json.dump(data, f, indent=2)             # to file

# Deserialization
parsed = json.loads(s)
with open("data.json") as f:
    parsed = json.load(f)

# Custom encoder
class DateEncoder(json.JSONEncoder):
    def default(self, obj):
        import datetime
        if isinstance(obj, datetime.date):
            return obj.isoformat()
        return super().default(obj)

json.dumps({"date": datetime.date.today()}, cls=DateEncoder)

# Custom decoder
def decode_hook(dct):
    if "__datetime__" in dct:
        import datetime
        return datetime.datetime.fromisoformat(dct["value"])
    return dct

json.loads(s, object_hook=decode_hook)
```

---

### 4.6 csv

```python
import csv

# Reading
with open("data.csv", newline="", encoding="utf-8") as f:
    reader = csv.reader(f)
    header = next(reader)
    for row in reader:
        print(row)   # list of strings

with open("data.csv", newline="", encoding="utf-8") as f:
    reader = csv.DictReader(f)
    for row in reader:
        print(row["name"], row["age"])  # dict keyed by header

# Writing
with open("out.csv", "w", newline="", encoding="utf-8") as f:
    writer = csv.writer(f)
    writer.writerow(["name", "age"])
    writer.writerows([["Alice", 30], ["Bob", 25]])

with open("out.csv", "w", newline="", encoding="utf-8") as f:
    writer = csv.DictWriter(f, fieldnames=["name", "age"])
    writer.writeheader()
    writer.writerow({"name": "Alice", "age": 30})

# Dialects
csv.list_dialects()   # ['excel', 'excel-tab', 'unix']

# Custom dialect
csv.register_dialect("pipes", delimiter="|", quotechar='"')
```

---

### 4.7 re — Regular Expressions

```python
import re

# Compiling (cache if using same pattern many times)
pattern = re.compile(r"\d+")

# Main functions
re.match(r"\d+", "123abc")       # match at start of string only
re.search(r"\d+", "abc123def")   # first match anywhere
re.findall(r"\d+", "a1 b22 c3")  # ['1', '22', '3'] — list of strings
re.finditer(r"\d+", "a1 b2")     # iterator of match objects
re.sub(r"\d+", "NUM", "a1 b2")   # 'a NUM b NUM'
re.subn(r"\d+", "NUM", "a1 b2")  # ('a NUM b NUM', 2)
re.split(r"\s+", "a  b   c")     # ['a', 'b', 'c']
re.fullmatch(r"\d+", "123")      # must match entire string

# Match objects
m = re.search(r"(\w+)@(\w+)\.(\w+)", "user@example.com")
m.group(0)    # full match
m.group(1)    # first group: "user"
m.group(2)    # "example"
m.groups()    # ('user', 'example', 'com')
m.start()
m.end()
m.span()

# Named groups
m = re.search(r"(?P<user>\w+)@(?P<domain>\w+)", "user@example.com")
m.group("user")
m.groupdict()

# Flags
re.IGNORECASE or re.I
re.MULTILINE or re.M   # ^ and $ match line boundaries
re.DOTALL or re.S      # . matches \n too
re.VERBOSE or re.X     # allow whitespace and comments in pattern

pattern = re.compile(r"""
    (\d{4})   # year
    -
    (\d{2})   # month
    -
    (\d{2})   # day
""", re.VERBOSE)
```

**Common patterns**

```
\d       digit [0-9]
\D       non-digit
\w       word char [a-zA-Z0-9_]
\W       non-word
\s       whitespace
\S       non-whitespace
\b       word boundary
.        any char except \n
^        start of string (or line with MULTILINE)
$        end of string
*        0 or more (greedy)
+        1 or more (greedy)
?        0 or 1
{n}      exactly n
{n,m}    between n and m
*?       non-greedy
+?       non-greedy
[abc]    character class
[^abc]   negated class
(abc)    capturing group
(?:abc)  non-capturing group
(?=abc)  positive lookahead
(?!abc)  negative lookahead
(?<=ab)  positive lookbehind
(?<!ab)  negative lookbehind
a|b      alternation
```

---

### 4.8 datetime

```python
import datetime

# Current time
now = datetime.datetime.now()
utcnow = datetime.datetime.utcnow()
today = datetime.date.today()

# Creating
d = datetime.date(2024, 6, 15)
t = datetime.time(14, 30, 0)
dt = datetime.datetime(2024, 6, 15, 14, 30, 0)

# Formatting
dt.strftime("%Y-%m-%d %H:%M:%S")
dt.strftime("%d/%m/%Y")
datetime.datetime.strptime("2024-06-15", "%Y-%m-%d")

# ISO format
dt.isoformat()                               # "2024-06-15T14:30:00"
datetime.datetime.fromisoformat("2024-06-15T14:30:00")

# Arithmetic with timedelta
delta = datetime.timedelta(days=7, hours=2, minutes=30)
future = dt + delta
diff = dt2 - dt1       # timedelta
diff.days
diff.total_seconds()

# Timezone aware (use zoneinfo in 3.9+)
from zoneinfo import ZoneInfo
tz = ZoneInfo("Asia/Kolkata")
dt_aware = datetime.datetime.now(tz)
dt_utc = dt_aware.astimezone(ZoneInfo("UTC"))
```

---

### 4.9 subprocess

```python
import subprocess

# Run and wait
result = subprocess.run(["ls", "-la"], capture_output=True, text=True)
result.stdout     # string
result.stderr
result.returncode

# Check for failure
result = subprocess.run(["ls", "/nonexistent"],
                        capture_output=True, text=True,
                        check=True)  # raises CalledProcessError on non-zero exit

# Shell mode (use carefully — injection risk with user input)
result = subprocess.run("ls -la | grep py", shell=True,
                        capture_output=True, text=True)

# Input piping
result = subprocess.run(["cat"], input="hello world",
                        capture_output=True, text=True)

# Timeout
try:
    result = subprocess.run(["sleep", "10"], timeout=2)
except subprocess.TimeoutExpired:
    print("timed out")

# Popen for more control
proc = subprocess.Popen(
    ["python3", "-c", "import sys; print(sys.stdin.read())"],
    stdin=subprocess.PIPE, stdout=subprocess.PIPE, text=True
)
stdout, stderr = proc.communicate(input="hello")
proc.returncode

# Streaming output
with subprocess.Popen(["tail", "-f", "/var/log/syslog"],
                      stdout=subprocess.PIPE, text=True) as proc:
    for line in proc.stdout:
        print(line, end="")
        if some_condition: proc.terminate()
```

---

### 4.10 argparse

```python
import argparse

parser = argparse.ArgumentParser(
    description="Process some files.",
    formatter_class=argparse.RawDescriptionHelpFormatter,
    epilog="Example:\n  prog.py input.txt -o output.txt -v"
)

# Positional argument
parser.add_argument("input", help="input file")

# Optional arguments
parser.add_argument("-o", "--output", default="out.txt", help="output file")
parser.add_argument("-v", "--verbose", action="store_true")
parser.add_argument("-n", "--count", type=int, default=10)
parser.add_argument("--log-level",
                    choices=["DEBUG", "INFO", "WARNING", "ERROR"],
                    default="INFO")
parser.add_argument("files", nargs="+")       # one or more
parser.add_argument("--tags", nargs="*")      # zero or more
parser.add_argument("--size", nargs=2, type=int, metavar=("W", "H"))
parser.add_argument("--flag", action="store_const", const=42)
parser.add_argument("--no-cache", dest="cache", action="store_false")

# Subcommands
subparsers = parser.add_subparsers(dest="command")
push_parser = subparsers.add_parser("push", help="push changes")
push_parser.add_argument("remote", default="origin")
pull_parser = subparsers.add_parser("pull")

args = parser.parse_args()
# or: args = parser.parse_args(["--verbose", "input.txt"])

if args.command == "push":
    ...

# Argument groups
group = parser.add_argument_group("connection options")
group.add_argument("--host", default="localhost")
group.add_argument("--port", type=int, default=8080)

# Mutually exclusive
mutex = parser.add_mutually_exclusive_group()
mutex.add_argument("--json", action="store_true")
mutex.add_argument("--csv", action="store_true")
```

---

### 4.11 logging

```python
import logging

# Basic config (good for scripts)
logging.basicConfig(
    level=logging.DEBUG,
    format="%(asctime)s %(levelname)-8s %(name)s: %(message)s",
    datefmt="%Y-%m-%d %H:%M:%S",
    handlers=[
        logging.StreamHandler(),
        logging.FileHandler("app.log")
    ]
)

# Module-level loggers (best practice)
logger = logging.getLogger(__name__)

logger.debug("detailed info")
logger.info("general info")
logger.warning("something unexpected but not an error")
logger.error("an error occurred")
logger.critical("program may not continue")
logger.exception("error with traceback")   # use inside except block

# Structured logging
logger.info("request received", extra={"user": "kush", "ip": "127.0.0.1"})

# Programmatic setup
handler = logging.StreamHandler()
handler.setLevel(logging.WARNING)
formatter = logging.Formatter("%(levelname)s: %(message)s")
handler.setFormatter(formatter)

logger = logging.getLogger("myapp")
logger.addHandler(handler)
logger.setLevel(logging.DEBUG)

# Rotating file handler
from logging.handlers import RotatingFileHandler, TimedRotatingFileHandler

handler = RotatingFileHandler("app.log", maxBytes=5_000_000, backupCount=3)
handler = TimedRotatingFileHandler("app.log", when="midnight", backupCount=7)
```

---

### 4.12 contextlib

```python
from contextlib import contextmanager, suppress, redirect_stdout
import contextlib

# Creating context managers with generator
@contextmanager
def temp_directory():
    import tempfile, shutil
    tmpdir = tempfile.mkdtemp()
    try:
        yield tmpdir
    finally:
        shutil.rmtree(tmpdir)

with temp_directory() as d:
    print(d)

# suppress — swallow specific exceptions
with contextlib.suppress(FileNotFoundError):
    os.remove("might_not_exist.txt")

# redirect_stdout
import io
buf = io.StringIO()
with contextlib.redirect_stdout(buf):
    print("this goes to buf")
buf.getvalue()

# ExitStack — dynamic context managers
with contextlib.ExitStack() as stack:
    files = [stack.enter_context(open(f)) for f in file_list]
    # all files closed on exit
```

---

### 4.13 threading

```python
import threading

def worker(n):
    print(f"worker {n} starting")
    # ... work
    print(f"worker {n} done")

threads = []
for i in range(5):
    t = threading.Thread(target=worker, args=(i,))
    t.daemon = True   # dies with main thread
    threads.append(t)
    t.start()

for t in threads:
    t.join()   # wait for completion
    t.join(timeout=2.0)  # with timeout

# Thread subclass
class MyThread(threading.Thread):
    def __init__(self, data):
        super().__init__()
        self.data = data
        self.result = None

    def run(self):
        self.result = sum(self.data)

t = MyThread([1, 2, 3])
t.start()
t.join()
print(t.result)

# Locks
lock = threading.Lock()

with lock:
    shared_resource += 1

# also: lock.acquire(), lock.release()

# RLock — reentrant (same thread can acquire multiple times)
rlock = threading.RLock()

# Semaphore
sem = threading.Semaphore(3)  # max 3 threads at once
with sem:
    ...

# Event — signaling between threads
event = threading.Event()
event.set()
event.clear()
event.wait()           # blocks until set
event.wait(timeout=5)
event.is_set()

# Condition
cond = threading.Condition()
with cond:
    while not data_ready:
        cond.wait()
    ...

with cond:
    data_ready = True
    cond.notify_all()

# Thread-local storage
local = threading.local()
local.value = 42   # each thread has its own .value

# Timer
t = threading.Timer(5.0, function, args=[...])
t.start()
t.cancel()

# The GIL
# Python's Global Interpreter Lock means only one thread runs Python bytecode
# at a time. Threading is good for I/O-bound tasks. For CPU-bound, use
# multiprocessing or concurrent.futures.ProcessPoolExecutor.
```

---

### 4.14 concurrent.futures

```python
from concurrent.futures import ThreadPoolExecutor, ProcessPoolExecutor, as_completed
import concurrent.futures

# Thread pool — for I/O bound
with ThreadPoolExecutor(max_workers=8) as executor:
    futures = [executor.submit(fetch_url, url) for url in urls]
    for future in as_completed(futures):
        try:
            result = future.result()
        except Exception as e:
            print(f"error: {e}")

# map — simpler interface
with ThreadPoolExecutor(max_workers=4) as executor:
    results = list(executor.map(process, items))
    # raises the first exception; use submit+as_completed for better control

# Process pool — for CPU bound
with ProcessPoolExecutor(max_workers=os.cpu_count()) as executor:
    futures = {executor.submit(cpu_task, x): x for x in data}
    for future in as_completed(futures):
        original = futures[future]
        result = future.result()

# Future API
f = executor.submit(func, arg)
f.done()
f.running()
f.result(timeout=10)   # blocks
f.exception()
f.cancel()             # if not started yet
f.add_done_callback(callback)

# wait
done, not_done = concurrent.futures.wait(
    futures,
    timeout=30,
    return_when=concurrent.futures.FIRST_COMPLETED
)
```

---

### 4.15 multiprocessing

```python
import multiprocessing as mp

def task(x):
    return x * x

if __name__ == "__main__":  # required on Windows/macOS
    # Pool
    with mp.Pool(processes=4) as pool:
        results = pool.map(task, range(10))
        results = pool.starmap(func, [(1,2), (3,4)])  # unpack args

    # Process
    p = mp.Process(target=task, args=(5,))
    p.start()
    p.join()
    p.terminate()

    # Shared memory
    val = mp.Value("i", 0)    # shared integer
    arr = mp.Array("d", 10)   # shared double array

    with val.get_lock():
        val.value += 1

    # Queue and Pipe
    q = mp.Queue()
    q.put(item)
    item = q.get()
    q.empty()

    parent_conn, child_conn = mp.Pipe()
    parent_conn.send(data)
    child_conn.recv()

    # Manager — shared objects across processes
    with mp.Manager() as manager:
        shared_list = manager.list()
        shared_dict = manager.dict()
        shared_lock = manager.Lock()
```

---

### 4.16 asyncio

```python
import asyncio

# Coroutines
async def fetch(url):
    await asyncio.sleep(1)   # non-blocking sleep
    return f"result from {url}"

# Running
asyncio.run(fetch("http://example.com"))  # 3.7+

# Concurrent tasks
async def main():
    results = await asyncio.gather(
        fetch("url1"),
        fetch("url2"),
        fetch("url3")
    )
    return results

# gather with exception handling
results = await asyncio.gather(*coros, return_exceptions=True)

# create_task — schedule without awaiting immediately
async def main():
    task1 = asyncio.create_task(fetch("url1"))
    task2 = asyncio.create_task(fetch("url2"))
    await asyncio.sleep(0)   # yield control
    r1 = await task1
    r2 = await task2

# asyncio.wait
done, pending = await asyncio.wait(
    tasks,
    timeout=5.0,
    return_when=asyncio.FIRST_COMPLETED
)

# Timeout
async def main():
    try:
        result = await asyncio.wait_for(fetch("url"), timeout=3.0)
    except asyncio.TimeoutError:
        print("timed out")

# Async iteration
async def gen():
    for i in range(5):
        await asyncio.sleep(0.1)
        yield i

async def main():
    async for value in gen():
        print(value)

# Async context manager
class AsyncDB:
    async def __aenter__(self):
        await self.connect()
        return self

    async def __aexit__(self, *args):
        await self.close()

async with AsyncDB() as db:
    ...

# Queue
q = asyncio.Queue(maxsize=10)
await q.put(item)
item = await q.get()
q.task_done()
await q.join()

# Locks and primitives
lock = asyncio.Lock()
async with lock:
    ...

sem = asyncio.Semaphore(5)
async with sem:
    ...

event = asyncio.Event()
await event.wait()
event.set()

# Running sync code in executor
loop = asyncio.get_event_loop()
result = await loop.run_in_executor(None, blocking_func, arg)
```

---

### 4.17 hashlib and secrets

```python
import hashlib
import hmac
import secrets

# Hashing
hashlib.md5(b"data").hexdigest()
hashlib.sha256(b"data").hexdigest()
hashlib.sha512(b"data").hexdigest()
hashlib.blake2b(b"data").hexdigest()

# File hashing
h = hashlib.sha256()
with open("file.bin", "rb") as f:
    for chunk in iter(lambda: f.read(65536), b""):
        h.update(chunk)
print(h.hexdigest())

# Password hashing — use hashlib.scrypt or hashlib.pbkdf2_hmac
dk = hashlib.pbkdf2_hmac("sha256", b"password", b"salt", 100_000)

# HMAC
mac = hmac.new(b"key", b"message", hashlib.sha256)
mac.hexdigest()
hmac.compare_digest(mac1, mac2)  # constant-time comparison

# secrets — cryptographically secure randomness
secrets.token_hex(32)       # 64-char hex string
secrets.token_bytes(32)     # 32 random bytes
secrets.token_urlsafe(32)   # URL-safe base64
secrets.choice(population)
secrets.randbelow(100)

# Not cryptographic, but useful: random
import random
random.random()              # [0.0, 1.0)
random.uniform(1, 10)
random.randint(1, 10)        # inclusive
random.choice([1, 2, 3])
random.choices([1,2,3], weights=[10,1,1], k=5)
random.sample([1,2,3,4,5], 3)   # without replacement
random.shuffle(lst)             # in place
random.seed(42)                 # reproducibility
```

---

### 4.18 struct — Binary Data Packing

```python
import struct

# Format chars: b=int8, B=uint8, h=int16, H=uint16,
#               i=int32, I=uint32, l=int32, L=uint32,
#               q=int64, Q=uint64, f=float32, d=float64,
#               s=char, p=pascal string, x=pad byte
# Endian: < little, > big, ! network (big), = native, @ native aligned

packed = struct.pack("<IHH", 0x1234, 0x56, 0x78)
struct.unpack("<IHH", packed)    # (4660, 86, 120)
struct.calcsize("<IHH")          # 8

# Struct class (reuse format)
fmt = struct.Struct(">4sHH")
fmt.pack(b"RIFF", 100, 200)
fmt.unpack(data)
fmt.size

# Reading binary files
with open("file.bin", "rb") as f:
    magic, version, flags = struct.unpack(">4sHH", f.read(8))
```

---

### 4.19 tempfile and shutil

```python
import tempfile
import shutil

# tempfile
with tempfile.NamedTemporaryFile(suffix=".txt", delete=True) as f:
    f.write(b"data")
    f.name   # path

with tempfile.TemporaryDirectory() as tmpdir:
    print(tmpdir)   # deleted on exit

f = tempfile.mkstemp(suffix=".dat")  # (fd, path)
d = tempfile.mkdtemp()               # returns path, manual cleanup

tempfile.gettempdir()

# shutil
shutil.copy("src", "dst")             # file, preserves permissions
shutil.copy2("src", "dst")            # file, preserves all metadata
shutil.copyfile("src", "dst")         # file content only
shutil.copytree("srcdir", "dstdir")   # recursive directory copy
shutil.rmtree("dir")                  # recursive directory delete
shutil.move("src", "dst")             # move/rename
shutil.which("python3")               # find executable on PATH
shutil.disk_usage("/")                # namedtuple: total, used, free
shutil.make_archive("archive", "zip", "srcdir")   # create zip/tar
shutil.unpack_archive("archive.zip", "dstdir")
```

---

### 4.20 glob and fnmatch

```python
import glob

glob.glob("*.py")
glob.glob("**/*.py", recursive=True)
glob.glob("/home/user/[Dd]ocuments/*.txt")
glob.iglob("**/*.log", recursive=True)  # iterator

import fnmatch
fnmatch.fnmatch("file.py", "*.py")       # True
fnmatch.fnmatch("file.PY", "*.py")       # case-sensitive on Unix
fnmatch.filter(["a.py", "b.txt"], "*.py")  # ["a.py"]
```

---

### 4.21 pickle

```python
import pickle

# Serialize
with open("data.pkl", "wb") as f:
    pickle.dump(obj, f)

serialized = pickle.dumps(obj)

# Deserialize
with open("data.pkl", "rb") as f:
    obj = pickle.load(f)

obj = pickle.loads(serialized)

# NEVER unpickle data from untrusted sources.
# For config/data persistence, prefer json, toml, or sqlite.
# pickle protocol 5 (Python 3.8+) is most efficient.
pickle.dumps(obj, protocol=5)
```

---

### 4.22 sqlite3

```python
import sqlite3

conn = sqlite3.connect("data.db")
conn = sqlite3.connect(":memory:")   # in-memory DB

conn.row_factory = sqlite3.Row       # rows behave like dicts

cur = conn.cursor()

cur.execute("""
    CREATE TABLE IF NOT EXISTS users (
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        name TEXT NOT NULL,
        age INTEGER
    )
""")
conn.commit()

# Parameterized queries — always use ? placeholders, never f-strings
cur.execute("INSERT INTO users (name, age) VALUES (?, ?)", ("Alice", 30))
cur.executemany("INSERT INTO users VALUES (NULL, ?, ?)", data_list)
conn.commit()

cur.execute("SELECT * FROM users WHERE age > ?", (25,))
row = cur.fetchone()
rows = cur.fetchall()
for row in cur.execute("SELECT * FROM users"):
    print(row["name"])  # dict-like if row_factory set

cur.execute("UPDATE users SET age = ? WHERE name = ?", (31, "Alice"))
cur.execute("DELETE FROM users WHERE id = ?", (1,))
conn.commit()

conn.close()

# Context manager
with sqlite3.connect("data.db") as conn:
    conn.execute("INSERT INTO users VALUES (NULL, ?, ?)", ("Bob", 25))
    # auto-commits on success, rolls back on exception

# Transactions
conn.isolation_level = None   # autocommit mode
conn.execute("BEGIN")
conn.execute("ROLLBACK")
conn.execute("COMMIT")
```

---

## PART 5: SCRIPTING PATTERNS

---

### 5.1 Script Structure

```python
#!/usr/bin/env python3
"""
Script description.

Usage:
    script.py [options] <input>
"""

import sys
import os
import argparse
import logging
from pathlib import Path


logger = logging.getLogger(__name__)


def parse_args():
    parser = argparse.ArgumentParser(description=__doc__)
    parser.add_argument("input", type=Path)
    parser.add_argument("-v", "--verbose", action="store_true")
    return parser.parse_args()


def setup_logging(verbose: bool = False):
    level = logging.DEBUG if verbose else logging.INFO
    logging.basicConfig(
        level=level,
        format="%(asctime)s %(levelname)-8s %(message)s",
        datefmt="%H:%M:%S",
    )


def main():
    args = parse_args()
    setup_logging(args.verbose)

    if not args.input.exists():
        logger.error("input file not found: %s", args.input)
        sys.exit(1)

    logger.info("processing %s", args.input)
    # ... logic
    logger.info("done")


if __name__ == "__main__":
    main()
```

---

### 5.2 Configuration Patterns

**Environment variables**

```python
import os
from dataclasses import dataclass

@dataclass
class Config:
    host: str = os.environ.get("APP_HOST", "localhost")
    port: int = int(os.environ.get("APP_PORT", "8080"))
    debug: bool = os.environ.get("APP_DEBUG", "false").lower() == "true"
    db_url: str = os.environ.get("DATABASE_URL", "sqlite:///app.db")

config = Config()
```

**INI files via configparser**

```python
import configparser

config = configparser.ConfigParser()
config.read("config.ini")

config["DEFAULT"]["timeout"] = "30"
config["database"]["host"] = "localhost"
config["database"]["port"] = "5432"

with open("config.ini", "w") as f:
    config.write(f)

config.get("database", "host")
config.getint("database", "port")
config.getboolean("app", "debug", fallback=False)

for section in config.sections():
    for key, value in config[section].items():
        print(f"[{section}] {key} = {value}")
```

**TOML (Python 3.11+)**

```python
import tomllib  # read-only in stdlib

with open("config.toml", "rb") as f:
    config = tomllib.load(f)

config["database"]["host"]
```

---

### 5.3 Stdin, Pipes, and Streams

```python
import sys

# Reading stdin
for line in sys.stdin:
    print(line.rstrip())

# Check if stdin is a pipe
if not sys.stdin.isatty():
    data = sys.stdin.read()

# Prompt only if interactive
if sys.stdin.isatty():
    name = input("Enter name: ")

# Line buffering for piped output
print("progress...", flush=True)
sys.stdout.flush()

# Disable stdout buffering
import functools
print = functools.partial(print, flush=True)

# Binary stdin
data = sys.stdin.buffer.read()
sys.stdout.buffer.write(b"\x00\x01")
```

---

### 5.4 Process Management Patterns

```python
import subprocess
import sys
import os
import signal

# Run command, stream output in real time
def run_streaming(cmd):
    proc = subprocess.Popen(cmd, stdout=subprocess.PIPE,
                            stderr=subprocess.STDOUT, text=True)
    for line in proc.stdout:
        print(line, end="")
    proc.wait()
    return proc.returncode

# Retry wrapper
import time
def run_with_retry(cmd, retries=3, delay=1.0):
    for attempt in range(retries):
        result = subprocess.run(cmd, capture_output=True, text=True)
        if result.returncode == 0:
            return result
        if attempt < retries - 1:
            time.sleep(delay * (2 ** attempt))  # exponential backoff
    raise RuntimeError(f"command failed after {retries} attempts")

# Signal handling
def handler(signum, frame):
    print("SIGTERM received, cleaning up")
    sys.exit(0)

signal.signal(signal.SIGTERM, handler)
signal.signal(signal.SIGINT, handler)

# Daemonizing (Unix)
def daemonize():
    if os.fork():
        sys.exit(0)
    os.setsid()
    if os.fork():
        sys.exit(0)
    os.chdir("/")
    sys.stdin = open(os.devnull)
    sys.stdout = open("/var/log/app.log", "a")
    sys.stderr = sys.stdout
```

---

### 5.5 File Processing Patterns

```python
from pathlib import Path
import fileinput

# Process multiple files like Unix tools
for line in fileinput.input(files=sys.argv[1:]):
    if fileinput.isfirstline():
        print(f"--- {fileinput.filename()} ---")
    print(line, end="")

# In-place editing
with fileinput.input(files=["config.txt"], inplace=True) as f:
    for line in f:
        print(line.replace("old", "new"), end="")

# Atomic write (write to temp, then replace)
import tempfile
import os

def atomic_write(path: Path, content: str):
    dir_ = path.parent
    with tempfile.NamedTemporaryFile("w", dir=dir_,
                                     delete=False,
                                     suffix=".tmp") as tmp:
        tmp.write(content)
        tmp_path = tmp.name
    os.replace(tmp_path, path)   # atomic on POSIX

# Watching for file changes (polling)
import time

def watch_file(path: Path, interval=1.0):
    mtime = path.stat().st_mtime
    while True:
        time.sleep(interval)
        new_mtime = path.stat().st_mtime
        if new_mtime != mtime:
            mtime = new_mtime
            yield path
```

---

### 5.6 Networking — socket and urllib

```python
import socket

# TCP client
with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as s:
    s.connect(("example.com", 80))
    s.sendall(b"GET / HTTP/1.0\r\nHost: example.com\r\n\r\n")
    data = s.recv(4096)

# TCP server
with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as s:
    s.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
    s.bind(("0.0.0.0", 9000))
    s.listen(5)
    while True:
        conn, addr = s.accept()
        with conn:
            data = conn.recv(1024)
            conn.sendall(b"HTTP/1.0 200 OK\r\n\r\nHello")

# UDP
with socket.socket(socket.AF_INET, socket.SOCK_DGRAM) as s:
    s.sendto(b"hello", ("127.0.0.1", 9001))
    data, addr = s.recvfrom(4096)

socket.gethostname()
socket.gethostbyname("example.com")
socket.getfqdn()

# urllib
from urllib.request import urlopen, Request
from urllib.parse import urlencode, urljoin, urlparse, quote, unquote
from urllib.error import URLError, HTTPError

# GET
with urlopen("https://httpbin.org/get", timeout=10) as r:
    body = r.read().decode("utf-8")
    r.status
    r.getheader("Content-Type")

# POST
data = urlencode({"key": "value"}).encode()
req = Request("https://httpbin.org/post", data=data, method="POST")
req.add_header("Content-Type", "application/x-www-form-urlencoded")
req.add_header("User-Agent", "my-script/1.0")

try:
    with urlopen(req, timeout=10) as r:
        print(r.read().decode())
except HTTPError as e:
    print(e.code, e.reason)
except URLError as e:
    print(e.reason)

# URL manipulation
urlparse("https://example.com:8080/path?q=1#frag")
urljoin("https://example.com/a/b", "../c")   # "https://example.com/a/c"
quote("hello world")            # "hello%20world"
unquote("hello%20world")        # "hello world"

# http.server — simple file server
# python3 -m http.server 8000
```

---

### 5.7 Caching and Memoization

```python
import functools
import time

# lru_cache
@functools.lru_cache(maxsize=256)
def expensive(n):
    return sum(range(n))

expensive.cache_info()    # CacheInfo(hits=0, misses=1, maxsize=256, currsize=1)
expensive.cache_clear()

# Time-based cache
def timed_cache(ttl=60):
    def decorator(func):
        cache = {}
        @functools.wraps(func)
        def wrapper(*args):
            now = time.time()
            if args in cache and now - cache[args][1] < ttl:
                return cache[args][0]
            result = func(*args)
            cache[args] = (result, now)
            return result
        wrapper.cache_clear = lambda: cache.clear()
        return wrapper
    return decorator

@timed_cache(ttl=300)
def fetch_config(key):
    ...
```

---

### 5.8 Dataclass Patterns for Config/Data

```python
from dataclasses import dataclass, field
from typing import Optional
import os

@dataclass
class DatabaseConfig:
    host: str = field(default_factory=lambda: os.environ.get("DB_HOST", "localhost"))
    port: int = field(default_factory=lambda: int(os.environ.get("DB_PORT", "5432")))
    name: str = field(default_factory=lambda: os.environ.get("DB_NAME", "app"))
    user: str = field(default_factory=lambda: os.environ.get("DB_USER", "postgres"))
    password: str = field(default_factory=lambda: os.environ.get("DB_PASS", ""))
    pool_size: int = 10
    timeout: float = 30.0

    @property
    def url(self):
        return f"postgresql://{self.user}:{self.password}@{self.host}:{self.port}/{self.name}"

    def __post_init__(self):
        if not self.name:
            raise ValueError("DB_NAME is required")
```

---

## PART 6: ADVANCED LANGUAGE FEATURES

---

### 6.1 Metaclasses

```python
# A class is an instance of its metaclass.
# type is the default metaclass of all classes.

type("MyClass", (object,), {"x": 1, "greet": lambda self: "hi"})

class Meta(type):
    def __new__(mcs, name, bases, namespace):
        # Enforce method naming convention
        for key in namespace:
            if callable(namespace[key]) and not key.startswith("_"):
                if not key.islower():
                    raise TypeError(f"Method {key} must be lowercase")
        return super().__new__(mcs, name, bases, namespace)

    def __init__(cls, name, bases, namespace):
        super().__init__(name, bases, namespace)
        cls._registry = {}

class MyBase(metaclass=Meta):
    pass

# __init_subclass__ — simpler hook, often replaces metaclasses
class Plugin:
    _registry = {}

    def __init_subclass__(cls, plugin_name=None, **kwargs):
        super().__init_subclass__(**kwargs)
        if plugin_name:
            Plugin._registry[plugin_name] = cls

class CSVPlugin(Plugin, plugin_name="csv"):
    def process(self): ...

Plugin._registry["csv"]  # CSVPlugin
```

---

### 6.2 __slots__

```python
class Point:
    __slots__ = ("x", "y")   # no __dict__, saves memory, faster attribute access

    def __init__(self, x, y):
        self.x, self.y = x, y

# p.z = 1 → AttributeError
# p.__dict__ → AttributeError

# With inheritance
class Point3D(Point):
    __slots__ = ("z",)   # adds z slot, inherits x, y
```

Use `__slots__` when creating millions of small objects. Not for general use.

---

### 6.3 Descriptors (advanced)

```python
class TypeChecked:
    def __set_name__(self, owner, name):
        self.public_name = name
        self.private_name = "_" + name

    def __get__(self, obj, objtype=None):
        if obj is None:
            return self
        return getattr(obj, self.private_name, None)

    def __set__(self, obj, value):
        self._validate(value)
        setattr(obj, self.private_name, value)

    def _validate(self, value):
        pass

class Integer(TypeChecked):
    def _validate(self, value):
        if not isinstance(value, int):
            raise TypeError(f"Expected int, got {type(value).__name__}")

class Positive(Integer):
    def _validate(self, value):
        super()._validate(value)
        if value <= 0:
            raise ValueError("Must be positive")

class Account:
    balance = Positive()
    owner_id = Integer()
```

---

### 6.4 __getattr__, __getattribute__, __missing__

```python
# __getattr__ — only called when normal lookup fails
class Flexible:
    def __getattr__(self, name):
        return f"no attribute {name}"

# __getattribute__ — called on EVERY attribute access
class Traced:
    def __getattribute__(self, name):
        print(f"accessing {name}")
        return object.__getattribute__(self, name)   # always use super() to avoid recursion

# __missing__ — called by dict subclass when key is missing
class DefaultDict(dict):
    def __missing__(self, key):
        self[key] = key.upper()
        return self[key]
```

---

### 6.5 Context Variables (contextvars)

```python
# Thread-safe, coroutine-safe context-local state
from contextvars import ContextVar

request_id: ContextVar[str] = ContextVar("request_id", default="none")

async def handle_request(rid):
    token = request_id.set(rid)
    try:
        await process()
    finally:
        request_id.reset(token)

async def process():
    print(request_id.get())  # thread/coroutine safe
```

---

### 6.6 Protocols and Structural Typing (3.8+)

```python
from typing import Protocol, runtime_checkable

@runtime_checkable
class Closeable(Protocol):
    def close(self) -> None: ...

class File:
    def close(self): print("closing")

isinstance(File(), Closeable)  # True if @runtime_checkable

# Self type (3.11+)
from typing import Self

class Builder:
    def set_name(self, name: str) -> Self:
        self.name = name
        return self
```

---

### 6.7 Operator Module

```python
import operator

operator.add(1, 2)          # 3
operator.itemgetter(1)([10, 20, 30])  # 20
operator.attrgetter("name")(obj)
operator.methodcaller("split", ",")(s)

# Useful for key= arguments
from operator import itemgetter, attrgetter, methodcaller

data.sort(key=itemgetter(1))
data.sort(key=attrgetter("score"))
data.sort(key=itemgetter("dept", "name"))  # multi-level sort
```

---

## PART 7: TESTING

---

### 7.1 unittest

```python
import unittest

class TestMath(unittest.TestCase):
    def setUp(self):
        self.data = [1, 2, 3, 4, 5]

    def tearDown(self):
        pass

    def test_sum(self):
        self.assertEqual(sum(self.data), 15)

    def test_max(self):
        self.assertEqual(max(self.data), 5)

    def test_raises(self):
        with self.assertRaises(ZeroDivisionError):
            1 / 0

    def test_almost_equal(self):
        self.assertAlmostEqual(0.1 + 0.2, 0.3, places=10)

    def test_in(self):
        self.assertIn(3, self.data)
        self.assertNotIn(6, self.data)

    def test_none(self):
        self.assertIsNone(None)

    def test_true(self):
        self.assertTrue(len(self.data) > 0)

    @unittest.skip("not implemented")
    def test_future(self): ...

    @unittest.skipIf(sys.platform == "win32", "Unix only")
    def test_unix(self): ...

    @unittest.expectedFailure
    def test_broken(self):
        self.assertEqual(1, 2)


class TestWithMock(unittest.TestCase):
    from unittest.mock import patch, MagicMock, call

    def test_mock(self):
        from unittest.mock import patch
        with patch("mymodule.requests.get") as mock_get:
            mock_get.return_value.status_code = 200
            mock_get.return_value.json.return_value = {"key": "val"}
            result = mymodule.fetch("url")
            mock_get.assert_called_once_with("url")

    def test_mock_side_effect(self):
        from unittest.mock import MagicMock
        mock = MagicMock(side_effect=[1, 2, ValueError("oops")])
        mock()  # 1
        mock()  # 2
        # mock()  # raises ValueError


if __name__ == "__main__":
    unittest.main(verbosity=2)
```

Run: `python -m unittest discover -s tests -p "test_*.py"`

---

### 7.2 doctest

```python
def add(a, b):
    """
    Add two numbers.

    >>> add(1, 2)
    3
    >>> add(-1, 1)
    0
    >>> add(0.1, 0.2)  # doctest: +ELLIPSIS
    0.3...
    """
    return a + b

if __name__ == "__main__":
    import doctest
    doctest.testmod(verbose=True)

# Run from CLI:
# python -m doctest module.py -v
```

---

## PART 8: PACKAGING AND MODULES

---

### 8.1 Import System

```python
import os                      # standard library
import os.path
from os.path import join, exists
from os import getcwd as cwd

import mypackage
from mypackage import mymodule
from mypackage.subpkg import thing
from . import sibling           # relative (inside package)
from .. import parent_sibling

# Dynamic import
mod = __import__("os")
import importlib
mod = importlib.import_module("json")
importlib.reload(mod)           # reload changed module

# Conditional import
try:
    import ujson as json
except ImportError:
    import json

# __all__ controls what "from module import *" exports
__all__ = ["PublicClass", "public_function"]
```

---

### 8.2 Package Structure

```
myproject/
├── pyproject.toml
├── README.md
├── src/
│   └── mypackage/
│       ├── __init__.py
│       ├── core.py
│       ├── utils.py
│       └── subpkg/
│           ├── __init__.py
│           └── feature.py
└── tests/
    ├── __init__.py
    ├── test_core.py
    └── test_utils.py
```

**pyproject.toml**

```toml
[build-system]
requires = ["setuptools>=61", "wheel"]
build-backend = "setuptools.backends.legacy:build"

[project]
name = "mypackage"
version = "0.1.0"
description = "Description"
requires-python = ">=3.10"
dependencies = [
    "requests>=2.28",
]

[project.optional-dependencies]
dev = ["pytest", "mypy", "ruff"]

[project.scripts]
my-tool = "mypackage.cli:main"

[tool.setuptools.packages.find]
where = ["src"]
```

---

### 8.3 Virtual Environments

```bash
python3 -m venv .venv
source .venv/bin/activate        # Linux/macOS
.venv\Scripts\activate           # Windows
deactivate

pip install package
pip install -e .                 # editable install
pip install -r requirements.txt
pip freeze > requirements.txt
pip list
pip show package
pip uninstall package
```

---

## PART 9: PERFORMANCE AND PROFILING

---

### 9.1 timeit

```python
import timeit

timeit.timeit("sum(range(100))", number=10_000)
timeit.timeit("[x**2 for x in range(100)]", number=10_000)

# From CLI:
# python -m timeit "sum(range(100))"
# python -m timeit -n 10000 -r 5 "[x**2 for x in range(100)]"

# Inline benchmark
setup = "data = list(range(10000))"
stmt = "sorted(data)"
timeit.timeit(stmt, setup=setup, number=1000)
```

---

### 9.2 cProfile

```python
import cProfile
import pstats

# Profile a function
cProfile.run("my_function()", "profile_output")

prof = pstats.Stats("profile_output")
prof.sort_stats("cumulative")
prof.print_stats(20)    # top 20 functions

# Context manager
with cProfile.Profile() as pr:
    my_function()
ps = pstats.Stats(pr)
ps.sort_stats("time")
ps.print_stats()

# CLI:
# python -m cProfile -o out.prof script.py
# python -m pstats out.prof
```

---

### 9.3 memory_profiler and tracemalloc

```python
# tracemalloc — stdlib, no install needed
import tracemalloc

tracemalloc.start()
# ... code
snapshot = tracemalloc.take_snapshot()
stats = snapshot.statistics("lineno")
for stat in stats[:10]:
    print(stat)

tracemalloc.get_traced_memory()   # (current, peak) in bytes
tracemalloc.stop()
```

---

### 9.4 Common Optimizations

- Use `locals()` lookup for hot loops: assign global/attribute to local variable
- Generator expressions over list comprehensions when iterating once
- `str.join()` over `+=` for string concatenation in loops
- `collections.deque` over `list` for queue operations
- `dict` and `set` for O(1) membership testing (not `list`)
- `bisect` for sorted-list binary search
- `array.array` for typed arrays (more compact than list for numerics)
- `bytearray` for mutable binary data
- `__slots__` for memory-dense object classes

```python
import bisect
import array

# Binary search
a = [1, 3, 5, 7, 9]
bisect.bisect_left(a, 5)    # 2
bisect.insort(a, 4)          # inserts in sorted position

# Typed array
a = array.array("d", [1.0, 2.0, 3.0])   # 'd' = double
a.append(4.0)
a.tolist()
a.tobytes()
a.frombytes(b"\x00" * 8)
```

---

## PART 10: MISCELLANEOUS ESSENTIALS

---

### 10.1 Walrus Operator (3.8+)

```python
# Assign and use in same expression
while chunk := f.read(8192):
    process(chunk)

if m := re.search(r"\d+", s):
    print(m.group())

filtered = [y for x in data if (y := transform(x)) > 0]
```

---

### 10.2 Structural Pattern Matching (3.10+)

```python
match event:
    case {"type": "click", "button": btn}:
        print(f"clicked {btn}")
    case {"type": "keypress", "key": str(k)} if k.isprintable():
        print(f"pressed {k}")
    case [x, y]:
        print(f"sequence of two: {x}, {y}")
    case [x, *rest]:
        print(f"starts with {x}")
    case Point(x=0, y=0):
        print("origin")
    case Point(x=x, y=y):
        print(f"point at {x}, {y}")
    case None:
        print("nothing")
    case _:
        print("no match")
```

---

### 10.3 Exception Groups (3.11+)

```python
try:
    raise ExceptionGroup("errors", [ValueError("a"), TypeError("b")])
except* ValueError as eg:
    print("value errors:", eg.exceptions)
except* TypeError as eg:
    print("type errors:", eg.exceptions)
```

---

### 10.4 Useful Built-ins

```python
# Numeric
abs(-5)
round(3.14159, 2)
pow(2, 10)
pow(2, 10, 1000)   # modular exponentiation: (2**10) % 1000
divmod(17, 5)      # (3, 2)
min([3,1,2])
max([3,1,2])
sum([1,2,3], start=10)
all([True, True, False])   # False
any([False, False, True])  # True

# Sequences
len(obj)
sorted(iterable, key=None, reverse=False)
reversed(sequence)
enumerate(iterable, start=0)
zip(*iterables, strict=False)  # strict=True raises if lengths differ
map(func, iterable)
filter(func, iterable)
list(range(10))

# Objects
repr(obj)
str(obj)
vars(obj)          # obj.__dict__
dir(obj)           # list of attributes
hasattr(obj, "x")
getattr(obj, "x", default)
setattr(obj, "x", value)
delattr(obj, "x")
callable(obj)
isinstance(obj, type_or_tuple)
issubclass(cls, base_or_tuple)
type(obj)
id(obj)
hash(obj)

# Misc
print(*objects, sep=" ", end="\n", file=sys.stdout, flush=False)
input("prompt: ")
open(file, mode="r", encoding=None, errors=None)
eval("1 + 1")       # evaluate expression string
exec("x = 1")       # execute statement string (avoid)
compile(source, filename, mode)

# Iteration helpers
next(iterator)
next(iterator, default)
iter(iterable)
iter(callable, sentinel)   # calls callable until it returns sentinel

# Type conversion
int("42")
int("ff", 16)
float("3.14")
bool(x)
str(x)
list(x)
tuple(x)
set(x)
frozenset(x)
dict(pairs)
bytes(n)
bytes(iterable)
bytes(str, encoding)
bytearray(n)
memoryview(buffer)
bin(255)       # "0b11111111"
oct(8)         # "0o10"
hex(255)       # "0xff"
chr(65)        # "A"
ord("A")       # 65

# Format
format(1234567, ",")        # "1,234,567"
format(3.14159, ".2f")      # "3.14"
format(42, "#010x")         # "0x0000002a"
```

---

### 10.5 copy module

```python
import copy

# Shallow copy — copies container, not nested objects
lst2 = lst.copy()
lst2 = list(lst)
lst2 = lst[:]
d2 = d.copy()
d2 = dict(d)
copy.copy(obj)      # calls obj.__copy__() if defined

# Deep copy — recursively copies all nested objects
obj2 = copy.deepcopy(obj)   # calls obj.__deepcopy__ if defined

# Custom deep copy
class MyObj:
    def __deepcopy__(self, memo):
        cls = self.__class__
        result = cls.__new__(cls)
        memo[id(self)] = result
        for k, v in self.__dict__.items():
            setattr(result, k, copy.deepcopy(v, memo))
        return result
```

---

### 10.6 warnings

```python
import warnings

warnings.warn("deprecated function", DeprecationWarning, stacklevel=2)
warnings.warn("something fishy", UserWarning)

# Filter
warnings.filterwarnings("ignore", category=DeprecationWarning)
warnings.filterwarnings("error", category=RuntimeWarning)  # turn into exception

with warnings.catch_warnings():
    warnings.simplefilter("ignore")
    noisy_function()
```

---

### 10.7 pprint and textwrap

```python
import pprint

pprint.pprint(complex_data, width=80, depth=3)
pprint.pformat(complex_data)    # returns string

pp = pprint.PrettyPrinter(indent=4, width=60)
pp.pprint(data)

import textwrap

long_text = "This is a very long text that needs to be wrapped for display"
textwrap.fill(long_text, width=40)
textwrap.wrap(long_text, width=40)  # returns list of lines
textwrap.dedent("""
    indented code
    with consistent indentation
""")
textwrap.indent("text\nmore text", prefix="    ")
textwrap.shorten("long text to shorten", width=20, placeholder="...")
```

---

### 10.8 heapq

```python
import heapq

heap = []
heapq.heappush(heap, 3)
heapq.heappush(heap, 1)
heapq.heappush(heap, 2)
heapq.heappop(heap)       # 1 (minimum)
heap[0]                   # peek minimum without popping

heapq.heapify(lst)        # in-place, O(n)
heapq.nlargest(3, lst)
heapq.nsmallest(3, lst)

# Priority queue pattern
import heapq
pq = []
heapq.heappush(pq, (priority, item))
priority, item = heapq.heappop(pq)

# Max heap: negate priorities or values
heapq.heappush(pq, (-priority, item))
```

---

### 10.9 base64, binascii, zlib

```python
import base64

base64.b64encode(b"hello world")          # b'aGVsbG8gd29ybGQ='
base64.b64decode(b"aGVsbG8gd29ybGQ=")
base64.urlsafe_b64encode(b"data")
base64.b16encode(b"hi")                   # b'6869'

import zlib
compressed = zlib.compress(data, level=6)
decompressed = zlib.decompress(compressed)
zlib.crc32(data)
zlib.adler32(data)

import gzip
with gzip.open("file.gz", "wb") as f:
    f.write(data)
with gzip.open("file.gz", "rb") as f:
    data = f.read()
gzip.compress(b"data")
gzip.decompress(compressed)
```

---

### 10.10 String Formatting Reference

```python
# Format spec: [[fill]align][sign][z][#][0][width][grouping][.precision][type]
# align: < left, > right, ^ center, = pad between sign and digits
# sign: + always, - only negative, ' ' space for positive
# type: d int, f float, e scientific, g general, x hex, o octal, b binary,
#       s string, % percent, n locale-aware number

f"{42:08d}"           # 00000042
f"{3.14:.4f}"         # 3.1400
f"{3.14:10.4f}"       #     3.1400
f"{0.001:.2e}"        # 1.00e-03
f"{255:#x}"           # 0xff
f"{'hi':^10}"         #     hi
f"{'hi':*^10}"        # ****hi****
f"{1000000:,}"        # 1,000,000
f"{0.975:.1%}"        # 97.5%
```

---

This reference covers Python from first principles through the idioms, standard library modules, and patterns needed for real scripting and systems work. It is intentionally terse where topics are straightforward and detailed where behavior is non-obvious. The goal is to be the document you reach for before searching the internet.