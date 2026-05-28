# Part 1 — Syntax Shock

Goal: read Python code without flinching. Cover what looks different, the few words you need to know, and the small set of features that surprise Java developers in their first hour.

**Prerequisites:** none. **Next:** [Part 2 — Java Idiom Translation](02_java_idiom_translation.md).

---

## Table of Contents

- [Core differences](#core-differences)
- [Variables and the object model](#variables-and-the-object-model)
- [Typing basics](#typing-basics)
- [Comments and docstrings](#comments-and-docstrings)
- [Functions](#functions)
- [Execution model](#execution-model)
- [Entry point](#entry-point)
- [Operators](#operators)
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
- [Console I/O](#console-io)
- [Walrus](#walrus)
- [Exception handling](#exception-handling)
- [Modules and imports](#modules-and-imports)
- [Python-specific keywords](#python-specific-keywords)
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

## Variables and the object model

A variable in Python is a **label** bound to an object on the heap. Not a typed slot, not a box holding a value — just a name with a pointer.

```python
x = 5
y = x                  # y is another name pointing at the same int
print(x is y)          # >>> True   (same object — see § None and is for what `is` checks)
```

Three consequences a Java dev should internalise early:

1. **Everything is an object** — `int`, `bool`, `float`, `None`, functions, classes, modules, all of it. There are no primitive types like Java's `int` / `long` / `boolean`. `True` is a singleton instance of `bool`; `42` is a real `int` object you can call methods on (`(42).bit_length()` returns `6`).

2. **Assignment never copies.** `b = a` binds `b` to whatever object `a` is already pointing at. For immutable objects (`int`, `str`, `tuple`, `frozenset`) you can't tell the difference; for mutable objects (`list`, `dict`, `set`) you can — modifying through one name shows through the other. Deep treatment in [Part 2 § Mutability](02_java_idiom_translation.md#mutability).

3. **Garbage collection is automatic** — just like the JVM. CPython uses reference counting plus a cycle collector for unreachable reference cycles; there is no `free()` or `delete` to call. (`del x` exists, but it only unbinds the name `x` — the object lives as long as anything else still references it.)

> ☕ **Java parallel:** Java splits the world into primitives (`int`, `boolean`, …) that live by value and reference types (objects, including boxed `Integer`) that live on the heap. Python skips that split — there is only the reference model. The good news for Java devs: this is the model you already use for every non-primitive in Java. The new part is that it applies to *everything*, including numbers and booleans.

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

## Comments and docstrings

Python has **one** comment syntax — `#` to end of line. There is no `/* ... */` block-comment form:

```python
# single-line comment

x = 1   # inline comment after code

# "Block comments" are just multiple single-line comments stacked.
# Each line needs its own #.
# Editors usually have a key to toggle this for a selection.
```

**Docstrings** are NOT comments. They're triple-quoted string literals that appear as the first statement of a `def` / `class` / module body. The interpreter binds them to `__doc__` and tools (`help()`, Sphinx, IDE hover, etc.) pick them up:

```python
def withdraw(balance: float, amount: float) -> float:
    """Subtract amount from balance.

    Raises ValueError if amount exceeds balance.
    """
    if amount > balance:
        raise ValueError("Insufficient funds")
    return balance - amount

print(withdraw.__doc__)        # the docstring as a str
help(withdraw)                  # rendered docstring + signature
```

> ☕ **Java parallel:** `#` → `//`. No direct equivalent of `/* ... */` — use stacked `#` lines instead. Docstrings (`""" ... """`) are the Python analog of `/** ... */` Javadoc — same role (API documentation, picked up by tooling), different mechanism (real string objects attached to the function/class/module).

> ⚠️ **Pitfall:** You'll see triple-quoted strings used as comment-like blocks in the middle of functions — `"""this is a fake comment"""` on its own line. Technically that's an expression statement whose value (a `str`) is discarded; the interpreter doesn't care, but it's not a real comment and linters flag it. Use `#` for real comments. Triple-quoted strings are for docstrings (first statement of a `def`/`class`/module) and multi-line string values, not as scratch comments.

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

**Default parameter values** make many Java overloads unnecessary:

```python
def greet(name, salutation="Hi"):
    print(f"{salutation}, {name}")

greet("Alice")                   # >>> Hi, Alice
greet("Bob", "Hello")            # >>> Hello, Bob
greet("Carol", salutation="Hey") # keyword argument — explicit at the call site
```

The mutable-default trap (`def f(items=[])`) is real and covered in [Part 2 § Mutability](02_java_idiom_translation.md#mutability). For `*args` / `**kwargs` and unpacking see [Part 3 § Args and kwargs](03_pythonic_idioms.md#args-and-kwargs).

## Execution model

- **Java:** source → `.class` bytecode → JVM → JIT optimization.
- **Python:** source compiled to `.pyc` bytecode internally → interpreted by CPython (the reference implementation).

The day-to-day implication: Python has no separate compile step you invoke. `python script.py` does it all. You'll feel this most in fast iteration loops.

### Python implementations

"Python" is a language specification. The interpreter that runs it is an implementation choice. When someone says "Python" they almost always mean **CPython** — but a Java developer should know the alternatives, because some run on the JVM.

| Implementation | What it is | When it matters |
| :--- | :--- | :--- |
| **CPython** | Reference implementation in C. From python.org. | Default for ~99% of Python in the wild. C extensions (NumPy, pandas, cryptography, …) all target it. The GIL is a CPython detail. |
| **PyPy** | Independent implementation with a tracing JIT. | ~3× average over CPython on long-running pure-Python compute (per the project's own benchmarks); workloads dominated by C extensions can be slower because the C-API shim (`cpyext`) adds overhead. |
| **Jython** | Python on the JVM. Compiles to Java bytecode; native interop with Java classes. | When you must run inside a JVM application. Caveat: largely stuck on Python 2.x; Jython 3 has been in progress for years. |
| **GraalPy** | Modern JVM-based Python on GraalVM. Python 3. | The 2026 answer for "Python + Java in one process." Run Python alongside Java, JS, Ruby on the same runtime. |
| **IronPython** | Python on .NET. IronPython 3 reached Python-3 parity (3.4) and is actively maintained, unlike Jython. | If you live in the .NET world. .NET interop is the selling point; you can call .NET libraries from Python and embed Python in .NET apps. |
| **MicroPython** | Stripped-down for microcontrollers. | Embedded / IoT. Subset of the stdlib. |

Practical consequences of the split:

- **The GIL is a CPython thing.** Jython and IronPython have no GIL (and never did). PyPy has one. GraalPy has one too — kept for C-extension compatibility with CPython's C API; the project aims to drop it as CPython's PEP 703 (free-threading) rolls out and ecosystem packages catch up. Most "Python concurrency" advice you read is implicitly CPython-specific.
- **Bytecode (`.pyc` files)** is CPython-specific. Other implementations have their own internal forms.
- **C extensions** (`.so` / `.pyd` built against `Python.h`) target CPython directly. PyPy supports many via `cpyext` (often slower); Jython / IronPython generally do not.
- **`int` arbitrary precision, the `str`/`bytes` split, generators, decorators** — all language-level. Every conforming implementation has them.
- **"Python is slow"** is almost always a CPython performance statement. PyPy and GraalPy close most of the gap for compute-bound code.

> ☕ **Java parallel:** "Python" is to "CPython" as "Java SE" is to "OpenJDK" — a spec and a default implementation that people treat as synonymous, with alternatives (OpenJ9, Azul, GraalVM) for specific contexts. For a Java dev, **Jython** and **GraalPy** are the bridge: both let you run Python *inside* a JVM application and call between the two languages at runtime.

## Entry point

In Java, execution starts from `main()`. In Python, top-level code runs immediately when the file is imported or executed.

To make a file behave like a script only when run directly — and not run that code if it's imported as a module — use the idiom:

```python
if __name__ == "__main__":
    print("Run only when this file is executed directly")
```

`__name__` is set to `"__main__"` when the file is the entry point, and to the module's name otherwise. Treat the `if __name__ == "__main__":` block as "this is the `main` method."

## Operators

Most arithmetic and logical operators look familiar, but a handful behave differently or don't exist at all in Java.

### Arithmetic

| Operator | Meaning | Java analog |
| :--- | :--- | :--- |
| `+`, `-`, `*` | as in Java | same |
| `/` | **always** float division | `(double)a / b` |
| `//` | floor division (`int` when both operands are `int`) | `a / b` for ints |
| `%` | modulo — **sign follows divisor** | `Math.floorMod(a, b)` |
| `**` | exponentiation | `Math.pow(a, b)` |

```python
print(7 / 2)       # >>> 3.5      (NOT integer division)
print(7 // 2)      # >>> 3        (floor division)
print(7 % 2)       # >>> 1
print(2 ** 10)     # >>> 1024
```

> ⚠️ **Pitfall:** Two arithmetic differences regularly bite Java devs:
> (1) **`/` is always float division** — `7/2 == 3.5`, not 3. Use `//` for integer division. The Java mistake in reverse: `1/2` in Java is 0, in Python is 0.5.
> (2) **`%` sign follows the divisor.** Python: `-7 % 3 == 2`. Java: `-7 % 3 == -1`. The Java analog of Python's behavior is `Math.floorMod(-7, 3)`. Hash bucketing and clock arithmetic ports break on this.

### Integers are unbounded

There is no `long`. There is no `BigInteger`. There is only `int` — and it's arbitrary precision out of the box. No overflow, no wraparound, no class-import dance.

```python
n = 2 ** 1000                           # arithmetic on a 1000-bit int — fine
print(len(str(n)))                      # >>> 302    (number of decimal digits)

big = 10 ** 100
print(big * big)                        # 200-digit number, no overflow
```

> ☕ **Java parallel:** Java distinguishes `int` (32-bit), `long` (64-bit), and `BigInteger` (heap-allocated, arbitrary precision). Python has just `int` — always a `PyLong` object internally, with a variable-length array of "digits" sized to fit the value. There is no primitive-int fast path in the interpreter; small values just use one internal digit, big values use more. `BigInteger`-style code (`a.multiply(b)`) just becomes `a * b`.

The trade-off: every Python int allocation has object overhead, and arithmetic on huge values scales with digit count. For tight numerical loops where you know a value fits in 64 bits, use NumPy arrays of `int64` ([Part 6 § Numerical and data libs](06_ecosystem_and_packaging.md#numerical-and-data-libs)) — that gets you machine-int speed plus vectorization. `sys.maxsize` exists but it's the `Py_ssize_t` maximum (platform-dependent — typically `2**63 - 1` on 64-bit systems), used for sizing things like list lengths; it is NOT a limit on `int` values.

### No `++` or `--`

Python has **no increment/decrement operators**. Use compound assignment:

```python
x = 0
x += 1                  # NOT x++
x -= 1                  # NOT x--
```

This is intentional — Python prefers statements that don't double as expressions.

### Bitwise

Same set as Java on `int`: `&` (AND), `|` (OR), `^` (XOR), `~` (NOT), `<<` (left shift), `>>` (right shift). Python ints are arbitrary precision, so shifts never overflow.

```python
print(0b1100 & 0b1010)  # >>> 8     (0b1000)
print(0b1100 | 0b1010)  # >>> 14    (0b1110)
print(0b1100 ^ 0b1010)  # >>> 6     (0b0110)
print(~0)               # >>> -1
print(1 << 5)           # >>> 32
```

> ⚠️ **Pitfall:** `**` is NOT XOR — it's exponentiation. XOR is `^` (same as Java). And `&` / `|` on `bool` values are **eager** (both sides evaluated, because `bool` is a subclass of `int`). Use `and` / `or` for short-circuit boolean logic — `False and expensive()` skips the call; `False & expensive()` does not.

### Comparison chaining

Python lets comparison operators chain — no Java analog:

```python
x = 5
print(0 < x < 10)        # >>> True       (same as: 0 < x and x < 10)
print(0 < x == 5)        # >>> True
```

Each variable is evaluated once. Idiomatic for range checks; don't overuse for compound conditions where `and` reads better.

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

**Ternary expression** — Python has the equivalent of Java's `cond ? a : b`, with unusual word order:

```python
age = 21
status = "adult" if age >= 18 else "minor"
print(status)        # >>> adult
```

Read it as "**value-if-true** `if` **condition** `else` **value-if-false**." Java devs almost always write it backwards the first few times.

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

**`id()` is what `is` actually compares.** `a is b` is equivalent to `id(a) == id(b)`. `id()` returns an integer that uniquely identifies an object for its lifetime — useful for tracing aliasing and debugging:

```python
a = "I love to learn"
print(id(a))                    # >>> 4402585072  (your number will differ)

b = a
print(id(b) == id(a))           # >>> True       (same object, b is just an alias)

c = [1, 2]
d = [1, 2]
print(id(c) == id(d))           # >>> False      (different objects, equal contents)
```

In CPython, `id()` happens to return the object's memory address — but that's a CPython implementation detail, not a language guarantee. PyPy, Jython, and GraalPy return some unique opaque integer that is *not* an address. Treat the value as an identifier valid only for the object's lifetime: once an object is garbage-collected, the same `id()` can be reused for a different object. Java has no equivalent — the JVM deliberately hides object addresses (`System.identityHashCode(o)` is the closest thing, but it's a hash, not an identity).

> ⚠️ **Pitfall:** Don't write `x is 3` or `x is "hello"`. CPython interns small ints and short string literals, so it may *appear* to work — but `is` checks identity, not value, so the behavior depends on interpreter internals. Python 3.8+ raises a `SyntaxWarning` for literals after `is`. Reserve `is` for `None`, `True`, `False`, and explicit sentinels; use `==` for everything else.

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

`zip(a, b, ...)` pairs up elements from two or more iterables, producing a lazy iterator of tuples:

```python
x = [1, 2, 3]
y = [4, 5, 6]

zipped = zip(x, y)
print(list(zipped))               # >>> [(1, 4), (2, 5), (3, 6)]
```

Wrap in `list()` to materialise (the iterator is single-use — once exhausted it stays empty). Most of the time you don't materialise; you iterate directly and destructure each tuple:

```python
x = [1, 2, 3]
y = [4, 5, 6]
for a, b in zip(x, y):
    print(a, b)
# >>> 1 4
# >>> 2 5
# >>> 3 6
```

By default `zip` **stops at the shortest** input, silently dropping trailing items from longer iterables. Use `zip(x, y, strict=True)` (3.10+) to raise `ValueError` on length mismatch, or `itertools.zip_longest(...)` to pad with a fill value.

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

**String literals** — single and double quotes are equivalent. Triple quotes (`"""..."""` or `'''...'''`) preserve newlines literally — useful for multi-line strings, docstrings, SQL queries, and JSON templates:

```python
single = 'one'
double = "one"                  # identical to single

multi = """\
SELECT id, name
FROM users
WHERE active = true
"""
```

The leading `\` after the opening `"""` suppresses the first newline so the string starts at `SELECT`. Triple-quoted strings are also the standard form for docstrings (the string immediately following a `def` / `class`).

**Breaking a long string across lines without newlines or indentation in the value.** Adjacent string literals are concatenated by the parser — no `+` operator needed:

```python
words = "desert" "you"
print(words)                       # >>> desertyou
```

To span lines, wrap the literals in parentheses (implied line continuation works inside `()`, `[]`, and `{}`):

```python
lyrics = ("Never gonna give you up, "
          "Never gonna let you down, "
          "Never gonna run around and desert you.")
print(lyrics)
# >>> Never gonna give you up, Never gonna let you down, Never gonna run around and desert you.
```

Unlike a triple-quoted string, this produces a single logical line — none of the source-code newlines or leading-space indentation ends up in the value. Use this for long error messages, SQL fragments, or URLs you want to keep readable in code without affecting runtime content. Use triple-quoted strings when you genuinely want every newline and space preserved (SQL with formatting, docstrings, templates). Java 15+'s text blocks (`"""..."""`) cover roughly the same ground as Python's triple-quoted form — but Java has no no-operator adjacent-literal equivalent: the JVM uses `+` (folded at compile time when both sides are constant literals) or `String.join`.

> ⚠️ **Pitfall:** Implicit concatenation joins only **literal** strings, not variables — `name "suffix"` is a `SyntaxError`. The classic real-world bite is a list of strings with a missing comma: `["foo", "bar" "baz"]` silently becomes `["foo", "barbaz"]` because `"bar" "baz"` merged into one literal. Linters (`ruff` rule `ISC001`, `pylint` `implicit-str-concat`) catch this.

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

## Console I/O

Two functions cover almost all stdin/stdout work.

### `input()` — read a line from stdin

```python
name = input("Name: ")          # prints the prompt, reads a line of text
print(f"Hello, {name}")
```

`input()` **always returns `str`**. For numbers, convert explicitly:

```python
age = int(input("Age: "))
temp = float(input("Temp: "))
```

> ⚠️ **Pitfall:** Java `Scanner.nextInt()` does the type conversion for you; Python `input()` does not. `int(input("n: ")) + 1` works; `input("n: ") + 1` raises `TypeError: can only concatenate str (not "int") to str`.

### `print()` keyword arguments

Beyond multi-arg printing, `print` takes useful keyword arguments:

```python
print("a", "b", "c")                    # >>> a b c           (default sep=' ')
print("a", "b", "c", sep=" | ")         # >>> a | b | c

print("loading", end="")                # no trailing newline
print(".", end=""); print(".", end="")
print(" done")                          # >>> loading.. done

import sys
print("error", file=sys.stderr)         # write to stderr instead of stdout
```

`sep` goes between args; `end` goes after the whole call (default `"\n"`); `file` redirects output. The combination replaces what would be a `StringBuilder` + `System.out.println` in Java.

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

**Catching multiple exception types** — pass a **tuple** to `except`; one block handles any of them:

```python
user_input = "0"
try:
    value = int(user_input)
    result = 100 / value
except (ValueError, ZeroDivisionError) as e:
    print(f"Bad input: {e}")               # >>> Bad input: division by zero
```

The parentheses are required — `except ValueError, ZeroDivisionError:` (no parens) is a syntax error in Python 3. The `as e` clause is optional; use it when you need the exception object.

You can also stack distinct `except` clauses when each type needs different handling:

```python
import json, logging
log = logging.getLogger(__name__)

def load_config(path):
    try:
        with open(path, "r") as f:         # OSError if path is missing/unreadable
            raw = f.read()
        return json.loads(raw)             # JSONDecodeError if contents are malformed
    except json.JSONDecodeError:
        log.warning("malformed json, skipping")
        return {}
    except OSError as exc:
        log.error("io failure: %s", exc)
        raise
```

Clauses are tried top-to-bottom; the first matching type wins. Order from **most specific to most general** — `except Exception:` at the top would shadow everything below it.

> ⚠️ **Pitfall:** The `as <name>` binding is **deleted at the end of the `except` block**, even on the success path. CPython does an implicit `del exc` to break a reference cycle between the exception, its traceback, and the local frame. So `exc` is unbound (not `None`) after the block — touching it raises `NameError`. If you need the value afterward, copy it to a variable defined *outside* the `except`: `err = None; try: ... except OSError as exc: err = exc`. Java's `catch (X e)` has no such cleanup; the variable just goes out of `catch` scope normally.

> ☕ **Java parallel:** Python's `except (A, B):` is Java's `catch (A | B e)` (multi-catch, Java 7+). Stacked `except` clauses are stacked `catch` blocks. Same precedence rule: most specific first.

**Raising and custom exceptions:**

```python
def withdraw(balance, amount):
    if amount > balance:
        raise ValueError("Insufficient funds")
    return balance - amount


class InvalidUserInputError(Exception):
    pass
```

**Custom exceptions with fields.** Exceptions are just classes — they can carry any state, not only the message. Add fields via `__init__` exactly like any other class:

```python
class HttpError(Exception):
    def __init__(self, status_code: int, message: str, headers: dict | None = None):
        super().__init__(message)            # what str(exc) shows
        self.status_code = status_code
        self.headers = headers or {}

try:
    raise HttpError(503, "service unavailable", headers={"Retry-After": "30"})
except HttpError as e:
    print(e.status_code)                     # >>> 503
    print(e.headers["Retry-After"])          # >>> 30
    print(e)                                 # >>> service unavailable
```

> 💡 **Pythonic:** Always pass the human-readable message to `super().__init__(message)` — that's what `str(exc)`, logging, and tracebacks display, and what ends up in `exc.args[0]`. Structured fields (`status_code`, `headers`, anything) go on `self` as plain attributes.

> ☕ **Java parallel:** Same shape as a Java exception subclass with extra fields and `super(message)` in the constructor. The only Python-specific quirk is `exc.args` — a tuple of whatever was passed to `super().__init__(...)`. Most apps just put the message there and access structured fields via the named attributes.

### Exception hierarchy

Every Python exception ultimately inherits from **`BaseException`**, not `Exception`. The two are different on purpose:

```
BaseException
├── SystemExit              ← raised by sys.exit()
├── KeyboardInterrupt       ← raised on Ctrl-C
├── GeneratorExit           ← raised when a generator is closed
├── BaseExceptionGroup *    ← group wrapper that may carry "don't catch" types (🐍 3.11+)
└── Exception               ← the base for everything app code should catch or subclass
    ├── ExceptionGroup *    ← group whose members are all Exception subclasses (🐍 3.11+)
    ├── ValueError
    ├── TypeError
    ├── RuntimeError
    ├── OSError
    └── ... (almost every other built-in error)
```

\* `ExceptionGroup` actually *multi-inherits* from both `BaseExceptionGroup` and `Exception` — `class ExceptionGroup(BaseExceptionGroup, Exception)`. It's drawn under `Exception` because that's the side that decides `except Exception:` catches it. The `BaseExceptionGroup` parent only matters when the group could carry a `KeyboardInterrupt` or other non-`Exception` member.

`SystemExit`, `KeyboardInterrupt`, `GeneratorExit`, and `BaseExceptionGroup` sit *outside* `Exception` deliberately — they signal "the program is being torn down" or "this group might contain a tear-down signal," not "a bug happened." Writing `except Exception:` correctly lets those signals propagate so Ctrl-C still exits and `sys.exit()` still works. See [Part 4 § Exception groups](04_concurrency.md#exception-groups) for the `except*` syntax that unpacks them.

> ⚠️ **Pitfall:** Never write bare `except:` or `except BaseException:` in application code. Both swallow `KeyboardInterrupt` and `SystemExit` — your program will refuse to die on Ctrl-C and `sys.exit()` calls will be eaten. Always use `except Exception:` (or something more specific). Bare `except:` is almost always a bug.

**Two rules of thumb:**
- **Catch:** `except Exception:` as your widest net — never wider.
- **Subclass:** `class MyError(Exception):` — never inherit from `BaseException` directly.

> ☕ **Java parallel:** `BaseException ≈ Throwable`. Python's `Exception` ≈ Java's `Exception` (the catchable branch). Python has no checked/unchecked split, so application errors behave operationally like Java's unchecked `RuntimeException` — callers aren't forced to declare or handle them. The "stuff outside `Exception`" carve-out exists for the same reason Java's `Error` branch does: signal "don't catch this in normal code."

**Chaining causes** (analogous to Java `throw new X(msg, cause)`):

```python
try:
    int("not a number")
except ValueError as e:
    raise RuntimeError("Could not parse user input") from e
```

The `from e` preserves the original exception in `__cause__` and the traceback shows both — exactly what `getCause()` gives you in Java.

**Suppressing the chain** with `raise ... from None`. When you `raise` inside an `except` block, Python *implicitly* chains the new exception to the one being handled (shown in tracebacks as `During handling of the above exception, another exception occurred`). To hide that chain — usually when translating an internal error into a clean public API error — use `from None`:

```python
_USERS = {"alice": {"id": 1}, "bob": {"id": 2}}

def get_user(name):
    try:
        return _USERS[name]
    except KeyError:
        raise LookupError(f"No user named {name!r}") from None    # no internal KeyError in the traceback

get_user("carol")
# Traceback (most recent call last):
#   ...
# LookupError: No user named 'carol'
```

Without `from None`, the traceback leaks the underlying `KeyError` to your caller. Java doesn't need this because it doesn't auto-chain — omitting `cause` from the constructor is enough.

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

## Python-specific keywords

Reference table of keywords that surprise Java devs. Each links to the deep treatment.

| Keyword | What surprises Java devs | Deep treatment |
| :--- | :--- | :--- |
| `pass` | Empty-body placeholder — Java uses `{}`. Required where syntax wants a body. | [§ Control flow](#control-flow) |
| `elif` | One word, not `else if`. | [§ Control flow](#control-flow) |
| `and`, `or`, `not` | Not `&&`, `\|\|`, `!`. Short-circuit. | [§ Control flow](#control-flow) |
| `is`, `is not` | Identity comparison (≈ Java `==` on references). Reserve for `None` / sentinels. | [§ None and is](#none-and-is) |
| `in`, `not in` | Membership test — Java needs `.contains()`. | [Part 3 § Iterable vs iterator](03_pythonic_idioms.md#iterable-vs-iterator) |
| `assert` | **Opt-out** instead of opt-in. Java `assert` is OFF by default (enabled with `-ea`); Python `assert` is ON by default (stripped with `-O`). Same "don't use for input validation" rule applies. | [Part 3 § Advanced control flow](03_pythonic_idioms.md#advanced-control-flow) |
| `del` | Remove a binding, list element, dict key, or attribute. | [Part 3 § Advanced control flow](03_pythonic_idioms.md#advanced-control-flow) |
| `for ... else`, `while ... else` | `else` runs if the loop did NOT `break`. **No Java analog.** | [Part 3 § Advanced control flow](03_pythonic_idioms.md#advanced-control-flow) |
| `with` | Like `try-with-resources` but for ANY context manager (locks, transactions, timing). | [Part 3 § Context managers](03_pythonic_idioms.md#context-managers) |
| `lambda` | Anonymous single-expression function — familiar from Java 8+, but Python prefers `def` for non-trivial bodies. | [Part 3 § Functions as first-class objects](03_pythonic_idioms.md#functions-as-first-class-objects) |
| `yield` | Turns the function into a generator. No Java keyword analog. | [Part 3 § Generators](03_pythonic_idioms.md#generators) |
| `yield from` | Delegate iteration to a sub-generator or iterable (forwards values, exceptions, and `.send()`). | [Part 3 § Generators](03_pythonic_idioms.md#generators) |
| `nonlocal` | Rebind an enclosing-function variable. Java lambdas can't — captured vars are effectively final. | [Part 3 § Scope and nonlocal](03_pythonic_idioms.md#scope-and-nonlocal) |
| `global` | Rebind a module-level variable. Smell if you reach for it often. | [Part 3 § Scope and nonlocal](03_pythonic_idioms.md#scope-and-nonlocal) |
| `async`, `await` | Coroutine declaration / pause-and-resume. | [Part 4 § Async and await](04_concurrency.md#async-and-await) |
| `match`, `case` | Structural pattern matching (3.10+). NOT Java `switch`. | [§ Match](#match), [Part 3 § Match patterns](03_pythonic_idioms.md#match-patterns) |
| `raise ... from ...` | Exception chaining — like Java `throw new X(msg, cause)`. | [§ Exception handling](#exception-handling) |
| `except*` | Handle `ExceptionGroup` branches (🐍 3.11+). | [Part 4 § Exception groups](04_concurrency.md#exception-groups) |

> ⚠️ **Top 3 surprises for a Java reader:**
> (1) **`for / while ... else`** — `else` runs only if the loop completed without `break`. Read it as `nobreak:`.
> (2) **`nonlocal`** — Java lambdas can't reassign captured variables; Python `nonlocal` can.
> (3) **`is`** — identity, not value. Use ONLY for `None`, `True`, `False`, and explicit sentinels. `x is "hello"` triggers a `SyntaxWarning`.

`break` and `continue` themselves work exactly like Java — they're not on this list. The Python twist is the `else` clause that can follow the loop, which interacts with `break`. See [Part 3 § Advanced control flow](03_pythonic_idioms.md#advanced-control-flow) for the `for/while ... else` deep treatment.

## Key Takeaways

- Indentation defines blocks — no braces.
- `None` is an object; compare with `is`, not `==`.
- Truthiness is a feature: `if items:` is idiomatic for "non-empty".
- `str` is Unicode text; `bytes` is raw bytes. Encode/decode only at I/O boundaries.
- `match` is structural pattern matching, not Java `switch` — no fall-through, `,` means tuple, use `|` for OR.
- `if __name__ == "__main__":` for script-only execution.
- Python has no checked exceptions — discipline matters more.
- `/` is always float division; `//` is floor division; `%` sign follows the divisor; no `++`/`--`; `**` is exponentiation (not XOR).
- `int` is unbounded — no separate `long`/`BigInteger`. `2**1000` just works.
- Ternary reads "value-if-true `if` cond `else` value-if-false."
- `input()` returns `str`; cast for numbers. `print(*, sep=, end=, file=)` covers most output needs.
