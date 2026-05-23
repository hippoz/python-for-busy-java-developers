# Python Language Learning for Busy Java Developers, Part 4

## 1. Core Differences at a Glance

| Java Area | Common Java Package / Class | Python Counterpart | Notes |
| :--- | :--- | :--- | :--- |
| **Core language utilities** | `java.lang` | built-ins + standard library | Python spreads core functionality across built-ins and modules |
| **Collections** | `java.util` | `list`, `dict`, `set`, `tuple`, `collections` | Many data structures are built directly into the language |
| **Dates and time** | `java.time` | `datetime`, `time`, `zoneinfo` | Modern Python date/time work is mainly in `datetime` |
| **Filesystem / paths** | `java.nio.file` | `pathlib`, `os`, `shutil` | `pathlib` is the most Pythonic modern API |
| **JSON** | Jackson / Gson / `java.json` | `json` | Built into the standard library |
| **Random / math** | `Math`, `Random` | `math`, `random`, `statistics` | Common numeric helpers are batteries-included |
| **Concurrency helpers** | `java.util.concurrent` | `threading`, `queue`, `concurrent.futures`, `asyncio` | Spread across several modules |
| **Regular expressions** | `java.util.regex` | `re` | Similar idea, different syntax details |
| **Logging** | `java.util.logging`, Log4j, SLF4J | `logging` | Built-in and sufficient for many projects |
| **Configuration / serialization** | `Properties`, serializers | `configparser`, `json`, `pickle` | Python tends to favor simpler text-based formats |

### Minimal comparison

**Java**
```java
import java.time.LocalDateTime;
System.out.println(LocalDateTime.now());
```

**Python**
```python
from datetime import datetime
print(datetime.now())
```

## 2. Core Language Utilities (`java.lang`-like)

In Java, `java.lang` contains the classes you use constantly: `String`, `Math`, wrappers, exceptions, `System`, and so on.
In Python, the equivalent functionality is split between:

- built-in types,
- built-in functions,
- small standard library modules.

### Common mappings

| Java | Python |
| :--- | :--- |
| `String` | `str` |
| `Math` | `math` |
| `System.out.println(...)` | `print(...)` |
| `Integer.parseInt(...)` | `int(...)` |
| `Double.parseDouble(...)` | `float(...)` |
| `null` | `None` |
| `Object` | `object` |
| `Exception` | `Exception` |

### Example

```python
name = "Alice"
print(name.upper())
print(len(name))
print(int("42"))
```

### Important difference

Java centralizes more of this functionality into class-based APIs.
Python is more function-oriented and type-oriented:
- methods live on built-in objects,
- many global operations are built-ins (`len`, `print`, `type`, `isinstance`).

## 3. Collections and Utility Helpers (`java.util`-like)

### Built-in collections

Python has several collection types built directly into the language:

- `list`
- `dict`
- `set`
- `tuple`

This is a major difference from Java, where many collection types come from `java.util`.

```python
numbers = [1, 2, 3]
user = {"name": "Alice", "age": 25}
roles = {"admin", "editor"}
point = (10, 20)
```

### `collections`

When built-ins are not enough, Python's `collections` module gives more specialized tools.

#### `Counter`

```python
from collections import Counter

counts = Counter(["a", "b", "a", "c", "a"])
print(counts)
```

#### `defaultdict`

```python
from collections import defaultdict

items = defaultdict(list)
items["fruit"].append("apple")
print(items)
```

#### `deque`

```python
from collections import deque

queue = deque([1, 2, 3])
queue.appendleft(0)
queue.append(4)
print(queue)
```

### Java comparison

These roughly map to habits like:
- `Map<K, Integer>` frequency maps,
- `HashMap<K, List<V>>`,
- `Deque` / `ArrayDeque`.

But in Python, they are usually shorter and more direct to use.

## 4. Date and Time (`java.time`-like)

Modern Python date/time work mostly uses `datetime`.

### Current time

```python
from datetime import datetime

now = datetime.now()
print(now)
```

### Date only

```python
from datetime import date

print(date.today())
```

### Time delta

```python
from datetime import datetime, timedelta

now = datetime.now()
future = now + timedelta(days=7)
print(future)
```

### Time zones

```python
from datetime import datetime
from zoneinfo import ZoneInfo

now_tokyo = datetime.now(ZoneInfo("Asia/Tokyo"))
print(now_tokyo)
```

### Java comparison

Roughly analogous to:
- `LocalDate`
- `LocalDateTime`
- `ZonedDateTime`
- `Duration` / `Period`

The biggest difference is that Python's API surface is smaller, but time zone handling still requires care.

## 5. Files, Paths, and the Filesystem (`java.nio.file`-like)

### `pathlib`

For modern Python, `pathlib` is the most Pythonic way to work with paths.

```python
from pathlib import Path

path = Path("logs/app.log")
print(path.name)
print(path.suffix)
print(path.parent)
```

### Reading and writing files

```python
from pathlib import Path

path = Path("notes.txt")
path.write_text("hello")
print(path.read_text())
```

### Create directories

```python
from pathlib import Path

Path("output/reports").mkdir(parents=True, exist_ok=True)
```

### Copy files

```python
import shutil

shutil.copy("source.txt", "backup.txt")
```

### Java comparison

This is conceptually similar to:
- `Path`
- `Files`
- file utility operations

But Python's syntax is usually lighter and less ceremony-heavy.

## 6. Text, Regular Expressions, and String Processing

### String operations

```python
text = "  hello,world  "
print(text.strip())
print(text.split(","))
print(text.replace("world", "python"))
```

### Regular expressions with `re`

```python
import re

text = "Order ID: 12345"
match = re.search(r"\d+", text)
print(match.group())
```

### Java comparison

Similar role to:
- `String`
- `StringBuilder` for some tasks
- `java.util.regex.Pattern` / `Matcher`

Python string handling is usually more concise, especially for everyday transformations.

## 7. JSON, Config, and Serialization

### JSON

```python
import json

data = {"name": "Alice", "age": 25}
json_text = json.dumps(data)
print(json_text)
```

### Config files


### Environment variables and `.env` files

For Java developers, this is the Python equivalent of externalized configuration such as:
- environment variables,
- application config files,
- secret injection at deployment time.

At the standard library level, Python reads environment variables through `os.environ`.

```python
import os

api_key = os.environ.get("API_KEY")
print(api_key)
```

A very common real-world pattern is to keep local development variables in a `.env` file, then load them into the process environment.
The most common tool for this is the third-party package `python-dotenv`.

Example `.env` file:

```env
APP_ENV=development
API_KEY=secret-key
DB_HOST=localhost
```

Example usage in Python:

```python
import os
from dotenv import load_dotenv

load_dotenv()

app_env = os.getenv("APP_ENV")
api_key = os.getenv("API_KEY")
print(app_env, api_key)
```

Why this matters:
- keeps secrets out of source code,
- makes local development easier,
- works well with deployment platforms and containers,
- is the most common lightweight config pattern in Python apps.

Java comparison:
- conceptually similar to externalized config in Spring Boot or reading system environment variables,
- but in Python, `.env` + `os.getenv(...)` is one of the most common lightweight defaults.

```python
import configparser

config = configparser.ConfigParser()
config["app"] = {"debug": "true", "port": "8080"}
print(config["app"]["port"])
```

### Pickle

```python
import pickle

payload = {"x": 1, "y": 2}
binary = pickle.dumps(payload)
restored = pickle.loads(binary)
print(restored)
```

### Important warning

`pickle` is convenient, but you should **not** load untrusted pickle data.
It is not a safe cross-system interchange format like JSON.

## 8. Math, Random, and Statistics

### `math`

```python
import math

print(math.sqrt(16))
print(math.ceil(2.3))
print(math.pi)
```

### `random`

```python
import random

print(random.randint(1, 10))
print(random.choice(["a", "b", "c"]))
```

### `statistics`

```python
import statistics

values = [1, 2, 3, 4, 5]
print(statistics.mean(values))
print(statistics.median(values))
```

### Java comparison

These replace a mix of:
- `Math`
- `Random`
- utility code you might otherwise write by hand

## 9. Logging, Debugging, and Runtime Inspection

### Logging

```python
import logging

logging.basicConfig(level=logging.INFO)
logging.info("Application started")
```

### Runtime inspection

```python
value = [1, 2, 3]
print(type(value))
print(isinstance(value, list))
```

### Java comparison

Python's built-in `logging` is often enough for small-to-medium applications.
For introspection, Python leans heavily on runtime type inspection compared with Java's stronger compile-time culture.

## 10. Concurrency-Related Standard Library Modules

If you are coming from `java.util.concurrent`, Python spreads similar ideas across multiple modules:

- `threading`
- `queue`
- `concurrent.futures`
- `multiprocessing`
- `asyncio`

This means the equivalent of Java's concurrency toolbox exists, but not under one single package umbrella.

## 11. Practical Takeaways for Java Developers

- Think of Python's standard library as **many small focused modules**, not one giant class hierarchy.
- Learn `pathlib`, `datetime`, `json`, `collections`, and `logging` early.
- Prefer built-ins first; many common tasks do not need external dependencies.
- When you think "Where is the Java equivalent package?", the Python answer is often: built-in type + small module.

## 12. Final Summary

Part 4 is about finding your bearings inside Python's standard library.
For a Java developer, the key shift is this:

- Java often organizes capability into class-heavy packages.
- Python often gives you a mix of built-in types, built-in functions, and small standard-library modules.

Once you know the rough mapping from `java.lang`, `java.util`, `java.time`, `java.nio.file`, and `java.util.concurrent` to Python's built-in and standard library ecosystem, day-to-day Python becomes much easier to navigate.
