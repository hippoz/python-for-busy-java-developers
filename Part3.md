# Python Language Learning for Busy Java Developers, Part 3

## 1. Core Differences at a Glance

| Topic                    | Java View                                            | Python View                                              |
| :----------------------- | :--------------------------------------------------- | :------------------------------------------------------- |
| **Exception handling**   | Checked + unchecked exceptions                       | No checked exceptions; simpler but easier to misuse      |
| **Comprehensions**       | Often loop or Stream based                           | Native literal-style syntax for transforming collections |
| **`*args` / `kwargs`**   | Method overloading / varargs / builder-like patterns | Flexible function signatures and argument unpacking      |
| **Iterable vs Iterator** | Explicit interfaces                                  | Protocol-based, built into `for` and `next()`            |
| **Mutable vs immutable** | Mixed, but less visible in syntax                    | A major everyday language distinction                    |
| **Truthiness**           | Conditions must be boolean                           | Many values are implicitly truthy/falsy                  |
| **`is` vs `==`**         | Reference equality vs logical equality               | Similar idea, but used more often in practice            |
| **Modules and imports**  | Package/class based organization                     | File/module oriented import model                        |
| **`dataclass`**          | Similar role to POJO / record                        | Lightweight class generation built into the language     |
| **Context manager**      | `try-with-resources`                                 | General-purpose `with` protocol, not just files          |

### Minimal comparison

**Java**
```java
List<Integer> squares = new ArrayList<>();
for (int n : List.of(1, 2, 3)) {
    squares.add(n * n);
}
```

**Python**
```python
squares = [n * n for n in [1, 2, 3]]
```

## 2. Exception Handling

### `try / except / else / finally`

Python exception handling is more lightweight than Java's.

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

Key difference for Java developers:
- Python has **no checked exceptions**.
- You are not forced to declare or catch exceptions.
- This reduces ceremony, but it also means discipline matters more.

### Raising exceptions

```python
def withdraw(balance, amount):
    if amount > balance:
        raise ValueError("Insufficient funds")
    return balance - amount
```

### Custom exceptions

```python
class InvalidUserInputError(Exception):
    pass
```

### Mental model for Java developers

- `except` is Python's `catch`
- `raise` is Python's `throw`
- no checked exceptions means API boundaries are simpler, but less explicit

## 3. Comprehensions

### List comprehension

A list comprehension is a compact way to build a list.

```python
numbers = [1, 2, 3, 4]
squares = [n * n for n in numbers]
print(squares)
```

### Filter inside comprehension

```python
evens = [n for n in numbers if n % 2 == 0]
print(evens)
```

### Dictionary comprehension

```python
squares_map = {n: n * n for n in numbers}
print(squares_map)
```

### Set comprehension

```python
lengths = {len(word) for word in ["cat", "dog", "apple"]}
print(lengths)
```

### Java comparison

Comprehensions often replace:
- explicit loops,
- temporary collections,
- simple Stream pipelines.

Use them when the transformation is clear and short.
If the logic becomes too complex, prefer a normal loop for readability.

## 4. Function Arguments and Unpacking

### `*args`

Use `*args` when a function should accept a variable number of positional arguments.

```python
def add_all(*args):
    return sum(args)

print(add_all(1, 2, 3, 4))
```

### `**kwargs`

Use `**kwargs` when a function should accept arbitrary named arguments.

```python
def print_user(**kwargs):
    for key, value in kwargs.items():
        print(key, value)

print_user(name="Alice", age=25)
```

### Argument unpacking

```python
values = [3, 4]
print(pow(*values))
```

```python
user = {"name": "Alice", "age": 25}
print_user(**user)
```

### Star unpacking in assignment

```python
first, *rest = [1, 2, 3, 4]
print(first)
print(rest)
```

### Java comparison

This replaces many Java habits such as:
- method overloading,
- varargs-only APIs,
- map/builder-style parameter passing.

## 5. Iterable vs Iterator

### Iterable

An iterable is something you can loop over.
Examples:
- list
- tuple
- set
- dict
- string
- generator

### Iterator

An iterator is an object that produces values one at a time and remembers where it is.

```python
numbers = [10, 20, 30]
it = iter(numbers)

print(next(it))
print(next(it))
print(next(it))
```

### Why this matters

A `for` loop in Python is really doing this under the hood:
- call `iter(...)`
- repeatedly call `next(...)`
- stop at `StopIteration`

### Java comparison

Very similar in spirit to `Iterable` and `Iterator`, but Python relies more on protocols than explicit interface declarations.

## 6. Mutable vs Immutable Objects

This is one of the most important Python day-to-day topics.

### Common immutable types
- `int`
- `float`
- `bool`
- `str`
- `tuple`

### Common mutable types
- `list`
- `dict`
- `set`

### Example

```python
x = [1, 2]
y = x
y.append(3)
print(x)
```

Both names point to the same mutable list.

### Common pitfall: mutable default arguments

```python
def add_item(item, items=[]):
    items.append(item)
    return items
```

This is dangerous because the same list is reused across calls.

Safer version:

```python
def add_item(item, items=None):
    if items is None:
        items = []
    items.append(item)
    return items
```

### Java comparison

Java developers also deal with references, but Python makes this more visible because variable binding is so lightweight and collections are used everywhere.

## 7. Truthiness

In Java, `if (...)` requires a boolean expression.
In Python, many objects can be used directly in conditionals.

Falsy values include:
- `None`
- `False`
- `0`
- `0.0`
- `""`
- `[]`
- `{}`
- `set()`

```python
items = []
if not items:
    print("Empty")
```

This is very Pythonic, but Java developers need to get used to it.

## 8. `is` vs `==`

This distinction is essential.

- `==` checks **value equality**
- `is` checks **object identity**

```python
a = [1, 2]
b = [1, 2]

print(a == b)
print(a is b)
```

Use `is` mainly for:
- `None`
- singleton-style sentinels

```python
value = None
if value is None:
    print("No value")
```

Java comparison:
- roughly like `.equals()` vs `==` on object references

## 9. Modules, Packages, and Imports

In Python, a file is a module.
A folder can be a package.

### Basic imports

```python
import math
print(math.sqrt(16))
```

```python
from math import sqrt
print(sqrt(16))
```

```python
import pandas as pd
```

### Why this feels different from Java

Java developers often think in terms of:
- packages,
- classes,
- static imports.

Python import structure is more file-oriented and direct.
That makes it flexible, but circular imports and module-level side effects are common pitfalls.

## 10. `dataclass`

If `dict` is too loose and a full class is too verbose, `dataclass` is often the sweet spot.

```python
from dataclasses import dataclass

@dataclass
class User:
    name: str
    age: int
```

This automatically gives you:
- `__init__`
- `__repr__`
- `__eq__`

### Java comparison

This is conceptually close to:
- a lightweight POJO,
- or even a Java `record` in simple cases.

## 11. Context Managers Beyond Files

You already saw `with open(...)`, but `with` is a general language feature.

A context manager controls setup and cleanup around a block.

```python
class Demo:
    def __enter__(self):
        print("enter")
        return self

    def __exit__(self, exc_type, exc, tb):
        print("exit")

with Demo():
    print("inside")
```

This is broader than Java's `try-with-resources` because Python can use the same pattern for many types of scoped behavior, not just closable resources.

## 12. Practical Takeaways for Java Developers

- Learn comprehensions early; they are everywhere.
- Use `*args` / `**kwargs` to read Python APIs comfortably.
- Understand iterable vs iterator to truly understand `for`, generators, and lazy evaluation.
- Never ignore mutable vs immutable behavior.
- Use `is` for `None`, not `==`.
- Treat truthiness as a language feature, not a hack.
- Reach for `dataclass` when a raw `dict` is too loose and a hand-written class is too verbose.
- Remember that Python's import system is simpler on the surface, but can still create circular dependency problems.

## 13. Final Summary

Part 3 focuses on the language features that make Python feel truly different from Java in everyday coding:

- exception handling is lighter,
- collection transformation is more built-in,
- function signatures are more flexible,
- iteration is protocol-based,
- mutability rules matter constantly,
- object identity and truthiness are part of daily reasoning,
- and `dataclass` plus context managers show how Python reduces boilerplate without losing expressiveness.

For a busy Java developer, these are the features that most often separate "Python syntax familiarity" from actually writing Pythonic code.
