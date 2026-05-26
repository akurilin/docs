# Alex's Python Reference

> A terse, code-first reference for the quirky Pythonic syntax of things you already know how to do in C++, Haskell, or TypeScript.

---

## Contents

**Types & data structures** — [Lists](#lists) · [Tuples](#tuples) · [Dictionaries](#dictionaries) · [Sets](#sets) · [Strings](#strings) · [Bytes & buffers](#bytes--binary-buffers) · [Numbers](#numbers) · [Dates & time](#dates--time) · [Copying / cloning](#copying--cloning) · [Time complexity (Big-O)](#time-complexity-big-o-of-common-ops)

**Language** — [Control flow](#control-flow-quirks) · [Functions](#functions) · [Comprehensions](#comprehensions) · [Iteration helpers](#iteration-helpers) · [Generators](#generators-yield) · [Enums](#enums) · [Classes](#classes) · [Dataclasses](#dataclasses) · [Exceptions](#exception-handling) · [Context managers](#context-managers-with) · [Type hints](#type-hints) · [Dependency injection](#dependency-injection-constructor-injection) · [Imports](#imports)

**Standard library** — [Pathlib](#pathlib-use-this-not-ospath) · [collections](#collections-module) · [heapq](#heapq-priority-queue) · [bisect](#bisect-binary-search-on-a-sorted-list) · [itertools](#itertools-highlights) · [functools](#functools-highlights)

**Concurrency** — [threading](#concurrency-threading) · [thread/process pools](#concurrency-threadprocess-pools) · [asyncio](#concurrency-asyncio)

**I/O & serialization** — [Streaming I/O](#streaming-io-files--sockets) · [JSON](#json) · [Pickling](#pickling-serialize-python-objects-to-disk) · [Gzip](#gzip-compression)

**Other** — [Misc Pythonisms](#misc-pythonisms-alex-keeps-forgetting)

---

## Lists
([⏱ complexity](#list))

Python lists are dynamic arrays (think `Vec<T>` / `ArrayList`), not linked lists.

```python
lst = [1, 2, 3]
lst.append(4)         # NOT push — adds one element at end
lst.extend([5, 6])    # append ALL elements of an iterable
lst += [7, 8]         # equivalent to extend
lst.insert(0, 0)      # insert at index (shifts the rest)
lst.pop()             # remove + return last
lst.pop(0)            # remove + return at index
lst.remove(3)         # remove FIRST occurrence by value (ValueError if missing)
lst.clear()           # empty in place
```

### Indexing & slicing

`lst[start:stop:step]` — stop is **exclusive**. All parts are optional.

```python
lst[-1]               # last element
lst[-2]               # second to last
lst[2:5]              # elements 2, 3, 4
lst[:3]               # first 3
lst[-3:]              # last 3        — ⚠️ negative start, empty stop (NOT lst[:-3])
lst[:-3]              # all BUT the last 3
lst[3:]               # from index 3 to end
lst[::2]              # every other element
lst[::-1]             # reversed (new list)
lst[:]                # shallow copy
lst[1:4] = [10, 20]   # slice assignment — replaces that range (can shrink/grow)
del lst[1:4]          # delete a range
```

### Sorting & reversing

```python
sorted(lst)                                # returns NEW list
sorted(lst, reverse=True)
sorted(lst, key=lambda x: x.name)          # sort by computed key
sorted(lst, key=lambda x: (x.age, x.name)) # tuple = multi-key sort

lst.sort()             # in place — returns None (!!), don't `x = lst.sort()`
lst.reverse()          # in place — also returns None

reversed(lst)          # returns ITERATOR, not a list
list(reversed(lst))    # if you want a list
```

### Other handy list ops

```python
lst.index(value)              # first index — ValueError if missing
lst.count(value)              # how many times value appears
len(lst)
value in lst                  # O(n) membership check
min(lst), max(lst), sum(lst)
any(lst), all(lst)            # truthiness over elements
```

---

## Tuples

Immutable, fixed-shape. Often used like records / product types.

```python
t = (1, 2, 3)
t = 1, 2, 3            # parens optional
single = (1,)          # trailing comma REQUIRED — (1) is just int 1
empty = ()

a, b, c = t            # unpack
a, *rest = t           # rest = [2, 3]  (always a list, even from a tuple)
a, _, c = t            # _ is the throwaway convention
```

---

## Dictionaries
([⏱ complexity](#dict-and-set))

Ordered (insertion order, guaranteed since 3.7). Hash map under the hood.

```python
d = {"a": 1, "b": 2}
d = dict(a=1, b=2)
d = dict([("a", 1), ("b", 2)])

# Lookup — the one Alex forgets the most
d["a"]                # KeyError if missing
d.get("a")            # None if missing
d.get("a", 0)         # default if missing
d.setdefault("a", 0)  # set + return if missing, else return existing

# Removal
d.pop("a")            # KeyError if missing
d.pop("a", None)      # default if missing
d.popitem()           # remove + return last (key, value) pair

# Membership / merging
"a" in d              # checks KEYS
d.update(other)       # merge in place
d | other             # NEW merged dict (3.9+)
d |= other            # in-place merge (3.9+)

# Views — these are LIVE (reflect changes to d)
d.keys(), d.values(), d.items()

# Iteration
for k in d: ...                     # iterates keys
for k, v in d.items(): ...

# Comprehensions
{k: v * 2 for k, v in d.items()}
{v: k for k, v in d.items()}        # invert
```

---

## Sets
([⏱ complexity](#dict-and-set))

```python
s = {1, 2, 3}
s = set()             # ⚠️ {} is an empty DICT, not an empty set

s.add(1)              # NOT append/push
s.update([4, 5, 6])   # add many

s.remove(1)           # KeyError if missing
s.discard(1)          # no error if missing
s.pop()               # remove arbitrary element

# Set algebra
a | b                 # union
a & b                 # intersection
a - b                 # difference
a ^ b                 # symmetric difference (in one, not both)
a <= b                # subset
a < b                 # proper subset

frozenset([1, 2])     # immutable, hashable — usable as dict key / set element
```

---

## Copying / cloning

```python
# Assignment NEVER copies — both names point to the SAME object
a = [1, 2, 3]
b = a
b.append(4)                  # a is now [1, 2, 3, 4] too

# Shallow copy — new outer container, SAME inner references
b = a.copy()                 # works on list / dict / set
b = a[:]                     # list slice — equivalent for lists
b = list(a)                  # constructor copy (works for any iterable)
b = {**d}                    # dict via unpacking
b = dict(d)
b = set(s)

# ⚠️ Shallow doesn't help with nested structures — inner objects are shared
grid = [[0, 0, 0], [0, 0, 0]]
clone = grid.copy()
clone[0][0] = 9              # grid[0][0] is ALSO 9 — inner lists shared
clone.append([7, 7, 7])      # fine — outer list IS independent

# Deep copy — recursively clones everything
import copy
b = copy.deepcopy(a)         # safe for nested structures (slower; handles cycles)
b = copy.copy(a)             # explicit shallow copy — same as a.copy()

# ⚠️ Classic trap: list multiplication creates SHARED references
matrix = [[0] * 3] * 3       # WRONG — outer list holds 3 refs to the SAME inner list
matrix[0][0] = 9             # → all three rows now start with 9

matrix = [[0] * 3 for _ in range(3)]   # RIGHT — each row is a fresh list

# Dataclasses: "copy with changes" (Haskell record-update vibes)
from dataclasses import replace
p2 = replace(p, x=10)        # new Point with x=10, other fields unchanged

# Tuples, frozensets, strings, numbers are immutable — no copy needed.
# copy.copy() on them just returns the same object.
```

---

## Strings
([⏱ complexity](#str))

```python
name, n = "Alex", 3
f"Hello {name}, count={n}"           # f-strings
f"{n:04d}"                           # zero-pad width 4 → "0003"
f"{3.14159:.2f}"                     # 2 decimals → "3.14"
f"{n:>10}"                           # right-align in width 10
f"{name!r}"                          # use repr() in f-string
f"{n=}"                              # → "n=3"  — debug helper (3.8+)

# Splitting / joining — note: join is a method on the SEPARATOR
",".join(["a", "b", "c"])            # "a,b,c"
s.split(",")                         # default: split on whitespace runs
s.splitlines()                       # by line, handles \r\n etc.

s.strip(), s.lstrip(), s.rstrip()    # whitespace by default; pass chars to strip those
s.replace("a", "b")                  # ALL occurrences
s.startswith("pre"), s.endswith(".py")
s.startswith(("a", "b"))             # tuple → any of these
"abc" in s                           # substring check

r"raw\nstring"                       # backslash-literal; no escapes
b"bytes"                             # bytes literal — different type! (see Bytes & buffers)
"""triple
quoted"""                            # multiline, keeps newlines
```

---

## Bytes & binary buffers

`bytes` is to binary what `str` is to text — and they DON'T mix (no implicit conversion).

```python
# bytes — IMMUTABLE sequence of ints 0–255
b = b"hi"                            # literal
bytes([104, 105])                    # from ints → b"hi"
b[0]                                 # ⚠️ an INT (104), NOT b"h"
b[0:1]                               # but a SLICE is bytes → b"h"
b.hex(), bytes.fromhex("6869")       # to/from hex string

# str <-> bytes — always an EXPLICIT encoding (utf-8 by default)
"café".encode()                      # str → bytes → b"caf\xc3\xa9"
b"caf\xc3\xa9".decode()              # bytes → str → "café"
# ⚠️ "abc" == b"abc" is False, and you can't concatenate str + bytes

# bytearray — MUTABLE bytes; build/patch buffers in place (used in the socket examples)
ba = bytearray(b"abc")
ba[0] = 65                           # in-place edit → bytearray(b"Abc")
ba.append(33); ba.extend(b"!!")      # grow like a list of bytes
bytes(ba)                            # freeze back to immutable bytes

# memoryview — zero-copy WINDOW into a bytes-like buffer; slice without copying
mv = memoryview(big_buffer)
chunk = mv[1024:2048]                # no copy — just a view (cheap on huge data)
mv[0:2] = b"\x00\x00"                # writes THROUGH to the underlying bytearray

# io.BytesIO — in-memory binary file object (file-like, but no disk)
import io
buf = io.BytesIO()
buf.write(b"data")                   # pass anywhere a file handle is expected
buf.getvalue()                       # → all bytes written so far
buf.seek(0); buf.read()              # rewind + read like a real file
# io.StringIO is the TEXT analog (in-memory text file)

# Common use: serialize to memory instead of disk
import pickle
buf = io.BytesIO()
pickle.dump(obj, buf)                # the "file" is really RAM
```

---

## Numbers

```python
1 / 2          # 0.5 — float division ALWAYS
1 // 2         # 0 — floor division (works on floats too: 3.7 // 2 == 1.0)
2 ** 10        # 1024 — power (NOT ^)
1 ^ 2          # XOR (bitwise) — ^ is NOT power in Python
10 % 3         # modulo
divmod(10, 3)  # (3, 1) — quotient and remainder together

# int has arbitrary precision — no overflow
10 ** 100      # works fine

# Conversion
int("42"), float("3.14"), str(42)
int("ff", 16), int("1010", 2)        # bases
hex(255), bin(10), oct(8)            # → "0xff", "0b1010", "0o10"

# Math
import math
math.floor, math.ceil, math.sqrt, math.log, math.log2, math.log10
math.inf, -math.inf, math.nan, math.isnan(x), math.isfinite(x)

# Exact decimal math (money etc.)
from decimal import Decimal
Decimal("0.1") + Decimal("0.2")      # exactly Decimal("0.3")

# Rationals
from fractions import Fraction
Fraction(1, 3) + Fraction(1, 6)      # Fraction(1, 2)
```

---

## Dates & time

```python
from datetime import datetime, date, time, timedelta, timezone

# Getting "now"
datetime.now()                       # ⚠️ NAIVE local time (no tz attached)
datetime.now(timezone.utc)           # aware UTC — prefer this for anything stored/compared
date.today()                         # just the date

# ⚠️ datetime.utcnow() is deprecated (3.12) AND returns a naive value — don't use it.

# Aware vs naive: a datetime with .tzinfo set is "aware", without is "naive".
# ⚠️ You CANNOT compare or subtract a naive and an aware datetime — TypeError.
dt.replace(tzinfo=timezone.utc)      # attach a tz to a naive dt (doesn't convert!)
aware.astimezone(timezone.utc)       # CONVERT an aware dt to another tz

# Components
dt.year, dt.month, dt.day, dt.hour, dt.minute, dt.second, dt.weekday()  # Mon=0

# Comparing & differencing — operators just work on like-kinded datetimes
dt1 < dt2                            # chronological order
delta = dt2 - dt1                    # → timedelta
delta.days, delta.total_seconds()    # ⚠️ .seconds is the leftover <1day part, NOT the total

# timedelta — arithmetic on durations
from datetime import timedelta
dt + timedelta(days=7, hours=3)
timedelta(weeks=2) > timedelta(days=10)

# Parse / format
datetime.fromisoformat("2026-05-25T14:30:00+00:00")   # ISO string → datetime
dt.isoformat()                                         # → "2026-05-25T14:30:00+00:00"
datetime.strptime("25/05/2026", "%d/%m/%Y")            # parse custom format
dt.strftime("%Y-%m-%d %H:%M")                          # format custom
# codes: %Y year %m month %d day %H hour(24) %M min %S sec %z offset %A weekday

# Unix timestamps (seconds since epoch, UTC)
dt.timestamp()                                  # aware dt → float seconds
datetime.fromtimestamp(ts, tz=timezone.utc)     # → aware dt

# ⚠️ Measuring ELAPSED time: use monotonic, NOT datetime/time.time().
# Wall clocks can jump (NTP, DST) and go backwards. monotonic never does.
import time
t0 = time.monotonic()
do_work()
elapsed = time.monotonic() - t0      # seconds (float)
time.perf_counter()                  # highest-resolution monotonic clock for benchmarks
time.sleep(1.5)                      # block this thread for N seconds
```

---

## Control flow quirks

```python
# No C-style for. Always for-each.
for i in range(10): ...              # 0..9
for i in range(2, 10): ...           # 2..9
for i in range(0, 10, 2): ...        # 0, 2, 4, 6, 8
for i in range(10, 0, -1): ...       # 10..1 — stop is exclusive

# Ternary
value = "yes" if cond else "no"

# Truthiness — these are FALSY:
# None, False, 0, 0.0, "", [], {}, set(), ()
# Everything else is truthy.

# for/while else — runs only if loop completes WITHOUT break
for x in items:
    if x == target:
        break
else:
    print("not found")               # only runs if no break

# Walrus operator — assign-and-use
if (n := len(data)) > 10:
    print(f"too long: {n}")

while (chunk := f.read(8192)):
    process(chunk)

# Match statement (3.10+) — structural pattern matching, not just switch
match point:
    case (0, 0):                print("origin")
    case (x, 0):                print(f"on x-axis at {x}")
    case (x, y) if x == y:      print("on diagonal")           # guard
    case Point(x=0, y=y):       print(f"on y-axis at {y}")     # class pattern
    case [first, *rest]:        print(f"list starting {first}")
    case {"name": name}:        print(f"dict with name={name}")
    case _:                     print("other")                  # default
```

---

## Functions

```python
def f(
    x,              # required; positional or keyword
    y=10,           # optional (default 10); positional or keyword
    *args,          # collects EXTRA positional args into a tuple
    key=None,       # keyword-only (anything after *args is keyword-only)
    **kwargs,       # collects EXTRA keyword args into a dict
):
    ...

# Different ways to call it — note how the args partition
f(1)                              # x=1
f(1, 2)                           # x=1, y=2
f(x=1, y=2)                       # same, by name
f(1, 2, 3, 4, 5)                  # x=1, y=2, args=(3, 4, 5)
f(1, 2, 3, 4, key="z")            # x=1, y=2, args=(3, 4), key="z"
f(1, y=2, key="z", extra="hi")    # x=1, y=2, args=(), key="z", kwargs={"extra": "hi"}

# Positional-only (before /), keyword-only (after *)
def g(pos_only, /, normal, *, kw_only):
    ...
g(1, 2, kw_only=3)        # ✅ pos by position, normal either way, kw_only by name
g(1, normal=2, kw_only=3) # ✅ normal can go either way
g(pos_only=1, ...)        # ❌ pos_only can't be passed by name
g(1, 2, 3)                # ❌ kw_only must be passed by name

# Type hints (NOT enforced at runtime — purely for tooling)
def greet(name: str, count: int = 1) -> str:
    return f"hi {name}" * count

# ⚠️ Mutable default trap — default is evaluated ONCE at def time
def bad(items=[]):           # DON'T — list is shared across calls
    items.append(1)
    return items

def good(items=None):        # DO this instead
    if items is None:
        items = []
    ...

# Lambda — single expression only, no statements
add = lambda a, b: a + b

# Unpacking in calls
f(*args, **kwargs)
f(1, 2, *more, key="x", **extras)
```

---

## Comprehensions

```python
[x*2 for x in lst]                            # list
[x*2 for x in lst if x > 0]                   # with filter
[x*y for x in xs for y in ys]                 # nested (xs is outer)
[[x*y for x in xs] for y in ys]               # 2D

{x for x in lst}                              # set
{k: v for k, v in pairs}                      # dict
(x*2 for x in lst)                            # generator expr — LAZY, not a tuple!

# Ternary INSIDE the expression (different position than filter)
[x if x > 0 else 0 for x in lst]              # transform
[x for x in lst if x > 0]                     # filter
```

---

## Iteration helpers

```python
for i, x in enumerate(lst): ...
for i, x in enumerate(lst, start=1): ...

for a, b in zip(xs, ys): ...                  # stops at SHORTEST
zip(xs, ys, strict=True)                      # 3.10+ — raises if lengths differ

# Transpose / unzip
nums, letters = zip(*[(1, "a"), (2, "b")])
matrix_T = list(zip(*matrix))
```

---

## Generators (yield)

A function containing `yield` becomes a generator: calling it returns a lazy iterator instead of running the body.

```python
def count_up_to(n):
    i = 0
    while i < n:
        yield i             # pauses here; resumes on next next() call
        i += 1

g = count_up_to(3)          # ⚠️ NOTHING has run yet — just returns a generator object
next(g)                     # 0   — runs until first yield
next(g)                     # 1
list(count_up_to(3))        # [0, 1, 2]
for x in count_up_to(3):    # the usual way to consume
    process(x)
```

Why bother: O(1) memory for arbitrarily long sequences, infinite streams, and clean pipelines.

```python
# Infinite stream — safe because consumers can stop early
def naturals():
    n = 0
    while True:
        yield n
        n += 1

from itertools import islice
list(islice(naturals(), 5))     # [0, 1, 2, 3, 4]

# Pipeline of generators — each stage is lazy, processes one item at a time
def read_lines(path):
    with open(path) as f:
        for line in f:
            yield line.rstrip()

def non_empty(lines):
    for line in lines:
        if line:
            yield line

for line in non_empty(read_lines("data.txt")):
    process(line)
```

### yield from — delegate to another iterable

```python
def chain(*iterables):
    for it in iterables:
        yield from it       # equivalent to: for x in it: yield x

def flatten(nested):
    for item in nested:
        if isinstance(item, list):
            yield from flatten(item)    # recursion via yield from
        else:
            yield item

list(flatten([1, [2, [3, 4], 5], 6]))   # [1, 2, 3, 4, 5, 6]
```

(Generator expressions like `(x*2 for x in lst)` are covered in [Comprehensions](#comprehensions) — same laziness, inline syntax.)

---

## Enums

```python
from enum import Enum, auto, IntEnum, StrEnum   # StrEnum is 3.11+

class Color(Enum):
    RED = auto()
    GREEN = auto()
    BLUE = auto()

Color.RED                # <Color.RED: 1>
Color.RED.name           # "RED"
Color.RED.value          # 1
Color(1)                 # lookup by VALUE → Color.RED
Color["RED"]             # lookup by NAME → Color.RED

for c in Color: ...      # iterate members

# StrEnum — members ARE strings, useful for APIs/serialization
class Status(StrEnum):
    OK = "ok"
    FAIL = "fail"
# Status.OK == "ok"  → True
# json.dumps({"s": Status.OK}) works directly
```

---

## Classes

```python
class Dog:
    species = "Canis familiaris"     # CLASS variable — shared by ALL instances

    def __init__(self, name, age):   # constructor; self = the instance being built
        self.name = name             # INSTANCE variables — unique per object
        self.age = age

    def bark(self):                  # instance method — self is passed automatically
        return f"{self.name} says woof"

d = Dog("Rex", 3)                    # calls __init__ (you never call it directly)
d.bark()                             # → "Rex says woof"  (d passed as self)
d.name                               # "Rex"
Dog.species                          # access class var via the class

# ⚠️ Mutable CLASS variable is shared across instances — classic trap
class Bad:
    tricks = []                      # ONE list for the whole class
    def add(self, t): self.tricks.append(t)
# fix: assign mutable state to self in __init__ instead (self.tricks = [])

# Method types
class C:
    def instance_method(self): ...       # operates on the instance (self)

    @classmethod
    def from_string(cls, s):             # gets the CLASS (cls), not an instance
        return cls(...)                  # common for "alternative constructors"

    @staticmethod
    def helper(x):                       # no self/cls — just a function namespaced on C
        return x * 2

# @property — computed attribute that LOOKS like a plain field (no parens to call)
class Circle:
    def __init__(self, r): self.r = r
    @property
    def area(self):                      # access as c.area, not c.area()
        return 3.14159 * self.r ** 2
    @area.setter                         # optional — makes c.area = ... work
    def area(self, value): ...

# __repr__ / __str__ — control how the object prints
class Point:
    def __init__(self, x, y): self.x, self.y = x, y
    def __repr__(self):                  # for devs/debug; aim to be unambiguous
        return f"Point({self.x}, {self.y})"
    def __str__(self):                   # for users/print(); falls back to __repr__

# Naming conventions (Python has no real "private" — enforced by convention)
self._internal                          # single _ : "internal, don't touch" (just a hint)
self.__mangled                          # double _ : name-mangled to _Class__mangled
                                        #            (avoids subclass clashes, NOT security)

# Inheritance + super()
class Animal:
    def __init__(self, name): self.name = name
    def speak(self): return "..."

class Cat(Animal):
    def __init__(self, name, indoor):
        super().__init__(name)           # call parent __init__ — don't skip this
        self.indoor = indoor
    def speak(self): return "meow"       # override

isinstance(c, Animal)                    # True — Cat IS-A Animal
issubclass(Cat, Animal)                  # True
```

---

## Dataclasses

```python
from dataclasses import dataclass, field

@dataclass
class Point:
    x: float
    y: float
    label: str = "origin"
    tags: list[str] = field(default_factory=list)   # mutable default — MUST use field

p = Point(1.0, 2.0)
p == Point(1.0, 2.0, "origin", [])   # auto __eq__, __repr__, __init__

@dataclass(frozen=True)              # immutable + hashable (usable in sets/dict keys)
class Vec:
    x: float
    y: float

@dataclass(slots=True)               # 3.10+ — less memory, faster attribute access
class Compact:
    x: int

@dataclass(order=True)               # adds <, <=, >, >= (lexicographic on fields)
class Score:
    points: int
    name: str

# Computed / derived fields
@dataclass
class Box:
    w: int
    h: int
    def __post_init__(self):
        self.area = self.w * self.h
```

---

## Exception handling

```python
try:
    risky()
except (ValueError, TypeError) as e:        # tuple = catch any of these
    handle(e)
except Exception as e:
    log(e)
    raise                                    # re-raise the current exception
except Exception as e:
    raise RuntimeError("wrap") from e        # chained — preserves __cause__
else:
    # runs only if NO exception was raised
    ...
finally:
    cleanup()                                # always runs

# Common built-ins
# ValueError, TypeError, KeyError, IndexError, AttributeError,
# RuntimeError, NotImplementedError, StopIteration, FileNotFoundError, OSError

# Custom
class MyError(Exception):
    pass

# ExceptionGroup + except* (3.11+) — for handling multiple errors at once
try:
    ...
except* ValueError as eg:                    # handle just the ValueErrors
    for e in eg.exceptions: ...
```

---

## Context managers (with)

```python
with open("file.txt") as f:
    data = f.read()
# file auto-closed here

# Multiple
with open("a") as a, open("b") as b:
    ...

# Custom — class-based
class Timer:
    def __enter__(self):
        self.t0 = time.time()
        return self
    def __exit__(self, exc_type, exc, tb):
        print(time.time() - self.t0)

# Custom — generator-based (simpler)
from contextlib import contextmanager

@contextmanager
def timer():
    t0 = time.time()
    try:
        yield
    finally:
        print(time.time() - t0)

with timer():
    do_work()

# Useful contextlib helpers
from contextlib import suppress, redirect_stdout, nullcontext
with suppress(FileNotFoundError):
    os.remove("maybe.txt")
```

---

## Pathlib (use this, not os.path)

```python
from pathlib import Path

p = Path("data") / "file.txt"        # / operator builds paths
p.exists(), p.is_file(), p.is_dir()
p.read_text(), p.write_text("...")
p.read_bytes(), p.write_bytes(b"...")

p.name           # "file.txt"
p.stem           # "file"
p.suffix         # ".txt"
p.parent         # Path("data")
p.parts          # ("data", "file.txt")
p.absolute(), p.resolve()
p.with_suffix(".bak")
p.with_name("other.txt")

p.iterdir()                          # immediate children
p.glob("*.py")                       # pattern
p.rglob("*.py")                      # recursive

p.mkdir(parents=True, exist_ok=True)
p.unlink(missing_ok=True)            # remove file
p.rename(new)

Path.home(), Path.cwd()
```

---

## collections module
([⏱ complexity: deque](#deque))

```python
from collections import defaultdict, Counter, deque, namedtuple, ChainMap

# defaultdict — auto-creates missing values via the factory
counts = defaultdict(int)
counts["x"] += 1                     # no KeyError, starts at 0

groups = defaultdict(list)
groups["a"].append(1)                # no "if 'a' not in groups" dance

# Counter — multiset
c = Counter("abracadabra")           # {'a': 5, 'b': 2, 'r': 2, 'c': 1, 'd': 1}
c.most_common(2)                     # [('a', 5), ('b', 2)]

# Increment / decrement a single key — missing keys read as 0 (no KeyError)
c["x"] += 1                          # increment; the += is what inserts the key
c["x"] -= 1                          # decrement; counts CAN go to 0 or negative
c.update(["x", "y", "x"])            # +1 per occurrence
c.update({"x": 5})                   # ⚠️ ADDS 5 (NOT dict.update, which REPLACES)
c.subtract({"x": 3})                 # subtract counts (can go negative)

c1 + c2                              # adds counts
c1 - c2                              # subtract, clamped at 0
c1 & c2                              # min, c1 | c2  → max

# deque — O(1) at BOTH ends (lists are O(n) at the left)
# ⚠️ A fast CONTAINER, not a thread channel. To pass work between threads with
#    blocking/backpressure, use queue.Queue (see Concurrency: threading), not deque.
d = deque([1, 2, 3])
d.append(4); d.appendleft(0)
d.pop(); d.popleft()
d.extend([5, 6])
d.extendleft([2, 1])                 # ⚠️ extends in REVERSE order: result is [1, 2, ...rest]
d.rotate(1)                          # rightward: last element → front
d.rotate(-1)                         # leftward: first element → back
d = deque(maxlen=100)                # bounded — drops oldest when full

# OrderedDict
# ⚠️ Regular dict preserves insertion order since 3.7, so plain dict covers most needs.
# OrderedDict is still useful for:
from collections import OrderedDict

od = OrderedDict([("a", 1), ("b", 2), ("c", 3)])
od.move_to_end("a")                  # move to END
od.move_to_end("a", last=False)      # move to FRONT
od.popitem()                         # last (LIFO) — same as dict
od.popitem(last=False)               # first (FIFO) — plain dict can't do this

# Equality is order-sensitive (dict's isn't)
OrderedDict([("a",1),("b",2)]) == OrderedDict([("b",2),("a",1)])   # False
{"a":1, "b":2} == {"b":2, "a":1}                                    # True

# Classic use: simple LRU cache
cache = OrderedDict()
def get(key):
    if key in cache:
        cache.move_to_end(key)       # mark as recently used
        return cache[key]
    ...

# ChainMap — search multiple dicts in order (good for configs/scopes)
cfg = ChainMap(overrides, defaults)
```

---

## heapq (priority queue)
([⏱ complexity](#heapq))

It's a **min-heap on a regular list**. There's no separate Heap class — `heapq` is a set of functions that operate on lists.

```python
import heapq

h = []
heapq.heappush(h, 3)
heapq.heappush(h, 1)
heapq.heappush(h, 2)
heapq.heappop(h)                     # 1 — always returns SMALLEST
h[0]                                 # peek at smallest without popping

# ⚠️ A heap is NOT a sorted list. Only h[0] (the min) is meaningful — h[1], h[2]…
# are in NO guaranteed order. You can't binary-search it or index "the 2nd smallest".
# Need a fully-ordered list with random access / search? That's bisect, not heapq.

# Heapify an existing list in place — O(n)
nums = [5, 1, 3, 8, 2]
heapq.heapify(nums)                  # nums is now a valid heap

# Combined ops — more efficient than separate push + pop
heapq.heappushpop(h, x)              # push then pop (smaller of new and existing min)
heapq.heapreplace(h, x)              # pop then push (assumes h non-empty)

# Top-N without sorting the whole list — O(n log k)
heapq.nlargest(3, nums)
heapq.nsmallest(3, nums)
heapq.nlargest(3, items, key=lambda x: x.score)

# ⚠️ No built-in max-heap. Workaround: negate the values.
heapq.heappush(h, -x)
biggest = -heapq.heappop(h)

# Priority queue with tuples: (priority, item)
# Tuples compare LEXICOGRAPHICALLY — item by item, left to right: (1, "z") < (2, "a")
# because 1 < 2 (the rest is never even looked at). That's WHY priority goes FIRST.
heapq.heappush(pq, (3, "task A"))
heapq.heappush(pq, (1, "task B"))
heapq.heappop(pq)                    # (1, "task B") — smallest first element wins

# ⚠️ If priorities can tie AND items aren't comparable (e.g. dicts, custom objects),
# add a tiebreaker counter so Python never tries to compare the items themselves:
import itertools
counter = itertools.count()
heapq.heappush(pq, (priority, next(counter), item))
```

---

## bisect (binary search on a sorted list)

Like `heapq`, it's functions over a plain list — but the list must be **fully sorted**, and it's for *searching*, not for repeatedly popping the min (that's `heapq`). ([complexity table ↓](#time-complexity-big-o-of-common-ops))

```python
import bisect

a = [1, 3, 4, 4, 7]                  # ⚠️ MUST be sorted ascending

# Insertion point — where x WOULD go to keep a sorted. The two variants differ on TIES:
bisect.bisect_left(a, 4)             # 2 — leftmost spot (before existing 4s)
bisect.bisect_right(a, 4)            # 4 — rightmost spot (after existing 4s); alias: bisect()

# Insert while staying sorted — ⚠️ search is O(log n) but the shift makes it O(n) overall
bisect.insort(a, 5)                  # a → [1, 3, 4, 4, 5, 7]

# Binary search "does x exist": bisect_left, then check the slot
i = bisect.bisect_left(a, 4)
found = i < len(a) and a[i] == 4

# Count occurrences of x in O(log n)
bisect.bisect_right(a, 4) - bisect.bisect_left(a, 4)   # 2

# Range query: how many elements in [lo, hi)
bisect.bisect_left(a, hi) - bisect.bisect_left(a, lo)

# Classic: map a value into a bucket (grades, tiers)
grades = "FDCBA"
breaks = [60, 70, 80, 90]
grades[bisect.bisect(breaks, score)]      # score=85 → "B"

# key= (3.10+) — search by a computed field, no separate key-list needed
bisect.bisect_left(rows, target_id, key=lambda r: r.id)
```

---

## Time complexity (Big-O of common ops)

CPython, average case. The ones interviewers probe are flagged ⚠️.

### list
Dynamic array.

| Operation | Cost | |
|---|---|---|
| `lst[i]` get/set | **O(1)** | direct index |
| `append(x)` / `pop()` | **O(1)** | amortized; at the END |
| `insert(0, x)` / `pop(0)` | **O(n)** ⚠️ | shifts everything — use `deque` for a queue |
| `insert(i, x)` / `del lst[i]` | **O(n)** | shift after `i` |
| `x in lst` | **O(n)** ⚠️ | linear scan — use a `set` for membership |
| `.index(x)` / `.count(x)` / `min` / `max` / `sum` | **O(n)** | full scan |
| `lst[a:b]` slice | **O(b−a)** | copies the range |
| `.sort()` | **O(n log n)** | |
| `len(lst)` | **O(1)** | stored, not counted |

### deque
O(1) at BOTH ends — that's the point.

| Operation | Cost | |
|---|---|---|
| `append` / `appendleft` / `pop` / `popleft` | **O(1)** | vs list's O(n) on the left |
| `d[i]` access in the middle | **O(n)** ⚠️ | not an array — no random access |

### dict and set
Hash-based.

| Operation | Cost | |
|---|---|---|
| get / set / del by key, `x in d`, `s.add` | **O(1)** | average; O(n) worst case (pathological hashes) |
| iterate | **O(n)** | |
| `a & b` set intersection | **O(min(len))** | |
| `a \| b` union / `a - b` difference | **O(len)** | |

### heapq
Binary heap on a list.

| Operation | Cost | |
|---|---|---|
| `heappush` / `heappop` | **O(log n)** | |
| `h[0]` peek smallest | **O(1)** | |
| `heapify(lst)` | **O(n)** ⚠️ | NOT O(n log n) — build is linear |
| `nlargest(k)` / `nsmallest(k)` | **O(n log k)** | beats sort-then-slice for small k |

### str

| Operation | Cost | |
|---|---|---|
| `s[i]`, `len(s)` | **O(1)** | |
| `+=` in a loop | **O(n²)** ⚠️ | strings are immutable — use `"".join(parts)` → O(n) |
| `x in s` substring | **O(n·m)** | worst case |

---

## itertools highlights

```python
import itertools as it

it.chain([1,2], [3,4])               # 1,2,3,4 — flatten one level
it.chain.from_iterable(list_of_lists)
it.product([1,2], [3,4])             # cartesian: (1,3),(1,4),(2,3),(2,4)
it.combinations([1,2,3], 2)          # (1,2),(1,3),(2,3)
it.permutations([1,2,3], 2)          # all 2-orderings
it.groupby(sorted_lst, key=fn)       # ⚠️ requires SORTED input
it.accumulate([1,2,3,4])             # 1,3,6,10 — running sum (or any binop)
it.accumulate([3,1,4,1,5], max)      # 3,3,4,4,5
it.takewhile(pred, iterable)         # take while true, stop at first false
it.dropwhile(pred, iterable)         # drop while true, then take rest
it.islice(iterable, 5, 10)           # slicing for ANY iterable
it.count(start=0, step=1)            # infinite counter
it.cycle(iterable)                   # infinite cycle
it.repeat(x, n)
it.pairwise([1,2,3,4])               # (1,2),(2,3),(3,4) — 3.10+
```

---

## functools highlights

```python
from functools import partial, reduce, cache, lru_cache, wraps

add5 = partial(lambda x, y: x + y, 5)   # pre-bind args

reduce(lambda a, b: a + b, [1, 2, 3], 0)  # 6 — fold-left

@cache                                   # unbounded memoization (3.9+)
def fib(n): return n if n < 2 else fib(n-1) + fib(n-2)

@lru_cache(maxsize=128)                  # bounded LRU memoization
def slow(x): ...

@wraps(original)                         # preserve name/doc in decorators
def wrapper(*args, **kwargs): ...
```

---

## Type hints

```python
from typing import Optional, Union, Any, Callable, Iterable, Iterator, Protocol, TypeVar

# Built-ins ARE generics (3.9+) — no need for List, Dict, Tuple
xs: list[int]
m: dict[str, list[int]]
t: tuple[int, str, float]            # fixed shape
t: tuple[int, ...]                   # variable length, all int

# Optional / union — 3.10+ pipe syntax
x: int | None
x: int | str
# older equivalent: Optional[int] / Union[int, str]

# Callable
fn: Callable[[int, str], bool]
fn: Callable[..., bool]              # any args

# Type aliases
type UserId = int                    # 3.12+
UserId = int                         # older

# Generics
T = TypeVar("T")
def first(xs: list[T]) -> T:
    return xs[0]

# 3.12+ generic syntax (cleaner)
def first[T](xs: list[T]) -> T:
    return xs[0]

# Structural typing
class HasName(Protocol):
    name: str
def greet(x: HasName) -> str:
    return f"hi {x.name}"
```

---

## Dependency injection (constructor injection)

No framework needed — it's just "pass dependencies IN via `__init__` instead of building them inside." Makes code testable: swap real deps for fakes without monkeypatching.

```python
# Duck typing: the service just needs SOMETHING with a .send(to, body) method.
# (In real code you'd declare a typing.Protocol for the dependency's shape — see
#  Type hints. For interview speed, skip the ceremony and rely on duck typing.)

class SignupService:
    def __init__(self, mailer):              # ← injected here, not constructed inside
        self.mailer = mailer                 # store it; the service doesn't pick the impl

    def register(self, email):
        self.mailer.send(email, "welcome")

# Production: wire the real deps once, at the entry point ("composition root")
class SmtpMailer:
    def send(self, to, body): ...            # real implementation
service = SignupService(SmtpMailer())

# Tests: inject a fake — no mock/patch needed, just any object with a .send()
class FakeMailer:
    def __init__(self): self.sent = []
    def send(self, to, body): self.sent.append((to, body))

fake = FakeMailer()
SignupService(fake).register("a@b.com")
assert fake.sent == [("a@b.com", "welcome")]

# ⚠️ Anti-pattern: building the dependency inside hardwires it & blocks testing
class Bad:
    def __init__(self):
        self.mailer = SmtpMailer()           # now you can't substitute it
```

---

## Concurrency: threading

```python
import threading, time

def worker(name, delay):
    time.sleep(delay)
    print(f"done {name}")

t = threading.Thread(target=worker, args=("a", 1))
t.start()
t.join()                             # block until finished

# Multiple
threads = [threading.Thread(target=worker, args=(n, 1)) for n in "abc"]
for t in threads: t.start()
for t in threads: t.join()

# Synchronization primitives — all support `with` (acquire/release) where it makes sense

# Lock — mutual exclusion; one holder at a time. The default choice.
lock = threading.Lock()
with lock:
    shared_state += 1
lock.acquire(timeout=5)              # non-with form; returns False if it times out

# RLock — reentrant: the SAME thread can acquire it multiple times without deadlocking
rlock = threading.RLock()            # use when a locked method calls another locked method

# Semaphore — allow up to N concurrent holders (e.g. cap to 10 in-flight requests)
sem = threading.Semaphore(10)
with sem: ...
# BoundedSemaphore — same, but raises if released more times than acquired (catches bugs)

# Event — one-shot/broadcast flag; any number of waiters wake when it's set
ready = threading.Event()
ready.wait()                         # block until set (optional timeout=)
ready.set(); ready.clear(); ready.is_set()

# Condition — wait for a predicate to become true; notify waiters when state changes
cond = threading.Condition()
with cond:
    cond.wait_for(lambda: queue_has_items)   # releases lock while waiting, reacquires on wake
with cond:
    cond.notify()                            # or notify_all() — wake waiter(s)

# Barrier — N threads block until all N have arrived, then all proceed together
barrier = threading.Barrier(3)
barrier.wait()                       # returns once the 3rd thread arrives

# queue.Queue — thread-safe producer/consumer channel (NOT the same as collections.deque,
# which is just a fast container with no blocking/backpressure). This is how threads hand
# work to each other. (asyncio.Queue is the async-task analog — see asyncio section.)
import queue
q = queue.Queue(maxsize=100)         # 0 = unbounded; maxsize gives backpressure
q.put(item)                          # blocks if full (timeout= optional)
item = q.get()                       # blocks until an item is available
q.task_done()                        # mark the just-gotten item as processed
q.join()                             # block until every put item has been task_done'd
# variants: queue.LifoQueue (stack), queue.PriorityQueue (min-heap of (priority, item))

# Daemon — dies with main program (won't keep process alive)
t = threading.Thread(target=worker, daemon=True)

# ⚠️ Threads CANNOT be cancelled from outside. threading.Thread has no .cancel().
# Workers must check a shared flag periodically (use the Event pattern above).
# (There's a ctypes hack to inject an exception — don't.)

# ⚠️ GIL: Python threads do NOT parallelize CPU-bound work.
# Good for I/O (file, network, sleep), bad for math/parsing/etc.
# For CPU-bound work, use multiprocessing or ProcessPoolExecutor.
```

---

## Concurrency: thread/process pools

```python
from concurrent.futures import ThreadPoolExecutor, ProcessPoolExecutor, as_completed

def fetch(url): ...

# Submit individual tasks → get Futures
with ThreadPoolExecutor(max_workers=8) as ex:
    fut = ex.submit(fetch, "http://example.com")
    result = fut.result()            # blocks until done; re-raises exception

    # Many tasks
    futures = [ex.submit(fetch, url) for url in urls]
    for f in as_completed(futures):  # yields as each completes (any order)
        print(f.result())

# Map — parallel; results in INPUT order, raises on first failure
with ThreadPoolExecutor(max_workers=8) as ex:
    for result in ex.map(fetch, urls):
        print(result)

# CPU-bound — same API, swap the class
with ProcessPoolExecutor() as ex:
    results = list(ex.map(heavy_calc, inputs))

# Future API
fut.done()
fut.cancel()                         # ⚠️ returns True only if NOT yet running.
                                     # Once started, returns False — workers can't
                                     # be interrupted (same constraint as threading).
fut.result(timeout=5)                # blocks; TimeoutError if not done
fut.exception()                      # returns the exception (doesn't raise)
fut.add_done_callback(cb)
```

⚠️ **Recursive submission gotcha** — exiting the `with` block calls `shutdown(wait=True)`, which waits for in-flight tasks BUT also rejects any new submissions. If worker tasks submit more work to the same executor, the main thread *must* block inside the `with` block — otherwise it falls off the end, shutdown begins, and grandchildren's `submit()` calls raise `RuntimeError: cannot schedule new futures after shutdown`.

```python
done = threading.Event()
with ThreadPoolExecutor() as ex:
    ex.submit(seed_task, ex, done)   # may itself submit more tasks to ex
    done.wait()                      # ⚠️ keeps the `with` block alive
# Without that wait(), main thread exits → shutdown → children blow up.
# TaskGroup doesn't have this footgun — see below.
```

---

## Concurrency: asyncio

```python
import asyncio

async def fetch(url):
    await asyncio.sleep(1)           # yields control — DON'T use time.sleep here
    return f"data from {url}"

# Top-level entry — creates and runs an event loop
asyncio.run(main())

# Inside async code:
result = await fetch(url)            # one at a time, sequential
```

### Running things concurrently

```python
# gather — start all, wait for all, return results in input order
results = await asyncio.gather(
    fetch("a"), fetch("b"), fetch("c"),
)

# return_exceptions=True → exceptions land in results instead of raising
results = await asyncio.gather(*coros, return_exceptions=True)

# Schedule without awaiting yet
task = asyncio.create_task(fetch("a"))
# ... do other work concurrently ...
result = await task
```

### TaskGroup (3.11+, preferred over gather for new code)

```python
async def main():
    async with asyncio.TaskGroup() as tg:
        t1 = tg.create_task(fetch("a"))
        t2 = tg.create_task(fetch("b"))
    # all tasks are done when the `async with` block exits
    # if ANY task raises, the others are cancelled, errors collected as ExceptionGroup
    print(t1.result(), t2.result())
```

Unlike `ThreadPoolExecutor`, a running task can freely call `tg.create_task(...)` to spawn more work into the same group. The `async with` block waits as long as *anything* is still running, so recursive/self-perpetuating task trees work without any explicit "all done" event.

### Cancellation

`task.cancel()` schedules a `CancelledError` to be raised inside the task at its **next `await` point** (not immediately — cancellation in asyncio is cooperative, but the cooperation happens at every await).

```python
task = asyncio.create_task(worker())
task.cancel()
try:
    await task                             # re-raises CancelledError
except asyncio.CancelledError:
    ...
```

⚠️ **`asyncio.CancelledError` inherits from `BaseException`, not `Exception` (since 3.8).** That's deliberate — it slips through `except Exception:` so cancellation propagates by default.

```python
async def worker():
    try:
        await something()
    except Exception:
        handle_error()                     # CancelledError NOT caught — good
    finally:
        cleanup()                          # finally STILL runs on cancellation

# If you DO need to catch it, re-raise (don't swallow cancellations)
async def worker():
    try:
        await something()
    except asyncio.CancelledError:
        await emergency_cleanup()
        raise                              # propagate it onward

# Protect a critical section from being cancelled mid-flight
await asyncio.shield(important_cleanup())
```

A couple more quirks worth knowing:

- In a `TaskGroup`, when one task fails the siblings are cancelled — but those `CancelledError`s don't show up in the resulting `ExceptionGroup`. Only the *original* failure(s) do.
- `wait_for` and `asyncio.timeout()` implement timeouts *by cancelling the inner coroutine*, then surfacing `TimeoutError` to the caller. So inside the cancelled coroutine, you'll see `CancelledError`, not `TimeoutError`.

### Timeouts

```python
# Single coroutine with timeout
try:
    result = await asyncio.wait_for(fetch(url), timeout=2.0)
except asyncio.TimeoutError:
    ...

# 3.11+ — timeout as a context manager (scopes any awaits inside)
async with asyncio.timeout(2.0):
    await fetch(url)
    await another()
```

### wait — lower-level than gather

```python
done, pending = await asyncio.wait(
    tasks,
    timeout=5,
    return_when=asyncio.FIRST_COMPLETED,   # or ALL_COMPLETED, FIRST_EXCEPTION
)
for p in pending:
    p.cancel()
```

### as_completed — process results as they finish

```python
for coro in asyncio.as_completed(coros):
    result = await coro
    print(result)
```

### Async sync primitives

```python
lock = asyncio.Lock()
async with lock: ...

sem = asyncio.Semaphore(10)          # limit concurrency (e.g. 10 in-flight requests)
async with sem: ...

event = asyncio.Event()
await event.wait()
event.set()

queue = asyncio.Queue(maxsize=100)   # 0 (default) = unbounded
await queue.put(x)                   # blocks if full
queue.put_nowait(x)                  # raises QueueFull if full
x = await queue.get()                # blocks if empty
queue.get_nowait()                   # raises QueueEmpty if empty
queue.qsize(), queue.empty(), queue.full()
queue.task_done()                    # mark current item processed
await queue.join()                   # wait until all put items are task_done

# Variants
asyncio.PriorityQueue()              # min-heap; items should be (priority, data)
asyncio.LifoQueue()                  # stack

# Producer / consumer pattern
async def producer(q):
    for item in items:
        await q.put(item)

async def consumer(q):
    while True:
        item = await q.get()
        try:
            await process(item)
        finally:
            q.task_done()
```

### Mixing with sync/blocking code

```python
# Run a blocking function in a thread without freezing the event loop
result = await asyncio.to_thread(blocking_fn, arg1, arg2)
```

### Common asyncio pitfalls

- Calling a blocking sync function (e.g. `time.sleep`, `requests.get`) inside async code **freezes the entire event loop**. Wrap with `await asyncio.to_thread(...)`.
- Forgetting to `await` a coroutine — it just creates a coroutine object that never runs. Linters / type checkers will warn.
- `await asyncio.sleep(0)` is the idiom for "yield to the event loop once".
- Mixing threading and asyncio works but is tricky. Pick one model when you can.

---

## Streaming I/O (files & sockets)

### Files

```python
# Line iteration is LAZY — doesn't load the file into memory
with open("big.log") as f:
    for line in f:                       # line includes the trailing \n
        process(line.rstrip())

# Read in chunks (binary or text) — for large files / media
CHUNK = 64 * 1024
with open("video.mp4", "rb") as f:
    while chunk := f.read(CHUNK):        # walrus: stops when read() returns b""
        process(chunk)

# Streaming copy — never holds whole file in RAM
import shutil
with open("src", "rb") as src, open("dst", "wb") as dst:
    shutil.copyfileobj(src, dst)         # does the chunked read/write for you

# Hash a huge file without loading it
import hashlib
h = hashlib.sha256()
with open("big.bin", "rb") as f:
    while chunk := f.read(64 * 1024):
        h.update(chunk)
digest = h.hexdigest()
```

### Sockets — TCP is a stream, not messages

`recv()` can return **fewer** bytes than requested, and returns `b""` when the peer closes. You have to build framing yourself.

```python
import socket

# Read EXACTLY n bytes — for length-prefixed / fixed-size binary protocols
def recv_exact(sock, n):
    buf = bytearray()
    while len(buf) < n:
        chunk = sock.recv(n - len(buf))
        if not chunk:
            raise ConnectionError("socket closed early")
        buf.extend(chunk)
    return bytes(buf)

# Drain until peer closes (for read-until-EOF protocols)
def recv_all(sock):
    chunks = []
    while chunk := sock.recv(64 * 1024):
        chunks.append(chunk)
    return b"".join(chunks)

# Line-oriented protocols (HTTP/1, SMTP, IRC…) — makefile() wraps the socket
# as a file-like object so you get the same lazy line iteration as files
with sock.makefile("rb") as f:
    for line in f:
        if line == b"\r\n":              # blank line ends HTTP headers
            break
        process_header(line)
```

### asyncio streams

Same chunk/line/exact patterns, but using `await` and an `asyncio.StreamReader`.

```python
# Client
reader, writer = await asyncio.open_connection("example.com", 80)
writer.write(b"GET / HTTP/1.0\r\n\r\n")
await writer.drain()                     # flow control: wait if send buffer is full

data  = await reader.read()              # read everything until EOF
chunk = await reader.read(64 * 1024)     # up to N bytes
line  = await reader.readline()          # through next \n (or empty at EOF)
exact = await reader.readexactly(8)      # exactly 8 bytes or IncompleteReadError
async for line in reader:                # lazy line iteration, like file objects
    ...

writer.close()
await writer.wait_closed()

# Server
async def handle(reader, writer):
    line = await reader.readline()
    writer.write(b"got it\n")
    await writer.drain()
    writer.close()
    await writer.wait_closed()

server = await asyncio.start_server(handle, "0.0.0.0", 8888)
async with server:
    await server.serve_forever()
```

---

## JSON

The portable, human-readable, cross-language format. Use this for anything leaving your program (APIs, config, untrusted input) — contrast with [Pickling](#pickling-serialize-python-objects-to-disk) below.

```python
import json

# In memory: object <-> string  (same dump/dumps naming as pickle)
s   = json.dumps(obj)                # object → JSON string  (note the 's')
obj = json.loads(s)                  # JSON string → object

# To/from disk — TEXT mode "w"/"r" (JSON is text, unlike pickle's binary)
with open("data.json", "w") as f:
    json.dump(obj, f)                # dump = write to file (no 's')
with open("data.json") as f:
    obj = json.load(f)               # load = read from file

# Output options
json.dumps(obj, indent=2)            # pretty-printed, indented
json.dumps(obj, sort_keys=True)      # deterministic key order
json.dumps(obj, ensure_ascii=False)  # keep unicode literal instead of \uXXXX escapes

# Type mapping: dict↔object, list/tuple→array, str↔string, int/float↔number,
#               True/False↔true/false, None↔null
# ⚠️ tuples come back as LISTS. dict keys come back as STRINGS (JSON keys must be strings).
# ⚠️ NOT serializable by default: datetime, set, bytes, Decimal, dataclasses, custom objects.

# Serialize the unsupported types via default= (called for anything json can't handle)
json.dumps({"when": now}, default=str)          # stringify dates/Decimal/etc.
from dataclasses import asdict
json.dumps(asdict(my_dataclass))                # dataclass → dict first
```

---

## Pickling (serialize Python objects to disk)

Turns almost any object (dicts, dataclasses, nested graphs, cycles) into bytes and back. Python-only, binary, not human-readable.

```python
import pickle

user = {"name": "Alex", "scores": [1, 2, 3]}   # any ordinary object

blob = pickle.dumps(user)            # object → bytes  (note the 's')
user = pickle.loads(blob)            # bytes → object

# To/from disk — binary mode "wb"/"rb" (pickle is bytes, not text)
with open("data.pkl", "wb") as f:
    pickle.dump(user, f)             # dump = write to file (no 's')
with open("data.pkl", "rb") as f:
    user = pickle.load(f)            # load = read from file

# ⚠️ Dumping straight onto the real path corrupts it if you crash mid-write.
# Safe overwrite (any file, not just pickle): temp file → fsync → atomic rename
import os
with open("data.pkl.tmp", "wb") as f:
    pickle.dump(user, f)
    f.flush(); os.fsync(f.fileno())  # force bytes to disk, not just OS cache
os.replace("data.pkl.tmp", "data.pkl")   # atomic on POSIX; old file intact until here

# ⚠️ NEVER unpickle untrusted data — load() can run arbitrary code (RCE). Use JSON.
# ⚠️ Not stable: tied to your Python version + class definitions. Caches, not archives.
# ⚠️ Can't pickle: open files/sockets, locks, lambdas, local funcs, most generators.
```

### Streaming large data

`dump`/`load` already write/read straight through the file handle (unlike `dumps`/`loads`, which build the full blob in RAM). But `load` rebuilds the *whole object graph at once* — no help for one huge object. It only helps for **many** records: a file can hold many concatenated pickles.

```python
with open("records.pkl", "wb") as f:
    for record in produce_records():     # millions, O(1) memory
        pickle.dump(record, f)           # each call appends one pickle

def load_all(path):
    with open(path, "rb") as f:
        while True:
            try:
                yield pickle.load(f)     # reads exactly one object
            except EOFError:             # ⚠️ how you detect end-of-file
                break
```

---

## Gzip (compression)

```python
import gzip

# In memory: bytes <-> compressed bytes
packed = gzip.compress(b"lots of repetitive data...")   # bytes → smaller bytes
raw    = gzip.decompress(packed)                         # back

# To/from disk: gzip.open is a drop-in for open() — compresses/decompresses for you.
# Binary mode "wb"/"rb" for bytes; text mode "wt"/"rt" needs an encoding.
with gzip.open("data.txt.gz", "wt", encoding="utf-8") as f:
    f.write("hello\n")               # stored compressed
with gzip.open("data.txt.gz", "rt", encoding="utf-8") as f:
    text = f.read()                  # read back decompressed

with gzip.open("blob.gz", "wb", compresslevel=6) as f:   # 1=fast … 9=smallest (default 9)
    f.write(b"...")

# It's a normal file object → lazy line/chunk iteration works, O(1) memory on huge files
with gzip.open("big.log.gz", "rt") as f:
    for line in f:                   # decompresses on the fly
        process(line)

# Combine with pickle / json — just wrap the gzip file object
import pickle
with gzip.open("data.pkl.gz", "wb") as f:
    pickle.dump(obj, f)
with gzip.open("data.pkl.gz", "rb") as f:
    obj = pickle.load(f)
```

⚠️ For an existing file, `compress`/`decompress` load the *whole thing* into memory. Use `gzip.open` (streamed) for large files. Other stdlib codecs share the same API: `bz2` (smaller, slower), `lzma`/`xz` (smallest, slowest).

---

## Imports

```python
import math                          # use as math.sqrt(x)
import math as m                     # alias
from math import sqrt                # use as sqrt(x) directly
from math import sqrt, pi            # multiple names
from math import sqrt as square_root # rename on import
from math import *                   # everything — avoid except in REPL

# Packages (folders, traditionally with __init__.py)
from mypackage.submodule import thing
from mypackage import submodule

# Relative imports — only valid INSIDE a package
from . import sibling                # same package
from .helpers import util            # submodule of same package
from ..other import thing            # parent package

# Make a file usable both as script AND importable module
if __name__ == "__main__":
    main()

# Dynamic import (rare — plugins, config-driven loading)
import importlib
mod = importlib.import_module("mypackage.dynamic")
```

---

## Misc Pythonisms Alex keeps forgetting

```python
# Multiple assignment / swap
a, b = b, a

# Chained comparisons actually do what you'd hope
if 0 < x < 10: ...                   # equivalent to (0 < x) and (x < 10)

# `is` vs `==`
# is: identity (same object).  ==: equality.
# Use `is` for None, True, False, and that's it.
if x is None: ...

# Boolean short-circuit returns the VALUE, not just True/False
name = user_input or "default"       # "default" if user_input is falsy
config = explicit or env_var or fallback

# Star unpacking
first, *middle, last = [1, 2, 3, 4, 5]
combined = [*a, *b, *c]
merged   = {**d1, **d2}

# print options
print("a", "b", sep=", ", end="!\n")
print("err", file=sys.stderr)

# Pretty-print
from pprint import pp
pp(big_nested_thing)

# Iterate index + value: enumerate (NOT range(len(lst)))
for i, x in enumerate(lst): ...

# Build a dict from two parallel lists
dict(zip(keys, values))
```
