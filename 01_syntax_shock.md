# Part 1 — Syntax Shock

Goal: read Python code without flinching. Cover what looks different, the few words you need to know, and the small set of features that surprise Java developers in their first hour.

**Prerequisites:** none. **Next:** [Part 2 — Java Idiom Translation](02_java_idiom_translation.md).

---

## Table of Contents

- [Core differences](#core-differences)
- [Typing basics](#typing-basics)
- [Functions](#functions)
- [Execution model](#execution-model)
- [Entry point](#entry-point)
- [Control flow](#control-flow)
- [None and is](#none-and-is)
- [Truthiness](#truthiness)
- [Match](#match)
- [Sequence utilities](#sequence-utilities)
- [Range](#range)
- [Complex](#complex)
- [Str vs bytes](#str-vs-bytes)
- [F-strings](#f-strings)
- [Legacy string formatting](#legacy-string-formatting)
- [Walrus](#walrus)
- [Exception handling](#exception-handling)
- [Modules and imports](#modules-and-imports)
- [Key Takeaways](#key-takeaways)

---

## Core differences

| Topic | Java | Python |
| :--- | :--- | :--- |
| Typing | Statically typed | Dynamically typed (with optional hints) |
| Syntax | Braces, semicolons, more boilerplate | Indentation-based, concise |
| Entry point | `main()` required | Top-level code runs directly |
| Paradigm | Primarily OOP | Multi-paradigm |
| Compilation | Source → bytecode → JVM with JIT | Compiles to bytecode internally; interpreted execution |

The same trivial program, side by side:

```java
public class Main {
    public static void main(String[] args) {
        int a = 5;
        int b = 3;
        System.out.println(a + b);
    }
}
```

```python
a = 5
b = 3
print(a + b)
```

The mindset shift this section sets up is small but real: less ceremony, indentation matters, and the file *is* the entry point unless you say otherwise.

## Typing basics

Java requires explicit types at declaration:

```java
int age = 25;
String name = "Alice";
```

Python infers types at runtime:

```python
age = 25
name = "Alice"
```

Python also supports optional **type hints**. They don't affect runtime behavior — they're for tooling (mypy, IDE autocomplete, documentation):

```python
def add(a: int, b: int) -> int:
    return a + b
```

> 💡 **Pythonic:** Hints are checked by a separate tool, not by the interpreter. `add("x", "y")` runs without complaint. See [Part 3 § Type hints](03_pythonic_idioms.md#type-hints) for the full story.

## Functions

Functions are lightweight. There's no enclosing class required. Multiple return values come back naturally via tuple unpacking:

```python
def process_numbers(a, b):
    return a + b, a * b

sum_val, prod_val = process_numbers(5, 3)
print(sum_val, prod_val)
# >>> 8 15
```

That `return a + b, a * b` is implicitly a tuple. The receiving `sum_val, prod_val = ...` unpacks it. Java patterns that use a record or a holder class for "two return values" disappear.

## Execution model

- **Java:** source → `.class` bytecode → JVM → JIT optimization.
- **Python:** source compiled to `.pyc` bytecode internally → interpreted by CPython (the reference implementation).

The day-to-day implication: Python has no separate compile step you invoke. `python script.py` does it all. You'll feel this most in fast iteration loops.

## Entry point

In Java, execution starts from `main()`. In Python, top-level code runs immediately when the file is imported or executed.

To make a file behave like a script only when run directly — and not run that code if it's imported as a module — use the idiom:

```python
if __name__ == "__main__":
    print("Run only when this file is executed directly")
```

`__name__` is set to `"__main__"` when the file is the entry point, and to the module's name otherwise. Treat the `if __name__ == "__main__":` block as "this is the `main` method."

## Control flow

The keywords look familiar but a handful are different:

| Java | Python |
| :--- | :--- |
| `else if` | `elif` |
| `&&` / `\|\|` / `!` | `and` / `or` / `not` |
| `catch` | `except` |
| `throw` | `raise` |
| `{}` empty block | `pass` |
| `switch` | `match` (3.10+, structural pattern matching) |

`pass` deserves a callout: Python doesn't have empty braces, so `pass` is the placeholder when syntax requires a body but you have nothing to put there yet:

```python
class TODO:
    pass

def not_implemented_yet():
    pass
```

## None and is

`None` is Python's equivalent of Java `null`, but it's a real object (a singleton of type `NoneType`):

```python
print(type(None))
# >>> <class 'NoneType'>
```

Two operators look similar but mean different things:

- `==` checks **value equality** (calls `__eq__`)
- `is` checks **object identity** (same object in memory)

```python
a = [1, 2]
b = [1, 2]

print(a == b)   # >>> True   (same contents)
print(a is b)   # >>> False  (different objects)
```

> 💡 **Pythonic:** Use `is` for `None` and other singletons (`is True`, `is False` if you really mean the singleton). Use `==` for value comparison.

```python
value = None
if value is None:
    print("No value")
```

The Java intuition `.equals()` vs `==` carries over: `is` is the reference-equality `==`; Python's `==` is the `.equals()` you actually want most of the time.

## Truthiness

In Java, `if (...)` requires a boolean expression. In Python, many objects can be used directly in conditionals. The **falsy** values are:

- `None`
- `False`
- `0`, `0.0`, `0j`
- `""` (empty string)
- `[]`, `{}`, `set()`, `()` (empty containers)

Everything else is truthy.

```python
items = []
if not items:
    print("Empty")
# >>> Empty
```

The idiomatic Python check for "non-empty list" is `if items:`, not `if len(items) > 0:`. Adopt this — Python reviewers will flag the verbose form.

## Match

Python 3.10+ supports `match ... case`, which is **structural pattern matching**, not Java's C-style `switch`:

```python
key = "c"

match key:
    case "a":
        result = 1
    case "b":
        result = 2
    case "c":
        result = 3
    case _:
        result = -1

print(result)
# >>> 3
```

The `_` is the wildcard catch-all (no equivalent to `default:` keyword).

> ⚠️ **Pitfall:** `match` is **not** Java `switch`. There's no fall-through. `case "a", "b":` does **not** mean "a OR b" — it means "a tuple of two strings." For OR, write `case "a" | "b":`. Pattern matching's real power is destructuring objects/sequences/mappings — covered in [Part 3 § Match patterns](03_pythonic_idioms.md#match-patterns).

## Sequence utilities

A handful of built-in functions show up in almost every Python codebase. They work on any iterable.

`zip(a, b, ...)` pairs up elements:

```python
x = [1, 2, 3]
y = [4, 5, 6]

for a, b in zip(x, y):
    print(a, b)
# >>> 1 4
# >>> 2 5
# >>> 3 6
```

`any(it)` and `all(it)` short-circuit boolean reductions:

```python
print(all([True, True, True]))    # >>> True
print(all([True, False, True]))   # >>> False
print(any([False, True, False]))  # >>> True
```

`sorted(it)` returns a new sorted list; `reversed(it)` returns an iterator in reverse:

```python
numbers = [1, 5, 3, 7, 8, 2]

print(sorted(numbers))         # >>> [1, 2, 3, 5, 7, 8]
print(list(reversed(numbers))) # >>> [2, 8, 7, 3, 5, 1]
```

Both `sorted` and `reversed` leave the original untouched (unlike `list.sort()` / `list.reverse()`, which mutate in place).

## Range

`range(start, stop, step)` represents a sequence of integers — but it's **not** a list. It's a lazy, memory-efficient range object commonly used in loops:

```python
numbers = range(1, 6)
print(numbers)              # >>> range(1, 6)
print(list(numbers))        # >>> [1, 2, 3, 4, 5]

for i in range(0, 10, 2):
    print(i)
# >>> 0 2 4 6 8
```

For Java developers, this is what you reach for when you'd write a classic indexed `for` loop. Note that `range(1, 6)` stops at 5 (the `stop` is exclusive), matching Java's `for (int i = 1; i < 6; i++)`.

## Complex

Python has a built-in complex number type — something Java doesn't ship in its core language:

```python
z = 3 + 4j
print(z.real)   # >>> 3.0
print(z.imag)   # >>> 4.0

z1 = 1 + 2j
z2 = 3 + 4j
print(z1 + z2)  # >>> (4+6j)
print(z1 * z2)  # >>> (-5+10j)
```

Useful in scientific, signal-processing, or any code that touches the complex plane. No external library needed.

## Str vs bytes

This is one of the most important Python concepts for a Java developer. Python makes a hard distinction between:

- **`str`** — Unicode text
- **`bytes`** — raw binary data

A Python `str` is text. It is not "a UTF-8 string" or "a GBK string" internally — those words apply only when you convert text into bytes (or vice versa) at an I/O boundary.

**Encode (`str` → `bytes`):**

```python
text = "你好, Python"
utf8_data = text.encode("utf-8")
gbk_data = text.encode("gbk")

print(utf8_data)  # >>> b'\xe4\xbd\xa0\xe5\xa5\xbd, Python'
print(gbk_data)   # >>> b'\xc4\xe3\xba\xc3, Python'
```

**Decode (`bytes` → `str`):**

```python
raw = b"hello"
text = raw.decode("utf-8")
print(text)  # >>> hello
```

**File handling:** specify encoding explicitly when you read or write text files.

```python
with open("notes.txt", "w", encoding="utf-8") as f:
    f.write("你好")

with open("notes.txt", "r", encoding="utf-8") as f:
    print(f.read())  # >>> 你好
```

> ⚠️ **Pitfall:** Decoding with the wrong encoding produces garbled text or raises `UnicodeDecodeError` / `UnicodeEncodeError`. Decoding UTF-8 bytes as GBK is the classic "mojibake" trap.

> ☕ **Java parallel:** Mirrors Java's `String` / `byte[]` / `Charset` separation, but Python makes the distinction visible in everyday code. Practical rule: keep data as `str` as long as possible; only `.encode()` / `.decode()` at the I/O edge.

## F-strings

The modern way to format strings (3.6+):

```python
name = "Alice"
age = 25
pi = 3.14159

print(f"{name} is {age}")           # >>> Alice is 25
print(f"Pi ≈ {pi:.2f}")             # >>> Pi ≈ 3.14
print(f"|{name:>10}|")              # >>> |     Alice|     (right-aligned width 10)
print(f"|{name:<10}|")              # >>> |Alice     |     (left-aligned)
print(f"|{name:^10}|")              # >>> |  Alice   |     (centered)
print(f"{age=}")                    # >>> age=25            (self-documenting, 3.8+)
print(f"{pi * 2 = :.3f}")           # >>> pi * 2 = 6.283    (inline expression)
```

The `{x=}` form is gold for debug prints — it includes the variable name automatically.

## Legacy string formatting

You'll encounter two older formatting styles in real codebases — they still work and aren't deprecated:

```python
# .format() method (3.0+)
print("{} is {}".format("Alice", 25))           # >>> Alice is 25
print("{name} is {age}".format(name="A", age=25))

# % formatting (C printf style, oldest)
print("%s is %d" % ("Alice", 25))               # >>> Alice is 25
```

You will also see `%` formatting inside `logging` calls — and that's idiomatic, not legacy:

```python
import logging
logger = logging.getLogger(__name__)
username = "alice"
logger.info("user %s logged in", username)
```

The `logger.info("...", arg)` form defers string formatting until the logger actually needs to emit the message. If the log level is suppressed, no formatting work happens. f-strings would do the work unconditionally.

Use f-strings for everything else.

## Walrus

The walrus operator `:=` assigns inside an expression (Python 3.8+). The idiomatic use is **assign-and-test** in conditions and comprehensions:

```python
# In a comprehension: compute once, filter and use
data = ["abc", "", "x", "hello"]
results = [u for s in data if (u := s.upper())]
print(results)  # >>> ['ABC', 'X', 'HELLO']

# In a while loop: read-until-EOF
with open("file.txt") as f:
    while (line := f.readline()):
        print(line.rstrip())

# In an if: capture the matched object
import re
if (m := re.search(r"\d+", "Order 1234")):
    print(m.group())  # >>> 1234
```

Don't use the walrus where a plain assignment would do — it's there to avoid recomputing or to bring a temporary into scope mid-expression.

## Exception handling

Python's exception model is lighter than Java's:

```python
try:
    value = int("42")
except ValueError:
    print("Bad input")
else:
    print("Conversion succeeded")
finally:
    print("Always runs")
```

The `else` runs only when no exception was raised — useful for code that should run on success but should not be guarded by the same `except`.

**Raising and custom exceptions:**

```python
def withdraw(balance, amount):
    if amount > balance:
        raise ValueError("Insufficient funds")
    return balance - amount


class InvalidUserInputError(Exception):
    pass
```

**Chaining causes** (analogous to Java `throw new X(msg, cause)`):

```python
try:
    int("not a number")
except ValueError as e:
    raise RuntimeError("Could not parse user input") from e
```

The `from e` preserves the original exception in `__cause__` and the traceback shows both — exactly what `getCause()` gives you in Java.

**Key model difference:** Python has **no checked exceptions**. You don't declare what a function might raise; callers aren't forced to handle anything. This reduces ceremony but makes it your responsibility to document and handle errors thoughtfully.

> ⚠️ **Pitfall:** Python's `assert` statement exists but it's **stripped by the `-O` optimization flag**. Use it for invariants and debugging only — never for input validation or business rules. Deep treatment in [Part 3 § Advanced control flow](03_pythonic_idioms.md#advanced-control-flow).

For concurrent-failure aggregation, see [Part 4 § Exception groups](04_concurrency.md#exception-groups).

## Modules and imports

In Python, **a file is a module** and **a folder (with code in it) can be a package**. The import system is file-oriented, not class-oriented.

```python
import math
print(math.sqrt(16))                # >>> 4.0

from math import sqrt
print(sqrt(16))                     # >>> 4.0

import pandas as pd                 # alias
```

You can import:
- the whole module: `import math` → `math.sqrt(...)`
- specific names: `from math import sqrt, pi`
- with an alias: `import numpy as np`
- everything (avoid): `from math import *`

Java developers think in terms of packages → classes → static imports. Python thinks in terms of files → top-level names. That makes imports flexible, but two things bite:

1. **Circular imports.** If `a.py` imports `b.py` and `b.py` imports `a.py`, you'll get partially-initialized module errors. Restructure so the shared code lives in a third module, or move the import inside a function.
2. **Module-level side effects.** Code at the top of a module runs on import. A file that does `connect_to_db()` at import time will surprise you.

## Key Takeaways

- Indentation defines blocks — no braces.
- `None` is an object; compare with `is`, not `==`.
- Truthiness is a feature: `if items:` is idiomatic for "non-empty".
- `str` is Unicode text; `bytes` is raw bytes. Encode/decode only at I/O boundaries.
- `match` is structural pattern matching, not Java `switch` — no fall-through, `,` means tuple, use `|` for OR.
- `if __name__ == "__main__":` for script-only execution.
- Python has no checked exceptions — discipline matters more.
