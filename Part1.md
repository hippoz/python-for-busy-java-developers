# Python Language Learning for Busy Java Developers, Part 1

## 1. Core Differences at a Glance

| Topic | Java | Python |
| :--- | :--- | :--- |
| **Typing** | Statically typed | Dynamically typed |
| **Syntax** | Braces, semicolons, more boilerplate | Indentation-based, concise |
| **Entry point** | `main()` required | Top-level code runs directly |
| **Paradigm** | Primarily OOP | Multi-paradigm |
| **Performance** | Faster, JIT on JVM | Slower, optimized for productivity |
| **Typical strengths** | Enterprise, backend, Android | Automation, scripting, AI/ML, data |

### Minimal comparison

**Java**
```java
public class Main {
    public static void main(String[] args) {
        int a = 5;
        int b = 3;
        System.out.println(a + b);
    }
}
```

**Python**
```python
a = 5
b = 3
print(a + b)
```

## 2. Syntax, Functions, and Execution Model

### Static vs dynamic typing

**Java** requires explicit types:

```java
int age = 25;
String name = "Alice";
```

**Python** infers types at runtime:

```python
age = 25
name = "Alice"
```

Python also supports optional type hints:

```python
def add(a: int, b: int) -> int:
    return a + b
```

### Functions and multiple return values

Python functions are lightweight and can return multiple values naturally via tuples.

```python
def process_numbers(a, b):
    return a + b, a * b

sum_val, prod_val = process_numbers(5, 3)
```

### Execution model

- **Java**: source code -> bytecode -> JVM -> JIT optimization.
- **Python**: interpreted execution model, though it also compiles to bytecode internally.

### Entry point

In Java, execution starts from `main()`. In Python, top-level code runs immediately.
To make a file behave like a script only when executed directly, use:

```python
if __name__ == "__main__":
    print("Run only when this file is executed directly")
```

## 3. Built-in Data Structures

### 3.1 Dictionary (`dict`) vs `HashMap`

Python dictionaries are the closest equivalent to Java `HashMap`.

```python
user_ages = {
    "Alice": 25,
    "Bob": 30,
}

print(user_ages["Alice"])
```


#### Is `dict` more like `HashMap` or `TreeMap`?

For a Java developer, Python `dict` is much closer to **`HashMap`** than to `TreeMap`.

Why:
- lookup is hash-based,
- keys are not sorted by natural order,
- equality and hashing matter for key behavior,
- average-case access is designed to be very fast.

Important nuance:
- modern Python `dict` **preserves insertion order**,
- but that does **not** make it a `TreeMap`.
- insertion order means keys come back in the order they were added, not sorted order.

```python
data = {"b": 2, "a": 1, "c": 3}
print(data)
```

This preserves insertion order, not alphabetical order.

So the best mental model is:
- `dict` = **ordered hash map**
- not a tree-based map
- not automatically sorted

If you need sorted behavior, you usually do it explicitly:

```python
data = {"b": 2, "a": 1, "c": 3}
for key in sorted(data):
    print(key, data[key])
```

### 3.1.1 Set vs `HashSet` / `TreeSet`

Python `set` is also much closer to **`HashSet`** than to `TreeSet`.

Why:
- membership is hash-based,
- elements are unique,
- iteration order is not sorted order,
- elements must be hashable.

```python
values = {"b", "a", "c"}
print(values)
```

Again, this is not sorted like a `TreeSet`.

So the best mental model is:
- `set` = **hash-based set**
- `frozenset` = immutable hash-based set
- neither is tree-based or automatically sorted

If you need sorted output, do it explicitly:

```python
values = {"b", "a", "c"}
print(sorted(values))
```

### When would this matter?

This matters when you are translating Java code mentally:
- `dict` is usually **not** `TreeMap`
- `set` is usually **not** `TreeSet`
- if you need sorted collections in Python, the common pattern is often:
  - store data in `dict` / `set`
  - sort when reading or presenting it

This is a very Pythonic design choice: keep the core collection fast and simple, and sort only when necessary.

#### Iterating over a dictionary

By default, `for` iterates over keys:

```python
height_dict = {"Tom": 163, "Dick": 178, "Harry": 182}

for key in height_dict:
    print(f"{key}'s height is {height_dict[key]}")
```

To iterate over both keys and values, use `.items()`:

```python
for key, value in height_dict.items():
    print(f"{key}'s height is {value}")
```

### 3.2 Tuple

A tuple is like a list, but **immutable**.

```python
x = (1, 2, 3, 4, 5)
print(x[0])
print(x[1:3])
print(x[-1])
print(x[::-1])
```

#### Single-element tuple quirk

A one-element tuple needs a trailing comma:

```python
non_tuple = (1)
singleton = (1,)

print(type(non_tuple))
print(type(singleton))
```

#### Convert list to tuple

```python
my_list = [1, 3, 4]
my_tuple = tuple(my_list)
print(my_tuple)
```

#### Tuple immutability

```python
x = (1, 2, 3)
# x[1] = 6      # TypeError
# x.append(6)   # AttributeError
```

### 3.3 Set

A set is like Java `HashSet`: unordered and duplicate-free.

```python
vowels = {"i", "o", "a", "a", "e", "u"}
print(vowels)
```

Use `set()` to remove duplicates from a sequence:

```python
words = ["let", "it", "go", "let", "it", "go"]
vocabulary = set(words)
print(vocabulary)
```

#### Empty set quirk

```python
empty_set = set()
empty_dict = {}

print(type(empty_set))
print(type(empty_dict))
```


### 3.4 `frozenset`

A `frozenset` is the immutable version of `set`.
Think of it as roughly analogous to an unmodifiable set in Java.

```python
roles = frozenset(["admin", "editor", "viewer"])
print(roles)
```

Why this matters:
- it cannot be modified after creation,
- it can be used as a dictionary key or as an element inside another set,
- normal `set` cannot do that because it is mutable.

```python
permissions = {
    frozenset(["read", "write"]): "editor",
    frozenset(["read"]): "viewer",
}
print(permissions)
```

### 3.5 Binary data: `bytes`, `bytearray`, and `memoryview`

These types matter when working with:
- files,
- network protocols,
- images,
- compressed data,
- binary APIs.

For a Java developer, think of them as roughly related to:
- `byte[]`
- mutable byte buffers
- zero-copy views over binary memory

#### `bytes`

`bytes` is an immutable sequence of byte values.
It is similar to an immutable `byte[]`.

```python
payload = b"hello"
print(payload)
print(payload[0])
```

You often see `bytes` when reading files in binary mode or receiving network data.

```python
with open("image.png", "rb") as file:
    data = file.read()
    print(type(data))
```

#### `bytearray`

`bytearray` is the mutable version of `bytes`.
Use it when you need to modify binary data in place.

```python
buffer = bytearray(b"hello")
buffer[0] = 72
print(buffer)
```

Java comparison:
- `bytes` is closer to read-only binary data,
- `bytearray` is closer to a mutable byte buffer.

#### `memoryview`

`memoryview` lets you create a view over binary data **without copying it**.
This is important for performance when working with large binary buffers.

```python
data = bytearray(b"abcdef")
view = memoryview(data)

print(view[0])
view[1] = 90
print(data)
```

Why this matters:
- avoids unnecessary copying,
- useful in performance-sensitive I/O code,
- especially relevant when slicing or passing binary buffers around.

A good mental model is:
- `bytes` = immutable binary data
- `bytearray` = mutable binary data
- `memoryview` = window onto existing binary data without copying


### 3.6 `range`

`range` represents a sequence of integers, but it is **not** a real list.
It is a lightweight, lazy range object commonly used in loops.

```python
numbers = range(1, 6)
print(numbers)
print(list(numbers))
```

You can iterate over it directly:

```python
for i in range(3):
    print(i)
```

Why this matters for Java developers:
- it is often used where Java developers would write a classic indexed `for` loop,
- but it behaves more like a compact sequence object than an eagerly created array or list.

```python
for i in range(0, 10, 2):
    print(i)
```

A useful mental model:
- `range(start, stop, step)` is Python's built-in way to express numeric iteration
- it is lazy and memory-efficient

### 3.7 `complex`

Python has a built-in complex number type.
This is something Java does **not** provide directly in the standard language or core library.

```python
z = 3 + 4j
print(z)
print(z.real)
print(z.imag)
```

You can also perform arithmetic directly:

```python
z1 = 1 + 2j
z2 = 3 + 4j
print(z1 + z2)
print(z1 * z2)
```

Why this matters:
- useful in mathematical, scientific, and signal-processing code,
- built directly into the language,
- no third-party class is needed for basic complex arithmetic.

A useful mental model:
- `complex` is a first-class numeric type in Python,
- similar to how Python treats `int` and `float`, but for numbers with real and imaginary parts.


### 3.8 Strings, Unicode, and Encodings

For Java developers, this is an important topic because Python makes a very clear distinction between:
- **text** (`str`)
- **binary data** (`bytes`)

In modern Python:
- `str` represents **Unicode text**
- `bytes` represents raw binary data

That means Python strings are not "UTF-8 strings" or "GBK strings" internally in the way people sometimes casually describe them. A Python `str` is text. 
The encoding only matters when you **convert between text and bytes**, such as:
- reading a file,
- writing a file,
- sending data over the network,
- decoding external binary content.

#### Unicode vs UTF-8 vs GBK

- **Unicode**: the universal character set / text model
- **UTF-8**: a very common encoding that converts Unicode text into bytes
- **GBK**: another encoding, commonly associated with Simplified Chinese text in legacy systems

A useful mental model is:
- `str` = characters
- `bytes` = encoded representation of those characters

#### Encode: `str` -> `bytes`

```python
text = "你好, Python"
utf8_data = text.encode("utf-8")
gbk_data = text.encode("gbk")

print(utf8_data)
print(gbk_data)
```

#### Decode: `bytes` -> `str`

```python
raw = b"hello"
text = raw.decode("utf-8")
print(text)
```

#### File handling with encoding

When reading or writing text files, you should specify the encoding when it matters.

```python
with open("notes_utf8.txt", "w", encoding="utf-8") as file:
    file.write("你好")

with open("notes_utf8.txt", "r", encoding="utf-8") as file:
    content = file.read()
    print(content)
```

If the wrong encoding is used, you may get:
- garbled text,
- `UnicodeDecodeError`,
- `UnicodeEncodeError`.

#### Common pitfall

```python
text = "你好"
data = text.encode("utf-8")

# Wrong: decoding UTF-8 bytes as GBK may fail or produce garbage
# broken = data.decode("gbk")
```

#### Java comparison

This is very similar in spirit to Java's distinction between:
- `String`
- `byte[]`
- `Charset` (`UTF-8`, `GBK`, etc.)

But Python makes this model extremely visible in everyday code:
- text processing -> `str`
- binary processing -> `bytes`
- transitions between them -> `.encode(...)` and `.decode(...)`

A practical rule:
- work with `str` as long as possible,
- only encode/decode at I/O boundaries.


## 4. Sequence Utilities and Built-in Functions

### `zip()`

`zip()` groups elements from multiple sequences together.

```python
x = [1, 2, 3]
y = [4, 5, 6]

for a, b in zip(x, y):
    print(a, b)
```

### `any()` and `all()`

```python
print(all([True, True, True]))
print(all([True, False, True]))
print(any([True, False, True]))
```

### `sorted()` and `reversed()`

```python
numbers = [1, 5, 3, 7, 8, 2]

print(sorted(numbers))
print(list(reversed(numbers)))
```

## 5. Control Flow and Keywords

### Common keyword differences

- `elif` instead of `else if`
- `and`, `or`, `not` instead of `&&`, `||`, `!`
- `except` instead of `catch`
- `raise` instead of `throw`
- `pass` as an empty placeholder block

### `None`

`None` is Python's equivalent of Java `null`, but it is an actual object.

```python
print(type(None))
```

Use identity comparison:

```python
value = None
if value is None:
    print("No value")
```

### Match-case (`switch`-like syntax)

Python 3.10+ supports `match ... case`.

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
```

## 6. Functions as First-Class Objects

In Python, functions can be passed around like any other value.

```python
def laugh():
    print("MUAHAHAHAHA!! :D")


def cry():
    print("WAAA!! TT_TT")


def multiplier(func, repeats):
    for _ in range(repeats):
        func()


multiplier(laugh, 2)
multiplier(cry, 1)
```

### Lambda functions

```python
square = lambda x: x * x
print(square(4))
```

### Nested functions and scope

```python
count = 0


def outer():
    def inner():
        global count
        count += 1
        print("Inside nested function!")

    inner()
```

## 7. Classes and OOP in Python

### 7.1 Basic class definition

```python
class Person:
    pass


person = Person()
```

Key ideas:
- `pass` means "do nothing".
- No `new` keyword is needed.
- Everything in Python is an object.

### 7.2 Sometimes use `dict`, not a class

If you only need a simple data container, a `dict` is often enough.

```python
user = {
    "name": "Alice",
    "age": 25,
}

print(user["name"])
```

### 7.3 Abstract classes
- **ABC**: A helper class that you inherit from to make a class "abstract".
- **@abstractmethod**: A decorator used to mark methods that _must_ be overridden by any concrete (non-abstract) subclass.
- **Enforce a Interface**: It guarantees that all subclasses follow a specific structure (e.g., every `Shape` must have an `area()` method).
- **Prevent Instantiation**: You cannot create an object directly from an abstract class; attempting to do so will raise a `TypeError`.
- **Code Reliability**: It catches missing method implementations at the time of instantiation rather than later when the code fails during execution

```python
from abc import ABC, abstractmethod


class Animal(ABC):
    @abstractmethod
    def make_sound(self):
        pass
```

### 7.4 Encapsulation the Pythonic way

Python usually keeps attributes public by default. If you need control later, use properties.

```python
class Person:
    def __init__(self, name, age):
        self.name = name
        self.age = age

    @property
    def age(self):
        return self.__age - 2

    @age.setter
    def age(self, new_age):
        if new_age < 30:
            self.__age = new_age
        else:
            print("Never! I am always under 30!")


person = Person("Josiah Wang", 20)
print(person.age)
person.age = 10
print(person.age)
person.age = 70
print(person.age)
```

### 7.5 `_protected` and `__private`

- `_name`: convention for internal/protected use only.
- `__name`: name mangling for stronger privacy.

Python does not enforce access like Java. It relies more on convention.

### 7.6 Composition over inheritance

In Python, composition is often preferred over deep inheritance trees because it reduces coupling.

## 8. Special Methods and Operators

Python classes can customize built-in behavior with dunder methods.

| Behavior | Method |
| :--- | :--- |
| constructor | `__init__` |
| string conversion | `__str__` |
| addition | `__add__` |
| equality | `__eq__` |
| length | `__len__` |
| membership | `__contains__` |
| indexing | `__getitem__` / `__setitem__` |
| iteration | `__iter__` |

### Example: vector with operator overloading

```python
class Vector:
    def __init__(self, x, y):
        self.x = x
        self.y = y

    def __add__(self, other):
        return Vector(self.x + other.x, self.y + other.y)

    def __str__(self):
        return f"Vector({self.x}, {self.y})"


v1 = Vector(1, 2)
v2 = Vector(5, 7)
v3 = v1 + v2
print(v3)
```


### Java `hashCode()`, `equals()`, `Comparable`, `toString()`, `Cloneable`, and deep clone in Python

If you are coming from Java, these are some of the most important object-model differences to understand.

#### `toString()` -> `__str__()`

Java's `toString()` maps most directly to Python's `__str__()`.
It controls what gets printed when you call `print(obj)` or `str(obj)`.

```python
class User:
    def __init__(self, name):
        self.name = name

    def __str__(self):
        return f"User(name={self.name})"


print(User("Alice"))
```

#### `equals()` -> `__eq__()`

Java's `equals()` corresponds to Python's `__eq__()`.
It defines value-based equality for `==`.

```python
class User:
    def __init__(self, name):
        self.name = name

    def __eq__(self, other):
        return isinstance(other, User) and self.name == other.name


print(User("Alice") == User("Alice"))
```

Important note:
- `==` in Python usually means value equality
- `is` means identity comparison (same object in memory)

#### `hashCode()` -> `__hash__()`

Java's `hashCode()` maps to Python's `__hash__()`.
This matters when objects are used in:
- `set`
- `frozenset`
- `dict` keys

```python
class User:
    def __init__(self, name):
        self.name = name

    def __eq__(self, other):
        return isinstance(other, User) and self.name == other.name

    def __hash__(self):
        return hash(self.name)
```

Very important rule, same spirit as Java:
- if two objects are equal, they must produce the same hash value.

Python-specific nuance:
- if you define `__eq__()` but do not define a compatible `__hash__()`, Python may make the object unhashable.
- this is Python's way of protecting you from broken equality/hash behavior.

#### `Comparable` -> rich comparison methods

Java often uses `Comparable<T>` with `compareTo(...)`.
Python usually uses rich comparison dunder methods such as:
- `__lt__()` (`<`)
- `__le__()` (`<=`)
- `__gt__()` (`>`)
- `__ge__()` (`>=`)

```python
class User:
    def __init__(self, age):
        self.age = age

    def __lt__(self, other):
        return self.age < other.age


print(User(20) < User(30))
```

A common Python pattern is:
- define `__eq__()` and `__lt__()`
- then use helpers such as `functools.total_ordering` if needed.

For sorting, Python more often prefers:
- `sorted(items, key=...)`

rather than forcing the class itself to define a universal ordering.

```python
users = [{"name": "Alice", "age": 30}, {"name": "Bob", "age": 20}]
print(sorted(users, key=lambda u: u["age"]))
```

#### `Cloneable` -> `copy` module

Python does **not** have a direct equivalent to Java's `Cloneable` interface.
Instead, copying is usually done with the `copy` module.

```python
import copy

items = [1, 2, 3]
shallow = copy.copy(items)
print(shallow)
```

This is much simpler than Java's old `Cloneable` model, which many Java developers already consider awkward.

#### Shallow copy vs deep clone

This is extremely important.

A **shallow copy** copies the outer container, but not the nested objects inside it.

```python
import copy

original = [[1, 2], [3, 4]]
shallow = copy.copy(original)
shallow[0].append(99)

print(original)
print(shallow)
```

A **deep copy** recursively copies nested objects too.

```python
import copy

original = [[1, 2], [3, 4]]
deep = copy.deepcopy(original)
deep[0].append(99)

print(original)
print(deep)
```

Java comparison:
- shallow copy is like copying the outer object but keeping the same inner references
- deep clone means recursively copying the full object graph

#### Practical mental model for Java developers

| Java | Python |
| :--- | :--- |
| `toString()` | `__str__()` |
| `equals()` | `__eq__()` |
| `hashCode()` | `__hash__()` |
| `Comparable.compareTo()` | rich comparison methods / `key=` sorting |
| `Cloneable` | `copy.copy()` / `copy.deepcopy()` |

The biggest mindset shift is:
- Python gives you smaller, more direct hooks,
- but it expects you to keep equality, hashing, ordering, and copying semantics consistent yourself.


#### `__repr__()`

Besides `__str__()`, Python also has `__repr__()`.
This is usually the more developer-oriented string representation.

A practical rule of thumb is:
- `__str__()` -> friendly output for end users
- `__repr__()` -> precise/debug-oriented output for developers

```python
class User:
    def __init__(self, name, age):
        self.name = name
        self.age = age

    def __str__(self):
        return f"User {self.name}"

    def __repr__(self):
        return f"User(name={self.name!r}, age={self.age!r})"


user = User("Alice", 25)
print(str(user))
print(repr(user))
```

For a Java developer, you can think of `__repr__()` as something like:
- a more debugging-focused `toString()`
- or the string form you wish many Java objects had by default during troubleshooting

#### `dataclass(order=True, frozen=True)`

Python's `dataclass` can automatically generate a lot of the object behavior that Java developers often write by hand.

```python
from dataclasses import dataclass

@dataclass(order=True, frozen=True)
class User:
    age: int
    name: str
```

What this gives you:
- generated `__init__()`
- generated `__repr__()`
- generated `__eq__()`
- generated ordering methods (`__lt__`, `__le__`, etc.) because of `order=True`
- immutability-like behavior because of `frozen=True`
- hashability is much easier to reason about for immutable value objects

Java comparison:
- this feels somewhat like combining:
  - a POJO,
  - generated `equals()` / `hashCode()` / `toString()`,
  - and part of what Java `record` gives you,
  - plus optional ordering support.

Why this matters:
- excellent for value objects,
- reduces boilerplate dramatically,
- often the most Pythonic replacement for hand-written Java-style model classes.

## 9. Class Methods and Static Methods

### `@classmethod`

A class method receives `cls` and can access class-level data.

```python
class Person:
    count = 0

    def __init__(self, name, age):
        self.name = name
        self.age = age
        Person.count += 1

    @classmethod
    def get_population(cls):
        return f"Population: {cls.count}"


lecturer = Person("Josiah", 20)
print(Person.get_population())
print(lecturer.get_population())
```

### `@staticmethod`

A static method is just a utility function placed inside a class namespace.

```python
class Person:
    @staticmethod
    def calculate_sum(num1, num2):
        return num1 + num2


print(Person.calculate_sum(5, 3))
```

## 10. File Handling and JSON

### File handling with `with open(...)`

Python uses context managers to safely manage files.

```python
with open("file.txt", "r") as file:
    content = file.read()
    print(content)

with open("file.txt", "w") as file:
    file.write("Hello, file!")
```

### JSON: load and dump

Use the built-in `json` module.

```python
import json

# Read from file
with open("input.json", "r") as jsonfile:
    data = json.load(jsonfile)

# Read from string
json_string = '[{"id": 2, "name": "Basilisk"}]'
parsed_data = json.loads(json_string)

# Write to file
with open("output.json", "w") as jsonfile:
    json.dump(parsed_data, jsonfile)

# Write to string
output = json.dumps(parsed_data)
print(output)
```

## 11. Assertions, Advanced Control Flow, and Scope Keywords

### `assert`

`assert` is used to verify that a condition is true during development. If the condition is false, Python raises an `AssertionError`.

```python
def divide(a, b):
    assert b != 0, "b must not be zero"
    return a / b
```

For a Java developer:
- it plays the same role as Java `assert`
- it is best used for internal assumptions, not user-facing validation
- Python assertions can be disabled with optimization flags, so do not use them for critical business rules

### `break`

`break` immediately exits the nearest loop.

```python
for n in range(10):
    if n == 5:
        break
    print(n)
```

### `while ... else`

Python supports `else` on loops. For `while`, it means:
- run the `else` block if the loop finishes normally
- do not run it if the loop exits via `break`

```python
n = 3

while n > 0:
    print(n)
    n -= 1
else:
    print("Loop finished normally")
```

### `for ... else`

The same rule applies to `for` loops.

```python
numbers = [2, 4, 6, 8]
target = 5

for n in numbers:
    if n == target:
        print("Found")
        break
else:
    print("Not found")
```

A good mental model is:
- loop `else` = **no `break` happened**

### `del`

`del` deletes a binding, list item, slice, dictionary entry, or object attribute.

```python
numbers = [10, 20, 30]
del numbers[1]
print(numbers)
```

```python
user = {"name": "Alice", "age": 25}
del user["age"]
print(user)
```

Important nuance:
- `del` does not force memory destruction
- it removes a reference or entry
- garbage collection still decides when memory is reclaimed

### `nonlocal`

`nonlocal` lets an inner function modify a variable from its enclosing function scope.

```python
def outer():
    count = 0

    def inner():
        nonlocal count
        count += 1
        return count

    return inner

counter = outer()
print(counter())
print(counter())
```

Java comparison:
- Java lambdas usually capture effectively final variables
- Python is more flexible here because `nonlocal` allows reassignment in the enclosing scope

## 12. Lazy Evaluation and Generators

### `yield`

When a function uses `yield`, it becomes a **generator function**.
Instead of returning once and finishing, it produces values one at a time and remembers its state between pauses.

```python
def countdown(n):
    while n > 0:
        yield n
        n -= 1

for value in countdown(3):
    print(value)
```

Why this matters:
- it feels similar to a lazy iterator
- it avoids loading everything into memory at once
- it is useful for streaming and pipelines

### Example: reading a big file 10 lines at a time

```python
def read_in_batches(file_path, batch_size=10):
    with open(file_path, "r") as file:
        batch = []

        for line in file:
            batch.append(line.rstrip("\n"))
            if len(batch) == batch_size:
                yield batch
                batch = []

        if batch:
            yield batch
```

```python
for lines in read_in_batches("large.log", batch_size=10):
    print("New batch:")
    for line in lines:
        print(line)
```

### Generator vs Coroutine

A generator and a coroutine both support pause/resume behavior, but they are used for different purposes.

| Concept | Generator | Coroutine |
| :--- | :--- | :--- |
| **Defined with** | `def` + `yield` | `async def` |
| **Main purpose** | Produce values lazily | Suspend during async work |
| **Typical use case** | Iteration, streaming, pipelines | Async I/O, event-loop concurrency |
| **Pause point** | `yield` | `await` |
| **Mental model** | lazy iterator | async task |

So the short answer is:
- a generator function is not usually what modern Python means by a coroutine
- but historically, generator-based coroutines did exist
- and both share the idea of suspend and resume

## 13. Reflection, Introspection, and Metaprogramming

Python supports something broadly similar to Java reflection, but the more common term is **introspection**.
Because Python is highly dynamic, many reflection-style tasks feel more natural and lightweight.

### Core reflection-style built-ins

#### `type()`
```python
value = [1, 2, 3]
print(type(value))
```

#### `isinstance()`
```python
value = [1, 2, 3]
print(isinstance(value, list))
```

#### `dir()`
```python
name = "Alice"
print(dir(name))
```

#### `getattr()` / `setattr()` / `hasattr()`
```python
class User:
    def __init__(self):
        self.name = "Alice"

user = User()
print(getattr(user, "name"))
print(hasattr(user, "age"))
setattr(user, "age", 25)
print(user.age)
```

### `inspect`

```python
import inspect

def greet(name: str) -> str:
    return f"Hello, {name}"

print(inspect.signature(greet))
```

### Metaprogramming: changing classes at runtime

Yes, Python supports this.
Classes are runtime objects, so you can:
- add attributes dynamically
- replace methods
- generate classes dynamically
- use metaclasses

#### Add a method at runtime
```python
class Person:
    def __init__(self, name):
        self.name = name

def greet(self):
    return f"Hello, I am {self.name}"

Person.greet = greet
p = Person("Alice")
print(p.greet())
```

#### Build a class dynamically
```python
User = type("User", (), {"role": "admin"})
print(User.role)
```

Java / Groovy comparison:
- Java reflection is more formal and restricted
- Groovy is much more dynamic
- Python is also highly dynamic and supports runtime metaprogramming, though it should be used carefully

## 14. Practical Takeaways for Java Developers

- Write less boilerplate.
- Prefer readability and directness.
- Use built-in data structures aggressively: `dict`, `list`, `tuple`, `set`.
- Treat functions as values.
- Use `@property` instead of writing Java-style getters/setters everywhere.
- Prefer composition over inheritance when possible.
- Use `if __name__ == "__main__":` for script-only execution.
- Use the standard library first; it already includes a lot.
- Remember that generator, reflection, and metaprogramming features are powerful, but readability still matters.

## 15. Final Summary

Java and Python are both powerful, but they optimize for different things:

- **Java** favors structure, explicitness, and performance.
- **Python** favors flexibility, brevity, and developer productivity.

If you are coming from Java, the biggest shift is not just syntax. It is learning when to stop writing Java-style code and start writing Pythonic code.