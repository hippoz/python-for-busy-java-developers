# Part 2 — Java Idiom Translation

Goal: take the Java instincts you already have (OOP, `equals`/`hashCode`, collections, enums, records) and translate them into Python that other Python developers will recognize as well-formed.

**Prerequisites:** [Part 1 — Syntax Shock](01_syntax_shock.md). **Next:** [Part 3 — Pythonic Idioms](03_pythonic_idioms.md).

---

## Table of Contents

- [Core differences](#core-differences)
- [Collections mapping](#collections-mapping)
- [Tuple](#tuple)
- [Frozenset](#frozenset)
- [Binary types](#binary-types)
- [OOP basics](#oop-basics)
- [Dict vs class](#dict-vs-class)
- [Abstract classes](#abstract-classes)
- [Encapsulation](#encapsulation)
- [Access conventions](#access-conventions)
- [Composition over inheritance](#composition-over-inheritance)
- [Class and static methods](#class-and-static-methods)
- [Dunder methods](#dunder-methods)
- [Java object-model mapping](#java-object-model-mapping)
- [Dataclass](#dataclass)
- [Enum](#enum)
- [Slots](#slots)
- [Mutability](#mutability)
- [Class vs instance attributes](#class-vs-instance-attributes)
- [MRO](#mro)
- [Init subclass](#init-subclass)
- [Key Takeaways](#key-takeaways)

---

## Core differences

| Topic | Java | Python |
| :--- | :--- | :--- |
| Class header | `public class X { … }` | `class X:` |
| Constructor | named after class, optional implicit super | `__init__(self, …)` |
| Field access | enforced by `private`/`public` | convention via `_name` / `__name` |
| Value object | `record` / Lombok | `@dataclass` / `@dataclass(frozen=True)` |
| Equality | `equals` + `hashCode` (must override together) | `__eq__` + `__hash__` (Python protects you if you forget) |
| Collections home | `java.util` | built into the language |
| Inheritance | single class + many interfaces | multiple class inheritance (with MRO) + structural `Protocol` |

```java
// Java
record User(String name, int age) {}
User u = new User("Alice", 25);
```

```python
# Python
from dataclasses import dataclass

@dataclass
class User:
    name: str
    age: int

u = User("Alice", 25)
```

## Collections mapping

Python's core collections live in the language itself. They're not in a `java.util` equivalent — they're built-in.

| Java | Python | Notes |
| :--- | :--- | :--- |
| `HashMap<K,V>` | `dict` | Insertion-ordered since 3.7; hash-based |
| `LinkedHashMap` | `dict` | Same type; ordering is now guaranteed |
| `TreeMap` | `dict` + `sorted()` at read time | Or `sortedcontainers.SortedDict` (3rd-party) |
| `HashSet<T>` | `set` | Hash-based, unordered, unique |
| `TreeSet<T>` | `sorted(some_set)` at read time | No sorted set built in |
| `ArrayList<T>` | `list` | Array-backed |
| `LinkedList<T>` used as `List` | `list` | Python `list` is what you want |
| `Deque` / `ArrayDeque` | `collections.deque` | For queue/stack usage |
| immutable list | `tuple` | Fixed-size, immutable |
| `byte[]` | `bytes` / `bytearray` | See [Binary types](#binary-types) |

```python
user_ages = {"Alice": 25, "Bob": 30}
print(user_ages["Alice"])           # >>> 25

# Iteration: keys by default
for key in user_ages:
    print(key)                      # >>> Alice  Bob

# Both keys and values
for key, value in user_ages.items():
    print(key, value)               # >>> Alice 25  Bob 30
```

> ⚠️ **Pitfall:** `dict` preserves *insertion* order, not sorted order. If you need sorted iteration, do it explicitly: `for k in sorted(my_dict): ...`. Same applies to `set`. Don't reach for a Python `TreeMap` — sort at the read site.

A `set` is the `HashSet` analog. It also deduplicates a sequence in one shot:

```python
words = ["let", "it", "go", "let", "it", "go"]
vocabulary = set(words)
print(vocabulary)                   # >>> {'let', 'it', 'go'}
```

**Empty-set quirk:** `{}` creates an empty **dict**, not an empty set. Use `set()`:

```python
print(type({}))                     # >>> <class 'dict'>
print(type(set()))                  # >>> <class 'set'>
```

## Tuple

A tuple is like a list, but **immutable** — and (when its elements are also hashable) that lets it be used as a dict key or set element. It also signals "fixed-shape record" rather than "growable sequence."

> ⚠️ **Pitfall:** A tuple of mutables is still a tuple, but it's **not** hashable. `(1, 2)` works as a dict key; `(1, [])` raises `TypeError: unhashable type: 'list'` when you try.

```python
x = (1, 2, 3, 4, 5)
print(x[0])      # >>> 1
print(x[1:3])    # >>> (2, 3)
print(x[-1])     # >>> 5
print(x[::-1])   # >>> (5, 4, 3, 2, 1)
```

**Single-element tuple quirk** — needs the trailing comma:

```python
not_a_tuple = (1)        # this is just int 1 in parens
singleton    = (1,)      # this is a 1-tuple

print(type(not_a_tuple)) # >>> <class 'int'>
print(type(singleton))   # >>> <class 'tuple'>
```

**Immutability is enforced at runtime:**

```python
x = (1, 2, 3)
# x[1] = 6      # TypeError: 'tuple' object does not support item assignment
# x.append(6)   # AttributeError: 'tuple' object has no attribute 'append'
```

`tuple(some_list)` does the conversion. For fixed-shape records with names, see [`@dataclass(frozen=True)`](#dataclass) below.

## Frozenset

The immutable version of `set`. Roughly analogous to an unmodifiable set in Java — but a true copy, not a view.

```python
roles = frozenset(["admin", "editor", "viewer"])
```

Because it's immutable and hashable, it can be a `dict` key or live inside another `set`:

```python
permissions = {
    frozenset(["read", "write"]): "editor",
    frozenset(["read"]):          "viewer",
}
```

A mutable `set` cannot, because hashing depends on contents.

> ⚠️ **Pitfall:** Java's `Collections.unmodifiableSet(s)` is a *view* — mutations to `s` ARE visible through the wrapper. `frozenset(s)` is a *copy* — it stands alone. Different guarantees.

## Binary types

When you work with files, network protocols, images, or any wire format, you'll meet three types:

- `bytes` — immutable byte sequence (≈ `byte[]` you can't write to)
- `bytearray` — mutable byte sequence (≈ a mutable byte buffer)
- `memoryview` — zero-copy window over an existing buffer

```python
# bytes literal
payload = b"hello"
print(payload[0])     # >>> 104   (integer, not 'h')

# reading binary
with open("image.png", "rb") as f:
    data = f.read()   # >>> bytes

# bytearray: mutate in place
buffer = bytearray(b"hello")
buffer[0] = 72        # H
print(buffer)         # >>> bytearray(b'Hello')

# memoryview: no copy
data = bytearray(b"abcdef")
view = memoryview(data)
view[1] = 90          # 'Z'
print(data)           # >>> bytearray(b'aZcdef')
```

A useful mental model:
- `bytes` = immutable binary data
- `bytearray` = mutable binary data
- `memoryview` = window onto existing binary data without copying

Use `memoryview` when slicing or passing large buffers around and you want to avoid copy overhead.

## OOP basics

A class definition with no body uses `pass`:

```python
class Person:
    pass

person = Person()
```

No `new` keyword. Calling the class name like a function instantiates it. There is no separate compilation step.

> 💡 **Pythonic:** "The Bean → @dataclass." Before reaching for a hand-written class with constructor, fields, getters, and `equals`, ask whether the class is just a value container. If yes, jump straight to [`@dataclass`](#dataclass).

A normal hand-written class:

```python
class Person:
    def __init__(self, name, age):
        self.name = name
        self.age = age

    def greet(self):
        return f"Hi, I'm {self.name}"

p = Person("Alice", 25)
print(p.greet())   # >>> Hi, I'm Alice
```

Two things to notice:

1. `self` is **explicit** — it's the first parameter of every instance method. It plays the role of Java's implicit `this`, but you write it out.
2. `__init__` is the initializer (not a "constructor" in the strict sense — the object exists before `__init__` runs).

## Dict vs class

If you only need a simple data container, a `dict` is often enough:

```python
user = {"name": "Alice", "age": 25}
print(user["name"])
```

This is idiomatic for "this is a row," "this is a JSON-shaped thing," or "this is a config blob." Reach for a class when you need behavior (methods), validation, or a stable schema. Reach for a `@dataclass` when you want a typed record without writing boilerplate.

## Abstract classes

Python has abstract classes via the standard library `abc` module. The contract is enforced at instantiation:

```python
from abc import ABC, abstractmethod

class Animal(ABC):
    @abstractmethod
    def make_sound(self):
        pass

class Dog(Animal):
    def make_sound(self):
        return "Woof"

# Animal()   # TypeError: Can't instantiate abstract class Animal
print(Dog().make_sound())  # >>> Woof
```

`ABC` is the helper base; `@abstractmethod` flags methods subclasses must override. The benefits are the same as in Java: enforce an interface, prevent instantiation of the abstract base, catch missing implementations early.

`abc.ABC` is **nominal** (subclasses declare themselves by inheriting). For **structural** subtyping (any class with the right methods qualifies, no inheritance required) Python offers `typing.Protocol` — see [Part 3 § Protocol](03_pythonic_idioms.md#protocol).

## Encapsulation

Python keeps attributes public by default. If you later need control — validation, derived values, change tracking — promote the attribute to a `@property`:

```python
class Person:
    def __init__(self, name, age):
        self.name = name
        self.age = age            # calls the setter below

    @property
    def age(self):
        return self._age

    @age.setter
    def age(self, new_age):
        if new_age < 0:
            raise ValueError("age must be non-negative")
        self._age = new_age

person = Person("Alice", 25)
print(person.age)                 # >>> 25       (reads via @property getter)
person.age = 30                   # writes via setter
```

Callers see plain attribute access. You retain control. The Java instinct of "start every field with a getter/setter just in case" is unnecessary — promote later if and when you need to.

## Access conventions

Python has no `private` keyword. It uses convention plus a tiny bit of mangling:

- `name` → public
- `_name` → "internal use" (convention; nothing enforced)
- `__name` → name-mangled to `_ClassName__name` to prevent accidental clashes in inheritance

```python
class Account:
    def __init__(self):
        self.balance = 0        # public
        self._cache = {}        # internal
        self.__secret = "shh"   # name-mangled

a = Account()
print(a.balance)                # >>> 0
print(a._cache)                 # >>> {}      (works, but you shouldn't)
# print(a.__secret)             # AttributeError
print(a._Account__secret)       # >>> shh     (the mangled name)
```

> 💡 **Pythonic:** "We're all consenting adults here" — Python relies on convention. Don't fight this with elaborate access-control schemes. If a name starts with `_`, treat it as private.

## Composition over inheritance

This is the same principle Java teaches you, but Python takes it further in practice. Because there's no `interface` keyword forcing you into hierarchies, and because duck typing means you don't *need* a common base, composition is often the lighter answer.

```python
# Inheritance — couples Engine into Car's type hierarchy
class Engine:
    def start(self):
        return "vroom"

class Car(Engine):
    def drive(self):
        return self.start() + " — driving"

# Composition — Car holds an engine, doesn't *is-a* engine
class Engine:
    def start(self):
        return "vroom"

class Car:
    def __init__(self, engine):
        self.engine = engine

    def drive(self):
        return self.engine.start() + " — driving"

c = Car(Engine())
print(c.drive())   # >>> vroom — driving
```

The composed version lets you swap engines (`Car(ElectricEngine())`), test with a fake, and avoid the diamond-inheritance and override-order headaches covered under [MRO](#mro).

## Class and static methods

A `@classmethod` receives the class itself (`cls`) and can access class-level state. A `@staticmethod` is just a function that happens to live inside a class namespace.

```python
class Person:
    count = 0                              # class-level attribute

    def __init__(self, name):
        self.name = name
        Person.count += 1

    @classmethod
    def get_population(cls):
        return f"Population: {cls.count}"

    @staticmethod
    def is_adult(age):
        return age >= 18

Person("Alice")
Person("Bob")
print(Person.get_population())    # >>> Population: 2
print(Person.is_adult(20))        # >>> True
```

Classmethods often serve as alternate constructors (`@classmethod def from_dict(cls, d): return cls(**d)`). Staticmethods are utility helpers that conceptually belong with the class but don't need its state.

## Dunder methods

"Dunder" = "double underscore". These special methods let your class participate in Python's built-in operations.

| Behavior | Method |
| :--- | :--- |
| constructor / initializer | `__init__` |
| user-facing string | `__str__` |
| debug-oriented string | `__repr__` |
| equality | `__eq__` |
| hashing | `__hash__` |
| `<`, `<=`, `>`, `>=` | `__lt__`, `__le__`, `__gt__`, `__ge__` |
| `+`, `-`, `*` | `__add__`, `__sub__`, `__mul__` |
| `len(x)` | `__len__` |
| `x in y` | `__contains__` |
| `x[i]` / `x[i] = v` | `__getitem__` / `__setitem__` |
| iteration (`for v in x`) | `__iter__` |
| context manager (`with x:`) | `__enter__` / `__exit__` |

A small example with operator overloading:

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
print(v1 + v2)            # >>> Vector(6, 9)
```

The next section covers the specific subset Java developers ask about: `equals`, `hashCode`, `toString`, `Comparable`, `Cloneable`.

## Java object-model mapping

| Java | Python | Where to read |
| :--- | :--- | :--- |
| `toString()` | `__str__` (user-facing) / `__repr__` (debug) | this section |
| `equals(Object)` | `__eq__(self, other)` | this section |
| `hashCode()` | `__hash__(self)` | this section |
| `Comparable.compareTo` | `__lt__` (+ `functools.total_ordering`) or `sorted(key=…)` | this section |
| `Cloneable` | `copy.copy` / `copy.deepcopy` | this section |

### `__str__` and `__repr__`

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
print(str(user))   # >>> User Alice
print(repr(user))  # >>> User(name='Alice', age=25)
```

Rule of thumb:
- `__str__` — friendly, for end users (`print(obj)`)
- `__repr__` — precise, for debugging (`>>> obj` in a REPL, log lines, error messages)

Define `__repr__` always; define `__str__` only if you want something different from `__repr__`.

### `__eq__` and `__hash__`

```python
class User:
    def __init__(self, name):
        self.name = name

    def __eq__(self, other):
        return isinstance(other, User) and self.name == other.name

    def __hash__(self):
        return hash(self.name)

print(User("Alice") == User("Alice"))  # >>> True
```

The Java rule applies in Python too: **if two objects are equal, they must have the same hash**. Get this wrong and your `set`/`dict` behavior silently breaks.

> ⚠️ **Pitfall:** If you define `__eq__` but **don't** define a compatible `__hash__`, Python sets `__hash__ = None` — your class becomes unhashable (can't go in sets, can't be dict keys). This is Python's way of protecting you from a broken equals/hash contract.

### `Comparable` → rich comparison methods

Java's `Comparable<T>` is one method. Python spreads it across `__lt__`, `__le__`, `__gt__`, `__ge__`:

```python
class User:
    def __init__(self, age):
        self.age = age

    def __lt__(self, other):
        return self.age < other.age

print(User(20) < User(30))   # >>> True
```

A common pattern: define `__eq__` and `__lt__`, then use `functools.total_ordering` to fill in the rest:

```python
from functools import total_ordering

@total_ordering
class User:
    def __init__(self, age):
        self.age = age

    def __eq__(self, other):
        return isinstance(other, User) and self.age == other.age

    def __lt__(self, other):
        return self.age < other.age
```

> 💡 **Pythonic:** For sorting, the idiomatic move is to pass a `key` to `sorted` rather than force the class to define a universal ordering: `sorted(users, key=lambda u: u.age)`. Define rich comparison methods only when ordering is intrinsic to the class.

### `Cloneable` → the `copy` module

Python has no `Cloneable` interface. Copying is handled by the `copy` module:

```python
import copy

original = [[1, 2], [3, 4]]
shallow = copy.copy(original)
shallow[0].append(99)
print(original)   # >>> [[1, 2, 99], [3, 4]]   (shared nested list!)
print(shallow)    # >>> [[1, 2, 99], [3, 4]]

deep = copy.deepcopy(original)
deep[0].append(77)
print(original)   # >>> [[1, 2, 99], [3, 4]]   (untouched)
print(deep)       # >>> [[1, 2, 99, 77], [3, 4]]
```

Same shallow-vs-deep distinction as Java, just without the awkward `Cloneable` interface.

## Dataclass

When `dict` is too loose and a hand-written class is too verbose, reach for `@dataclass`:

```python
from dataclasses import dataclass

@dataclass
class User:
    name: str
    age: int

u = User("Alice", 25)
print(u)              # >>> User(name='Alice', age=25)
print(u == User("Alice", 25))  # >>> True
```

This auto-generates `__init__`, `__repr__`, and `__eq__`.

**`frozen=True` and `order=True`** give you a Java-`record`-like value object:

```python
@dataclass(order=True, frozen=True)
class User:
    age: int
    name: str

print(sorted([User(30, "B"), User(20, "A")]))
# >>> [User(age=20, name='A'), User(age=30, name='B')]
```

`frozen=True` blocks attribute *rebinding* (`u.age = 30` raises `FrozenInstanceError`) and lets `__hash__` be generated. It is **not** deep immutability — a frozen dataclass with a `list` field still lets `u.items.append(...)`. And the generated `__hash__` still fails at runtime if any field's value is itself unhashable. For value-object semantics that go deeper, use immutable field types (`tuple`, `frozenset`) too. `order=True` generates `__lt__`/`__le__`/`__gt__`/`__ge__` based on field order.

**`__post_init__` for validation:**

```python
from dataclasses import dataclass

@dataclass
class User:
    name: str
    age: int

    def __post_init__(self):
        if self.age < 0:
            raise ValueError("age must be non-negative")
        self.name = self.name.strip()
```

`__post_init__` runs after `__init__`. It's the right place to validate fields, normalize values, or compute derived state. Java developers used to "validation in the constructor" should look here first.

**Mutable defaults need `field(default_factory=...)`:**

```python
from dataclasses import dataclass, field

@dataclass
class Cart:
    items: list = field(default_factory=list)   # NOT items: list = []
```

The `default_factory` runs once per instance. Writing `items: list = []` in a dataclass actually raises `ValueError: mutable default <class 'list'> for field items is not allowed: use default_factory` at class-definition time — `@dataclass` knows about the trap covered under [Mutability](#mutability) and refuses to compile. (Plain `def f(items=[]):` has no such protection, which is why that section warns about it.)

> ☕ **Java parallel:** `@dataclass(frozen=True)` is the closest thing to Java `record` + auto-generated `equals`/`hashCode`/`toString` + immutability. For runtime validation comparable to Bean Validation, see `pydantic` in [Part 6 § Productivity libs](06_ecosystem_and_packaging.md#productivity-libs).

## Enum

Python's `enum` module covers what you'd reach for Java `enum` to do:

```python
from enum import Enum, IntEnum, StrEnum, auto

class Color(Enum):
    RED   = auto()
    GREEN = auto()
    BLUE  = auto()

print(Color.RED)         # >>> Color.RED
print(Color.RED.name)    # >>> RED
print(Color.RED.value)   # >>> 1

# Identity comparison is fine — Enum members are singletons
c = Color.RED
print(c is Color.RED)    # >>> True

# IntEnum — members compare as integers
class HttpStatus(IntEnum):
    OK             = 200
    NOT_FOUND      = 404
    SERVER_ERROR   = 500

print(HttpStatus.OK == 200)  # >>> True

# StrEnum (3.11+) — members compare as strings
class Role(StrEnum):
    ADMIN  = "admin"
    EDITOR = "editor"

print(Role.ADMIN == "admin") # >>> True
```

> 🐍 **Python 3.11+:** `StrEnum` is the right choice for "enum that's also a string" — e.g., enum values that need to round-trip through JSON or a database without `.value` everywhere.

For methods on enum members, define them inside the `Enum` class — just like Java:

```python
class HttpStatus(IntEnum):
    OK            = 200
    NOT_FOUND     = 404
    SERVER_ERROR  = 500

    def is_error(self):
        return self.value >= 400
```

## Slots

Each Python instance normally carries a `__dict__` for its attributes — flexible but ~100 bytes of overhead. For classes you instantiate by the million (libraries, data structures), `__slots__` shrinks instances and skips the dict:

```python
class Point:
    __slots__ = ("x", "y")

    def __init__(self, x, y):
        self.x = x
        self.y = y

p = Point(1, 2)
# p.z = 3   # AttributeError — slots forbids new attributes
```

Tradeoffs: no `__dict__` (can't add attrs at runtime), and multiple inheritance with slots gets fiddly. In application code this is overkill; in library hot paths it's a real win. Rare to need, useful to recognize.

## Mutability

This is one of the most consequential Python topics for a Java developer.

**Common immutable types:** `int`, `float`, `bool`, `str`, `tuple`, `frozenset`, `bytes`.

**Common mutable types:** `list`, `dict`, `set`, `bytearray`, plain user classes by default.

```python
x = [1, 2]
y = x
y.append(3)
print(x)   # >>> [1, 2, 3]
```

Both `x` and `y` point at the same list object. This is identical to Java reference semantics — just more visible because Python uses mutable collections constantly.

**Pitfall: mutable default arguments.** This is one of the most-misunderstood Python footguns:

```python
def add_item(item, items=[]):       # ⚠️ DANGER
    items.append(item)
    return items

print(add_item(1))  # >>> [1]
print(add_item(2))  # >>> [1, 2]      ← shared across calls!
print(add_item(3))  # >>> [1, 2, 3]
```

The `items=[]` default is evaluated **once at function-definition time**, not on each call. All calls without an explicit `items` share the same list.

**The fix** — use `None` as the sentinel and create the mutable inside:

```python
def add_item(item, items=None):
    if items is None:
        items = []
    items.append(item)
    return items
```

Same trap appears with dataclass defaults — see [`field(default_factory=...)`](#dataclass).

## Class vs instance attributes

A subtle pitfall Java developers regularly hit. Putting a mutable on the class (not in `__init__`) makes it **shared across all instances**:

```python
class Inbox:
    messages = []         # ⚠️ class attribute — shared!

    def add(self, msg):
        self.messages.append(msg)

a = Inbox()
b = Inbox()
a.add("hello")
print(b.messages)         # >>> ['hello']    ← b sees a's message
```

The fix is to assign in `__init__` so each instance gets its own:

```python
class Inbox:
    def __init__(self):
        self.messages = []          # instance attribute — per-instance

    def add(self, msg):
        self.messages.append(msg)

a = Inbox()
b = Inbox()
a.add("hello")
print(b.messages)         # >>> []
```

**When class attributes ARE the right choice:** constants and shared counters that you intentionally want shared.

```python
class Person:
    SPECIES = "Homo sapiens"          # constant — fine as class attr

    def __init__(self, name):
        self.name = name              # per-instance
```

If you ever assign through `self.x = ...` where the class also has `x`, you create an *instance* attribute that shadows the class one for that single instance. That can be intentional, or it can be a bug. Be deliberate.

## MRO

Python supports multiple inheritance. When the same method exists in multiple ancestors, the **Method Resolution Order** (MRO) decides which wins. Python uses the **C3 linearization** algorithm.

```python
class A:
    def hello(self): return "A"

class B(A):
    def hello(self): return "B"

class C(A):
    def hello(self): return "C"

class D(B, C):
    pass

print(D().hello())   # >>> B
print(D.__mro__)
# >>> (<class 'D'>, <class 'B'>, <class 'C'>, <class 'A'>, <class 'object'>)
```

`D.__mro__` is the lookup order: `D → B → C → A → object`. `super()` walks this chain, so cooperative multiple inheritance can work — though it requires every class in the diamond to call `super()` and accept consistent `**kwargs`.

```python
class A:
    def __init__(self, **kw): print("A"); super().__init__(**kw)
class B(A):
    def __init__(self, **kw): print("B"); super().__init__(**kw)
class C(A):
    def __init__(self, **kw): print("C"); super().__init__(**kw)
class D(B, C):
    def __init__(self, **kw): print("D"); super().__init__(**kw)

D()
# >>> D
# >>> B
# >>> C
# >>> A
```

> 💡 **Pythonic:** Multiple inheritance + `super()` works, but composition (see [Composition over inheritance](#composition-over-inheritance)) is usually clearer. Reach for diamond inheritance only when mixins genuinely model the problem (often: `LoggerMixin`, `JSONSerializableMixin`).

## Init subclass

`__init_subclass__` is a classmethod-style hook that runs when a **subclass is defined** (not when it's instantiated). It's the Pythonic registry hook — no metaclasses required.

```python
class Plugin:
    registry = {}

    def __init_subclass__(cls, **kwargs):
        super().__init_subclass__(**kwargs)
        Plugin.registry[cls.__name__] = cls

class Exporter(Plugin):
    pass

class Importer(Plugin):
    pass

print(Plugin.registry)
# >>> {'Exporter': <class 'Exporter'>, 'Importer': <class 'Importer'>}
```

> ☕ **Java parallel:** Think Spring `@Component` scanning. Spring discovers your beans by classpath scanning; Python collects them via the class-definition hook. Much lighter — no scanner, no annotations, no reflection. Drop a subclass anywhere it gets imported, and the registry sees it.

Common uses: plugin discovery, schema/model registration (web frameworks, ORMs), enforcing subclass invariants at definition time.

## Key Takeaways

- Dunder methods are hooks, not magic — `__eq__` and `__hash__` must stay consistent.
- `@dataclass(frozen=True)` ≈ Java `record` + `equals`/`hashCode`/`toString`; use `__post_init__` for constructor-style validation.
- Reach for `dict` over a hand-written class for pure data containers; reach for `@dataclass` when you want types.
- Class attributes are *shared*; instance attributes (`self.x`) are per-object. The mutable-class-attr trap is the same one as mutable default arguments.
- Composition > inheritance — even more than in Java, because duck typing and `Protocol` mean you rarely need a shared base.
- `__init_subclass__` replaces Spring-style classpath scanning with a lightweight definition-time hook.
