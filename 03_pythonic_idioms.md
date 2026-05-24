# Part 3 — Pythonic Idioms

Goal: stop writing Java-in-Python. Cover the language features that separate "Python syntax familiarity" from "another Python developer would recognize this as well-formed Python."

**Prerequisites:** [Part 1 — Syntax Shock](01_syntax_shock.md). **Next:** [Part 4 — Concurrency](04_concurrency.md).

---

## Table of Contents

- [Core differences](#core-differences)
- [Comprehensions](#comprehensions)
- [Args and kwargs](#args-and-kwargs)
- [Iterable vs iterator](#iterable-vs-iterator)
- [Generators](#generators)
- [Context managers](#context-managers)
- [Decorators](#decorators)
- [Type hints](#type-hints)
- [Protocol](#protocol)
- [Match patterns](#match-patterns)
- [Functions as first-class objects](#functions-as-first-class-objects)
- [Functional patterns](#functional-patterns)
- [Scope and nonlocal](#scope-and-nonlocal)
- [Advanced control flow](#advanced-control-flow)
- [Introspection](#introspection)
- [Metaprogramming](#metaprogramming)
- [Key Takeaways](#key-takeaways)

---

## Core differences

| Topic | Java | Python |
| :--- | :--- | :--- |
| Transforming a collection | loop or Stream pipeline | comprehension (literal syntax) |
| Variable-arity arguments | varargs / overloads / builder | `*args` / `**kwargs` + unpacking |
| Iteration | `Iterable<T>` / `Iterator<T>` interfaces | protocol (`__iter__`, `__next__`) — no declaration needed |
| Lazy streams | `Stream` / `Iterator` | generator function (`yield`) |
| Scoped resource cleanup | `try-with-resources` (closeable only) | `with` (any context manager); `async with` for coroutines |
| Cross-cutting concerns | AOP frameworks, proxies | decorators (function or class) |
| Interface contracts | nominal (`implements X`) | both: `abc.ABC` (nominal) and `typing.Protocol` (structural) |

```java
List<Integer> squares = new ArrayList<>();
for (int n : List.of(1, 2, 3)) {
    squares.add(n * n);
}
```

```python
squares = [n * n for n in [1, 2, 3]]
```

That density is the through-line of this part. Each section converts a Java instinct into the lighter Python form.

## Comprehensions

A list comprehension builds a list from another iterable in one expression:

```python
numbers = [1, 2, 3, 4]
squares = [n * n for n in numbers]
print(squares)                         # >>> [1, 4, 9, 16]
```

**Filtering** with a guard:

```python
evens = [n for n in numbers if n % 2 == 0]
print(evens)                           # >>> [2, 4]
```

**Dict** and **set** comprehensions use `{}`:

```python
squares_map = {n: n * n for n in numbers}
print(squares_map)                     # >>> {1: 1, 2: 4, 3: 9, 4: 16}

lengths = {len(word) for word in ["cat", "dog", "apple"]}
print(lengths)                         # >>> {3, 5}
```

**Generator expression** uses `()` — the lazy form. Same syntax, no list built:

```python
total = sum(n * n for n in range(1_000_000))   # no million-element list materialized
```

> 💡 **Pythonic:** Reach for a comprehension when the transformation is one expression and one (maybe two) clauses. If you need nested ifs, multiple statements, or side effects, write a real `for` loop. A comprehension that reads like an essay is the wrong tool.

> ☕ **Java parallel:** Comprehensions replace most one-shot `Stream` pipelines: `stream().map(...).filter(...).collect(toList())` → `[f(x) for x in xs if g(x)]`.

## Args and kwargs

`*args` collects extra positional arguments into a tuple; `**kwargs` collects extra named arguments into a dict.

```python
def add_all(*args):
    return sum(args)

print(add_all(1, 2, 3, 4))             # >>> 10

def print_user(**kwargs):
    for key, value in kwargs.items():
        print(key, value)

print_user(name="Alice", age=25)
# >>> name Alice
# >>> age 25
```

**Argument unpacking** is the reverse — spread a collection into a call:

```python
values = [3, 4]
print(pow(*values))                    # >>> 81      (pow(3, 4))

user = {"name": "Alice", "age": 25}
print_user(**user)                     # same as print_user(name="Alice", age=25)
```

**Star unpacking in assignment** captures "the rest":

```python
first, *rest = [1, 2, 3, 4]
print(first)                           # >>> 1
print(rest)                            # >>> [2, 3, 4]

*head, last = [1, 2, 3, 4]
print(last)                            # >>> 4
```

> ☕ **Java parallel:** Replaces method overloading, varargs-only APIs, and map-of-params/builder patterns. A single signature `def f(*args, **kwargs)` covers what Java would model with several overloads.

## Iterable vs iterator

An **iterable** is anything you can loop over: `list`, `tuple`, `set`, `dict`, `str`, `range`, a generator, a file object.

An **iterator** is a stateful object that produces values one at a time and remembers where it is. You get one by calling `iter()` on an iterable.

```python
numbers = [10, 20, 30]
it = iter(numbers)

print(next(it))   # >>> 10
print(next(it))   # >>> 20
print(next(it))   # >>> 30
# next(it)        # StopIteration
```

A `for` loop is really doing this:

```python
it = iter(iterable)
while True:
    try:
        value = next(it)
    except StopIteration:
        break
    # body
```

> ☕ **Java parallel:** Same shape as `Iterable<T>` / `Iterator<T>`. The difference: Python relies on protocols (any class with `__iter__` and `__next__` works) rather than explicit interface declarations. No `implements Iterable<T>` required.

This protocol-based approach is the bridge to **generators** (next section) and **`Protocol`** (later in this part).

## Generators

A function that uses `yield` becomes a **generator function**. Calling it doesn't run the body — it returns a generator object that produces values one at a time and pauses between them, preserving local state.

```python
def countdown(n):
    while n > 0:
        yield n
        n -= 1

for value in countdown(3):
    print(value)
# >>> 3
# >>> 2
# >>> 1
```

Each `yield` hands a value back and freezes the function until the next `next()` call. This is the lazy iterator pattern with almost no ceremony.

**Why this matters in real code:** streaming. You can process arbitrarily large inputs without loading them.

```python
def read_in_batches(file_path, batch_size=10):
    with open(file_path, "r") as f:
        batch = []
        for line in f:
            batch.append(line.rstrip("\n"))
            if len(batch) == batch_size:
                yield batch
                batch = []
        if batch:
            yield batch

for lines in read_in_batches("large.log", batch_size=10):
    process(lines)
```

The entire file is never in memory. Each call to `yield` hands back one batch, then the function suspends until the next iteration.

**`yield from`** delegates to another iterable:

```python
def flatten(lists):
    for sublist in lists:
        yield from sublist        # equivalent to: for x in sublist: yield x

print(list(flatten([[1, 2], [3, 4], [5]])))   # >>> [1, 2, 3, 4, 5]
```

**Generator expressions** are the inline form (already covered under Comprehensions): `(x*x for x in nums)`.

### Generator vs coroutine

Both pause and resume, but they're for different jobs:

| Concept | Generator | Coroutine |
| :--- | :--- | :--- |
| Defined with | `def` + `yield` | `async def` |
| Main purpose | Produce values lazily | Suspend during async I/O |
| Typical use case | Iteration, streaming, pipelines | Async I/O, event-loop concurrency |
| Pause point | `yield` | `await` |
| Mental model | lazy iterator | async task |

For coroutines, see [Part 4 § Coroutines](04_concurrency.md#coroutines). Historically there were generator-based coroutines too, but modern Python uses `async def`.

## Context managers

The `with` statement runs setup before a block and guaranteed cleanup after — even if an exception is raised. You've already seen `with open(...) as f:`. The pattern is general.

**Implement your own:**

```python
class Demo:
    def __enter__(self):
        print("enter")
        return self                     # bound to `as x:`

    def __exit__(self, exc_type, exc, tb):
        print("exit")
        # return True to suppress the exception; falsy lets it propagate

with Demo() as d:
    print("inside")
# >>> enter
# >>> inside
# >>> exit
```

`__exit__` runs whether the block completed normally or raised. If you return a truthy value from `__exit__`, the exception is suppressed — use this sparingly.

### `@contextmanager` — the shortcut

The `contextlib.contextmanager` decorator turns a generator into a context manager. Code before `yield` runs on entry; code after `yield` runs on exit:

```python
from contextlib import contextmanager
import time

@contextmanager
def timed(label):
    start = time.perf_counter()
    try:
        yield                           # the `with` block runs here
    finally:
        elapsed = time.perf_counter() - start
        print(f"{label}: {elapsed:.3f}s")

with timed("work"):
    sum(range(1_000_000))
# >>> work: 0.012s
```

The `try / finally` around `yield` is the idiomatic shape — it guarantees cleanup even on exception. This is the lightweight way to build a custom context manager without writing a class.

### `contextlib.ExitStack` — dynamic composition

When the number of context managers is decided at runtime, `ExitStack` stacks them programmatically:

```python
from contextlib import ExitStack

paths = ["a.txt", "b.txt", "c.txt"]
with ExitStack() as stack:
    files = [stack.enter_context(open(p)) for p in paths]
    # all files are open here; all guaranteed to close on exit
    for f in files:
        process(f)
```

> ☕ **Java parallel:** "try-with-resources → with." Same idea, but Python `with` works on any object implementing the protocol — not just resources you'd call "closeable." Common Pythonic uses: timing, locks, DB transactions, redirecting `stdout`, suppressing exceptions, mocking in tests.

For async resources (`async with`), see [Part 4 § Async context managers](04_concurrency.md#async-context-managers).

## Decorators

A decorator is a function (or class) that wraps another function or class, returning a replacement. The `@dec` syntax above a definition is sugar:

```python
@some_decorator
def f(x):
    return x

# is equivalent to:
def f(x):
    return x
f = some_decorator(f)
```

That's it. The rest is what people *do* with that pattern.

### Writing one

```python
import functools

def log_calls(func):
    @functools.wraps(func)               # preserve func's name, docstring, signature
    def wrapper(*args, **kwargs):
        print(f"calling {func.__name__}({args}, {kwargs})")
        result = func(*args, **kwargs)
        print(f"  -> {result!r}")
        return result
    return wrapper

@log_calls
def add(a, b):
    return a + b

add(2, 3)
# >>> calling add((2, 3), {})
# >>>   -> 5
```

> ⚠️ **Pitfall:** Always use `@functools.wraps(func)` on the inner wrapper. Without it, the decorated function takes on the wrapper's name and empty docstring, and there's no `__wrapped__` link back to the original (introspection tools like `inspect.signature` follow that link to recover the real signature). Skipping `wraps` breaks debuggers, doc generators, and stack traces.

### Parameterized decorators

When the decorator itself takes arguments, you need one more layer — a decorator factory:

```python
def retry(times):
    def decorator(func):
        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            for attempt in range(times):
                try:
                    return func(*args, **kwargs)
                except Exception:
                    if attempt == times - 1:
                        raise
        return wrapper
    return decorator

@retry(times=3)
def flaky():
    ...
```

Reading inside-out: `retry(times=3)` returns `decorator`; `@decorator` wraps `flaky`. Three nesting levels is the cost — the alternative (a class-based decorator with `__call__`) trades nesting for class boilerplate.

### Stacking and ordering

Decorators apply **bottom-up** — closest-to-the-function first:

```python
@log_calls
@retry(times=3)
def operation():
    ...

# Equivalent to:  operation = log_calls(retry(times=3)(operation))
```

So calls go: `log_calls` outer wrapper → `retry` wrapper → `operation`. If `retry` re-runs on failure, `log_calls` logs once per retry only if it's *inside*. Get this wrong and your retries are silently uncounted in logs.

### Standard-library wins

These are decorators you'll reach for almost every project:

```python
from functools import cache, lru_cache, cached_property

@cache                                   # unbounded; 3.9+
def fib(n):
    return n if n < 2 else fib(n-1) + fib(n-2)

@lru_cache(maxsize=1024)                 # bounded LRU
def expensive(key):
    ...

class User:
    def __init__(self, name): self.name = name
    @cached_property
    def display_name(self):              # computed once per instance
        return self.name.title()
```

> ☕ **Java parallel:** `@functools.cache` and `@lru_cache` are the in-process equivalent of Spring `@Cacheable` or Guava `Cache` (without a distributed backend). `@cached_property` is the field-level once-and-only-once pattern.

You've already seen these decorators in earlier parts:
- `@property`, `@classmethod`, `@staticmethod` — [Part 2 § Class and static methods](02_java_idiom_translation.md#class-and-static-methods), [§ Encapsulation](02_java_idiom_translation.md#encapsulation)
- `@dataclass` — [Part 2 § Dataclass](02_java_idiom_translation.md#dataclass)
- `@abstractmethod` — [Part 2 § Abstract classes](02_java_idiom_translation.md#abstract-classes)
- `@contextmanager` — earlier in this section
- `@pytest.mark.parametrize` — [Part 6 § Pytest](06_ecosystem_and_packaging.md#pytest)
- `@retry` (from `tenacity`) — [Part 6 § HTTP production behavior](06_ecosystem_and_packaging.md#http-production-behavior)

Decorators are everywhere in modern Python. Read them as "this function/class is wrapped to add cross-cutting behavior" and you'll recognize 90% of the AOP-style code you see.

## Type hints

Python is dynamically typed. Type hints are **optional annotations** consumed by tools — mypy, IDEs, runtime validators — not by the interpreter. `add("x", "y")` runs fine even when annotated `def add(a: int, b: int) -> int:`. The tooling complains; the interpreter doesn't.

That's the most important sentence in this section. Type hints buy you static checking and tooling support, not runtime safety. For runtime validation, use `pydantic` (see [Part 6](06_ecosystem_and_packaging.md#productivity-libs)).

### The basics

```python
def greet(name: str, times: int = 1) -> str:
    return ("Hello, " + name + "! ") * times

age: int = 25
users: list[str] = ["Alice", "Bob"]
scores: dict[str, int] = {"Alice": 90}
```

Built-in generic syntax (`list[str]`, `dict[str, int]`, `tuple[int, ...]`) works since 3.9 without imports from `typing`.

### Optional and Union

`None` is a value, so a function that can return `None` must say so:

```python
def find_user(uid: int) -> User | None:           # 3.10+ union syntax — preferred
    ...

from typing import Optional
def find_user(uid: int) -> Optional[User]:        # equivalent, older syntax
    ...
```

`X | Y` is the modern union syntax. `Optional[X]` is exactly `X | None`.

### Generics

```python
def first(items: list[int]) -> int:
    return items[0]

# Generic — works on any element type:
from typing import TypeVar
T = TypeVar("T")

def first(items: list[T]) -> T:
    return items[0]
```

> 🐍 **Python 3.12+:** PEP 695 introduces native generic syntax — no `TypeVar` import needed:
> ```python
> def first[T](items: list[T]) -> T:
>     return items[0]
>
> type IntList = list[int]                  # type alias
>
> class Stack[T]:
>     def __init__(self): self.items: list[T] = []
>     def push(self, item: T) -> None: self.items.append(item)
> ```

### `TypedDict`, `Literal`, `Callable`, `Self`

```python
from typing import TypedDict, Literal, Callable, Self

# TypedDict: a dict with a known shape — useful for JSON-shaped data
class UserDict(TypedDict):
    name: str
    age: int

def make_user() -> UserDict:
    return {"name": "Alice", "age": 25}

# Literal: restrict to specific values
def set_log_level(level: Literal["DEBUG", "INFO", "WARNING", "ERROR"]) -> None:
    ...

# Callable: type for "something you can call"
def apply(fn: Callable[[int, int], int], a: int, b: int) -> int:
    return fn(a, b)

# Self (3.11+): return type referring to the current class — for fluent APIs
class Builder:
    def step(self) -> Self:
        ...
        return self
```

### `Sequence` / `Mapping` vs concrete types

A Java instinct is to declare parameters as `List<T>`. The Pythonic equivalent is the most general protocol you can accept — typically `Iterable`, `Sequence`, or `Mapping`. Pick by what your function actually does to its argument:

```python
from collections.abc import Sequence, Mapping, Iterable

def total(scores: Iterable[int]) -> int:        # only iterates → Iterable accepts list/tuple/set/generator
    return sum(scores)

def first_score(scores: Sequence[int]) -> int:  # needs len + indexing → list/tuple/range OK, NOT generators
    if not scores:
        raise ValueError("empty")
    return scores[0]

def render(config: Mapping[str, str]) -> str:   # needs key lookup → dict and dict-likes
    return config["title"]
```

> ⚠️ **Pitfall:** `Sequence[int]` does *not* accept a generator — sequences support `len()` and indexing, generators don't. If your function only iterates once, declare `Iterable[int]` (this is the genuine `List<T> ≈ Stream<T>` analog for parameters).

Rule of thumb: **accept the broadest type that your code actually uses, return the most specific.** The reverse of what's tempting from Java.

### Escape hatches: `cast`, `Any`, and pragmas

```python
from typing import cast, Any

def f(x: Any) -> int:        # Any disables type checking for x
    return x

raw = some_untyped_lib_call()        # mypy can't infer this
user = cast(User, raw)                # assert to mypy: "trust me, this is a User"
```

When mypy is wrong and you're right, suppress with a pragma — but include the rule name:

```python
result: int = ...
# type: ignore[no-untyped-call]       # the inline form
result = noisy_call()                 # type: ignore[no-untyped-call]

x: User = some_value      # type: ignore[assignment]
```

> ⚠️ **Pitfall:** Bare `# type: ignore` (no `[rule]`) suppresses *every* error on that line — including future real bugs. Always include the specific rule code.

`assert isinstance(x, T)` is the third escape hatch. It both runtime-checks and **narrows the static type** for the rest of the function:

```python
def f(x: object) -> int:
    assert isinstance(x, int)
    return x + 1                      # mypy now knows x is int
```

(But remember [Part 1 § Exception handling](01_syntax_shock.md#exception-handling): `assert` is stripped by `python -O`. Use `isinstance(x, T)` + a real raise for production input validation.)

### What hints do NOT give you

- Runtime validation (use pydantic).
- Generic specialization at runtime — `list[int]` and `list[str]` are the same class.
- Guaranteed correctness — only what mypy/pyright can prove from your code.

Type hints are absolutely worth adopting. Run mypy in CI. Annotate public APIs aggressively, internals lazily. But don't confuse them with Java's compile-time type system: they're a separate tool layered over a dynamic language.

## Protocol

`typing.Protocol` is **structural typing** — duck typing made checkable. A class satisfies a protocol by having the right methods, not by inheriting:

```python
from typing import Protocol

class Closeable(Protocol):
    def close(self) -> None: ...

def safely_close(resource: Closeable) -> None:
    resource.close()

class File:
    def close(self) -> None:
        print("closed")

safely_close(File())          # ✓ — File matches Closeable structurally, no inheritance
```

`File` doesn't inherit from `Closeable`. It doesn't even know `Closeable` exists. mypy checks the shape.

> ☕ **Java parallel:** "The Interface → Protocol (structural) **and** `abc.ABC` (nominal)." Java's `interface` is nominal — implementers must declare `implements X`. Python gives you both. Use `abc.ABC` when you want explicit declared subtyping (a contract the implementer opts into); use `Protocol` when you want duck typing made checkable (a contract any compatible type satisfies).

When to reach for which:
- **`Protocol`** when you accept "anything that quacks" — third-party types you don't control, ad-hoc fakes in tests, common shapes like "anything with `.read()`".
- **`abc.ABC`** when you own the hierarchy and want explicit opt-in, error-at-instantiation guarantees, and shared implementation in the base.

Both are first-class. Pick by intent.

## Match patterns

[Part 1 § Match](01_syntax_shock.md#match) covered literal matches. The full power is **structural matching** — destructuring sequences, mappings, and class instances.

```python
def describe(event):
    match event:
        case {"type": "login", "user": user}:
            return f"login by {user}"
        case {"type": "logout", "user": user}:
            return f"logout by {user}"
        case [first, *rest]:                  # any non-empty sequence
            return f"sequence of {1 + len(rest)} items, first={first}"
        case Point(x=0, y=0):                 # class pattern — see below
            return "origin"
        case _:
            return "unknown"
```

**Class patterns** match by type and destructure attributes:

```python
from dataclasses import dataclass

@dataclass
class Point:
    x: int
    y: int

def quadrant(p):
    match p:
        case Point(x=0, y=0):     return "origin"
        case Point(x=0, y=_):     return "on Y axis"
        case Point(x=_, y=0):     return "on X axis"
        case Point(x=x, y=y) if x > 0 and y > 0: return "Q1"
        case Point():             return "elsewhere"
```

**OR** with `|`:

```python
match command:
    case "quit" | "exit" | "q":
        shutdown()
```

**Guards** with `if`:

```python
match user:
    case User(age=age) if age >= 18:
        ...
```

**Capture** binds a sub-pattern to a name (you saw it above as `user=user`, `x=x`, `*rest`).

> ⚠️ **Pitfall:** Bare names in a pattern **capture**, they don't compare. `case x:` matches anything and binds it to `x` — it does NOT match a variable `x` from enclosing scope. To compare against an existing constant, use a dotted name: `case MyEnum.RED:` (or `case Color.RED:`). Bare `case RED:` would capture, not compare.

Reinforcing [Part 1](01_syntax_shock.md#match): this is structural pattern matching, not Java `switch`. Reach for it when you're destructuring or doing type-based dispatch. Reach for `if`/`elif` when you're just comparing values.

## Functions as first-class objects

Functions are values. You can pass them, store them, return them:

```python
def laugh(): print("ha ha ha")
def cry():   print("waa")

def n_times(fn, n):
    for _ in range(n):
        fn()

n_times(laugh, 2)
# >>> ha ha ha
# >>> ha ha ha
```

**Lambdas** are anonymous functions of one expression:

```python
square = lambda x: x * x
print(square(4))                       # >>> 16

sorted_users = sorted(users, key=lambda u: u.age)
```

Lambdas are restricted to single expressions (no statements). For anything more, use `def` and give it a name — readability wins.

> ☕ **Java parallel:** "The Abstract Factory → first-class functions / closures." A factory class with one method becomes a function returning a function. A strategy interface becomes a callable parameter. A builder becomes `**kwargs`. Many "needs a class" Java patterns become one function in Python.

`map(fn, it)` and `filter(fn, it)` exist but are usually less readable than a comprehension:

```python
list(map(lambda x: x*x, range(5)))     # works
[x*x for x in range(5)]                # preferred
```

## Functional patterns

Python is multi-paradigm. Most code is procedural with OOP, but the functional building blocks are first-class. This section pulls together the pieces a Java/Streams developer reaches for — once you know they exist, you'll see them everywhere.

### `operator` module

For higher-order functions that need "get this key" / "get this attribute" / "call this method," reach for `operator` before reaching for a lambda:

```python
from operator import itemgetter, attrgetter, methodcaller

users = [{"city": "NYC", "name": "Alice", "age": 30},
         {"city": "NYC", "name": "Bob",   "age": 25},
         {"city": "SF",  "name": "Carol", "age": 28}]

sorted(users, key=itemgetter("age"))               # sort by single dict key
sorted(users, key=itemgetter("city", "age"))       # multi-key sort (tuple form)

class User:
    def __init__(self, name, age):
        self.name, self.age = name, age

people = [User("Alice", 30), User("Bob", 25)]
sorted(people, key=attrgetter("age"))              # sort by attribute

list(map(methodcaller("upper"), ["alice", "bob"])) # call .upper() on each
# >>> ['ALICE', 'BOB']
```

> 💡 **Pythonic:** `itemgetter("age")` is both faster and clearer than `lambda x: x["age"]`. Same for `attrgetter` and `methodcaller`. Reach for `operator` before reaching for `lambda` whenever the lambda body is just a getter or method call.

### `functools.reduce`

The Java `Stream.reduce()` analog. Less common in Python than in Java because comprehensions plus `sum`/`max`/`min`/`math.prod` cover most cases — so reach for `reduce` only when **no built-in fits**. Two genuine examples:

```python
from functools import reduce
import operator

# Intersect N sets — no built-in for "all-pairs intersection"
common = reduce(operator.and_, [{1, 2, 3}, {2, 3, 4}, {3, 4, 5}])
print(common)                                  # >>> {3}

# Merge N dicts left-to-right (later wins) — operator | is dict merge in 3.9+
merged = reduce(operator.or_, [{"a": 1}, {"b": 2}, {"a": 3}])
print(merged)                                  # >>> {'a': 3, 'b': 2}
```

Anti-patterns to recognize and avoid:

```python
# Don't: textbook-but-wrong — built-ins are clearer AND preserve first-max on ties
worst = reduce(lambda a, b: a if len(a) >= len(b) else b, ["apple", "fig", "banana"])
# Do:
best  = max(["apple", "fig", "banana"], key=len)

# Don't: reduce(operator.add, xs, 100)
# Do:    sum(xs, start=100)
```

Rule of thumb: try `sum`, `min`, `max`, `math.prod`, `any`, `all`, `math.gcd` first. Reach for `reduce` only when the accumulator function genuinely has no built-in.

### `functools.partial`

Fix some arguments of a function, get a new function back:

```python
from functools import partial
import logging

logger = logging.getLogger(__name__)

# Pre-fill a logging call with level and component name
debug_log = partial(logger.log, logging.DEBUG)
debug_log("starting")                       # same as logger.log(logging.DEBUG, "starting")

# Adapt a 3-arg function to a 1-arg interface
def fetch(url, timeout, retries):
    ...
fetch_with_defaults = partial(fetch, timeout=5, retries=3)
fetch_with_defaults("https://example.com")  # only need to pass url
```

> ☕ **Java parallel:** `partial(f, x)` ≈ a lambda capturing `x`. Less a "currying" tool than a clean way to adapt one signature to another at the call site — common as a callback that needs extra context.

### Stream → Python idiom map

| Java Streams | Python idiom |
| :--- | :--- |
| `stream().map(f).toList()` | `[f(x) for x in xs]` or `list(map(f, xs))` |
| `stream().filter(p).toList()` | `[x for x in xs if p(x)]` or `list(filter(p, xs))` |
| `stream().reduce(0, Integer::sum)` | `sum(xs)` (or `reduce(operator.add, xs, 0)`) |
| `stream().sorted(comparing(...))` | `sorted(xs, key=...)` |
| `stream().min(comparing(...))` | `min(xs, key=...)` |
| `stream().mapToInt(f).sum()` | `sum(f(x) for x in xs)` (generator expression — lazy, no list) |
| `stream().count()` | `sum(1 for _ in xs)` or `len(xs)` if it's a list |
| `stream().distinct()` | `set(xs)` (loses order) or `list(dict.fromkeys(xs))` (preserves order) |
| `Collectors.groupingBy(f)` | `defaultdict(list)` + single-pass loop (true materialized `Map<K, List<V>>` — see below) |
| `Collectors.partitioningBy(p)` | single-pass loop into `truthy` / `falsy` lists (see below — two-comprehension form fails on iterators) |
| `Collectors.joining(", ")` | `", ".join(str(x) for x in xs)` |
| `Optional<T>.map(f)` | `f(x) if x is not None else None` |
| `Optional<T>.orElse(d)` | `x if x is not None else d` — same shape; Java `orElse(d)` evaluates `d` eagerly because it's an argument |
| `Optional<T>.orElseGet(() -> d())` | `x if x is not None else d()` — Python's conditional expression is lazy on the unused branch, so `d()` runs only when `x is None` |
| `Function.andThen(g)` | `lambda x: g(f(x))` for one-off composition; `toolz.compose` or `functools.reduce` for chains (no stdlib `compose`) |
| `Function.identity()` | `lambda x: x` |

### Grouping and partitioning

Java's `Collectors.groupingBy` returns a materialized `Map<K, List<V>>`. The Pythonic equivalent is **not** `itertools.groupby` — `groupby` only groups *adjacent* equal-keyed items and returns one-shot iterators. The materialized idiom is a single-pass loop into `defaultdict(list)`:

```python
from collections import defaultdict

people = [{"name": "Alice", "city": "NYC"},
          {"name": "Bob",   "city": "SF"},
          {"name": "Carol", "city": "NYC"}]

by_city = defaultdict(list)
for p in people:
    by_city[p["city"]].append(p["name"])

print(dict(by_city))    # >>> {'NYC': ['Alice', 'Carol'], 'SF': ['Bob']}
```

O(N), no sort required, no iterator-exhaustion footguns. Use `itertools.groupby` only when the input is already sorted by the key and you want "compress adjacent runs."

Partitioning (boolean grouping) is the same shape — single pass into two lists, never two comprehensions over the same input:

```python
def partition(pred, iterable):
    truthy, falsy = [], []
    for x in iterable:
        (truthy if pred(x) else falsy).append(x)
    return truthy, falsy

evens, odds = partition(lambda n: n % 2 == 0, range(10))
print(evens, odds)      # >>> [0, 2, 4, 6, 8] [1, 3, 5, 7, 9]
```

> ⚠️ **Pitfall:** Don't write `([x for x in xs if p(x)], [x for x in xs if not p(x)])`. It iterates twice (calling `p` twice per element) and **silently breaks** if `xs` is a generator or any one-shot iterator — the first comprehension exhausts the source and the second list comes out empty.

### Generator pipelines

Generators (see [Generators](#generators)) compose into **lazy** pipelines — values flow through stage by stage without ever building intermediate lists. This is the closest match to Java's `Stream` semantics:

```python
def words(file):
    for line in file:
        for w in line.split():
            yield w

def lengths(it):
    for w in it:
        yield len(w)

# Lazy end-to-end: no list ever built
with open("text.txt") as f:
    total = sum(lengths(words(f)))
```

Each pipeline stage pulls one value through at a time. Memory use stays constant regardless of file size — the same property that makes Java `Stream` attractive.

### `itertools` as combinators

The `itertools` module ([Part 5 § Stdlib quick-wins](05_standard_library.md#stdlib-quick-wins)) is a functional toolkit for iterators. The Stream operations without a direct comprehension equivalent live here:

| Combinator | Use |
| :--- | :--- |
| `chain(a, b, ...)` | concatenate iterables lazily |
| `groupby(it, key)` | run-length encode by key (sort first!) |
| `accumulate(it, fn)` | running totals / prefix folds |
| `dropwhile(p, it)` / `takewhile(p, it)` | skip-while / take-while |
| `islice(it, start, stop, step)` | slice that works on any iterator (not just lists) |
| `product(a, b)` / `combinations(it, r)` / `permutations(it, r)` | combinatorics |
| `zip_longest(...)` | zip that pads with a fill value instead of truncating |
| `tee(it, n)` | duplicate an iterator into N independent iterators |

> 💡 **Pythonic:** Pythonic functional code is **comprehensions + generators + small named functions + `operator` / `functools` / `itertools` helpers**. It is NOT deep currying, point-free style, or maximum compositionality. Python's syntax doesn't reward those, and reviewers won't either. Reach for FP when the data flow is the story; reach for procedural when state is.

## Scope and nonlocal

Python has three scopes (plus built-ins): local, enclosing, global. A name lookup walks them in that order — LEGB (Local, Enclosing, Global, Built-in).

```python
x = "global"

def outer():
    x = "enclosing"
    def inner():
        x = "local"
        print(x)
    inner()

outer()                                # >>> local
```

To **modify** an enclosing-function variable from a nested function, use `nonlocal`:

```python
def make_counter():
    count = 0
    def step():
        nonlocal count
        count += 1
        return count
    return step

c = make_counter()
print(c(), c(), c())                   # >>> 1 2 3
```

To modify a module-level (global) variable from inside a function, use `global` — sparingly. (If you reach for `global` regularly, you probably want a class.)

> ☕ **Java parallel:** Java lambdas can only read effectively final variables. Python `nonlocal` lets the inner function reassign the enclosing scope's binding. More flexible, and the source of much of the closure-based Python you'll see.

## Advanced control flow

A few syntactic features that aren't survival-critical but you'll meet in real code.

### `assert`

Runs an invariant check. Raises `AssertionError` on failure:

```python
def divide(a, b):
    assert b != 0, "b must not be zero"
    return a / b
```

> ⚠️ **Pitfall:** `assert` is stripped when you run Python with `-O`. Use it for invariants and debugging — never for input validation or business rules. Use `if not x: raise ValueError(...)` for things that must always run.

### `while … else` and `for … else`

The `else` clause runs **only if the loop completed without `break`**:

```python
for n in [2, 4, 6, 8]:
    if n == 5:
        print("Found")
        break
else:
    print("Not found")
# >>> Not found

for n in [2, 4, 5, 6]:
    if n == 5:
        print("Found")
        break
else:
    print("Not found")
# >>> Found
```

Useful for search loops. The mental model: **loop-`else` = "no break happened."** (Surprising name; if Python could rename it, it'd be `nobreak:`.)

### `del`

Removes a binding, list element, dict key, slice, or attribute. Does not force garbage collection — only removes the reference.

```python
numbers = [10, 20, 30]
del numbers[1]
print(numbers)                         # >>> [10, 30]

user = {"name": "Alice", "age": 25}
del user["age"]
print(user)                            # >>> {'name': 'Alice'}
```

## Introspection

Python lets you ask objects about themselves at runtime — the lighter Pythonic analog of Java reflection.

```python
value = [1, 2, 3]
print(type(value))                     # >>> <class 'list'>
print(isinstance(value, list))         # >>> True

class User:
    def __init__(self):
        self.name = "Alice"

u = User()
print(hasattr(u, "name"))              # >>> True
print(getattr(u, "name"))              # >>> Alice
setattr(u, "age", 30)
print(u.age)                           # >>> 30

print(dir("hello"))                    # all attributes/methods of a str
```

The `inspect` module goes further — signatures, source, call frames:

```python
import inspect

def greet(name: str) -> str:
    return f"Hello, {name}"

print(inspect.signature(greet))        # >>> (name: str) -> str
print(inspect.getsource(greet))
```

This is the day-to-day reflection-style API. Most reflection use cases in Python (decorators, registries, custom validators) lean on this rather than on a separate framework.

## Metaprogramming

Classes are runtime objects, so you can modify them dynamically. Add methods, build classes from scratch, install class-creation hooks.

**Add a method at runtime:**

```python
class Person:
    def __init__(self, name): self.name = name

def greet(self):
    return f"Hi, {self.name}"

Person.greet = greet
print(Person("Alice").greet())         # >>> Hi, Alice
```

**Build a class with `type(...)`:**

```python
User = type("User", (), {"role": "admin"})
print(User.role)                       # >>> admin
```

**Metaclasses** (the class of a class — `type` is the default) let you intercept class creation itself. They're powerful and rarely needed:

```python
class TrackedMeta(type):
    instances = []
    def __call__(cls, *args, **kwargs):
        obj = super().__call__(*args, **kwargs)
        TrackedMeta.instances.append(obj)
        return obj

class Widget(metaclass=TrackedMeta):
    pass

Widget(); Widget()
print(len(TrackedMeta.instances))      # >>> 2
```

> 💡 **Pythonic:** Before reaching for metaclasses, ask whether a **decorator** (function or class), `__init_subclass__` ([Part 2](02_java_idiom_translation.md#init-subclass)), or a `Protocol` would do the job. Metaclasses are appropriate when you genuinely need to intercept class *creation* (registries, ORM model definition, framework plumbing) — for most cross-cutting behavior, decorators are simpler.

## Key Takeaways

- Comprehensions for transformation; generators for streaming; both keep memory and code small.
- Decorators are the cross-cutting tool — `@functools.cache` is the gateway drug; always `@functools.wraps`.
- `Protocol` for structural typing, `abc.ABC` for nominal — Python gives you both.
- Type hints aren't runtime validation. Use pydantic for that. Annotate, run mypy, but don't confuse them with Java's compile-time guarantees.
- Reach for `Sequence`/`Mapping`/`Iterable` over concrete `list`/`dict` in function signatures.
- `match` is structural pattern matching — destructure, don't `switch`.
- Reach for metaclasses only when decorators, `__init_subclass__`, or `Protocol` won't fit.
- Pythonic FP = comprehensions + generators + `operator` / `functools` / `itertools` helpers. Not deep currying.
