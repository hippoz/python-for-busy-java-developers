# Part 5 — Standard Library

Goal: know the stdlib map well enough to reach for the right module before installing a third-party package. "Just `import`" — no venv knowledge required.

**Prerequisites:** [Part 1 — Syntax Shock](01_syntax_shock.md). **Next:** [Part 6 — Ecosystem & Packaging](06_ecosystem_and_packaging.md).

---

## Table of Contents

- [Core differences](#core-differences)
- [Stdlib philosophy](#stdlib-philosophy)
- [Collections module](#collections-module)
- [Datetime](#datetime)
- [Date and time parsing](#date-and-time-parsing)
- [Pathlib](#pathlib)
- [Cross-platform path notes](#cross-platform-path-notes)
- [Regex and strings](#regex-and-strings)
- [JSON](#json)
- [Configparser](#configparser)
- [Env vars](#env-vars)
- [Pickle](#pickle)
- [Math, random, stats](#math-random-stats)
- [Decimal](#decimal)
- [Logging](#logging)
- [Structured logging notes](#structured-logging-notes)
- [Stdlib quick-wins](#stdlib-quick-wins)
- [Urllib.parse](#urllibparse)
- [Key Takeaways](#key-takeaways)

---

## Core differences

| Java area | Common Java packages/classes | Python counterpart |
| :--- | :--- | :--- |
| Core utilities | `java.lang` | built-ins + small stdlib modules |
| Collections | `java.util` | `dict`, `list`, `set`, `tuple` + `collections` |
| Dates and time | `java.time` | `datetime`, `zoneinfo` |
| Filesystem | `java.nio.file` | `pathlib`, `os`, `shutil` |
| JSON | Jackson / Gson | `json` (stdlib) |
| Math / random | `Math`, `Random` | `math`, `random`, `statistics` |
| Regex | `java.util.regex` | `re` |
| Logging | `java.util.logging`, Log4j, SLF4J | `logging` |
| Concurrency helpers | `java.util.concurrent` | `threading`, `queue`, `concurrent.futures`, `asyncio` (see [Part 4](04_concurrency.md)) |

```java
import java.time.LocalDateTime;
System.out.println(LocalDateTime.now());
```

```python
from datetime import datetime
print(datetime.now())
```

## Stdlib philosophy

Java tends to organize functionality into class-heavy packages (`java.util.*`, `java.nio.file.*`). Python spreads similar functionality across:

- built-in **types** with methods on them (`str.upper()`, `list.append(...)`)
- built-in **functions** (`len`, `print`, `type`, `isinstance`, `sorted`, `enumerate`)
- small **stdlib modules** (`math`, `re`, `pathlib`, `json`)

The day-to-day implication: when you wonder "where's the Java equivalent of `X`?", the Python answer is often "the built-in type already has it" or "a small focused module exists." The standard library is one of Python's strongest selling points — make a habit of checking it before installing anything.

## Collections module

The built-in `dict` / `list` / `set` / `tuple` cover most needs. For specialized patterns, `collections` ships ready-made data structures.

**`Counter`** — frequency map:

```python
from collections import Counter

counts = Counter(["a", "b", "a", "c", "a"])
print(counts)                       # >>> Counter({'a': 3, 'b': 1, 'c': 1})
print(counts.most_common(2))        # >>> [('a', 3), ('b', 1)]
```

**`defaultdict`** — auto-creates missing keys with a factory:

```python
from collections import defaultdict

groups = defaultdict(list)
groups["fruit"].append("apple")
groups["fruit"].append("banana")
print(groups)                       # >>> defaultdict(<class 'list'>, {'fruit': ['apple', 'banana']})
```

**`deque`** — double-ended queue with O(1) appends/pops from both ends:

```python
from collections import deque

q = deque([1, 2, 3])
q.appendleft(0)
q.append(4)
print(q)                            # >>> deque([0, 1, 2, 3, 4])
q.popleft()                         # O(1) — unlike list.pop(0) which is O(n)
```

> ☕ **Java parallel:** `Counter` ≈ `Map<K,Integer>` frequency counts; `defaultdict(list)` ≈ `Map<K,List<V>>` with `computeIfAbsent`; `deque` ≈ `ArrayDeque`.

Other useful ones: `OrderedDict` (rarely needed since regular `dict` is now ordered), `namedtuple` (rarely needed since `@dataclass` exists), `ChainMap` (search across multiple dicts).

## Datetime

The modern Python date/time API lives in `datetime` + `zoneinfo` (3.9+).

```python
from datetime import datetime, date, timedelta
from zoneinfo import ZoneInfo

print(datetime.now())                       # naive local datetime
print(date.today())                         # date only
print(datetime.now() + timedelta(days=7))   # arithmetic with timedelta

# Timezone-aware
now_utc = datetime.now(ZoneInfo("UTC"))
now_tokyo = now_utc.astimezone(ZoneInfo("Asia/Tokyo"))
print(now_tokyo)
```

> ☕ **Java parallel:** `datetime.datetime` ≈ `LocalDateTime` (naive) / `ZonedDateTime` (when tz-aware); `datetime.date` ≈ `LocalDate`; `timedelta` ≈ `Duration` (`Period` for date-only). `zoneinfo.ZoneInfo` replaced the third-party `pytz` library.

> ⚠️ **Pitfall:** `datetime.now()` returns a **naive** datetime (no timezone). Naive and aware datetimes can't be compared or subtracted from each other — `TypeError`. Default to timezone-aware (`datetime.now(ZoneInfo("UTC"))`) in any service code.

## Date and time parsing

`datetime` constructors are for building — parsing strings into datetimes is a separate concern that bites everyone eventually.

**ISO-8601 input** — `fromisoformat` (preferred, fast, no format string):

```python
from datetime import datetime

dt = datetime.fromisoformat("2026-05-23T14:30:00+00:00")
print(dt)                                  # >>> 2026-05-23 14:30:00+00:00
print(dt.tzinfo)                           # >>> UTC
```

**Arbitrary format** — `strptime` (with explicit format string):

```python
dt = datetime.strptime("23 May 2026 14:30", "%d %b %Y %H:%M")
print(dt)                                  # >>> 2026-05-23 14:30:00
```

**Formatting back to string** — `strftime`:

```python
print(dt.strftime("%Y-%m-%d %H:%M"))       # >>> 2026-05-23 14:30
print(dt.isoformat())                      # >>> 2026-05-23T14:30:00
```

> ⚠️ **Pitfall:** `fromisoformat` on a string without timezone offset returns a **naive** datetime. Decide your policy explicitly: either *reject* naive inputs (`if dt.tzinfo is None: raise ValueError(...)`), or *assert* a known source zone (`dt = dt.replace(tzinfo=ZoneInfo("UTC"))` — note this attaches a label, it doesn't *convert*). Use `.astimezone(...)` after that if you need to convert.

> ☕ **Java parallel:** `strptime` ≈ `DateTimeFormatter.parse(s, formatter)`. `fromisoformat` ≈ `OffsetDateTime.parse(s)` for ISO inputs.

## Pathlib

`pathlib` is the modern, object-oriented filesystem API. Prefer it over `os.path` for new code:

```python
from pathlib import Path

p = Path("logs") / "2026" / "app.log"       # use / operator, NOT string concat
print(p.name)                                # >>> app.log
print(p.suffix)                              # >>> .log
print(p.stem)                                # >>> app
print(p.parent)                              # >>> logs/2026

# Read/write text
p.write_text("hello\n")
print(p.read_text())                         # >>> hello
```

**Create directories:**

```python
Path("output/reports").mkdir(parents=True, exist_ok=True)
```

`parents=True` creates intermediate dirs; `exist_ok=True` doesn't error if it already exists. The combination is the day-to-day pattern.

**File handling** — `with open(...)` is still the right tool for streaming reads:

```python
with open("data.csv") as f:
    for line in f:                          # streamed, one line at a time
        print(line, end="")                 # process however you need
```

For the small-file case, `Path.read_text()` / `Path.write_text()` are shorter. For large files, the streamed `with open(...)` form keeps memory bounded.

**Copy/move:**

```python
import shutil
shutil.copy("source.txt", "backup.txt")     # copies file contents + permissions
shutil.move("a.txt", "b.txt")               # move/rename
```

> ☕ **Java parallel:** `Path` / `Files` / `try-with-resources` → `pathlib.Path` + `with open(...)` + `shutil`. Same shape, less ceremony.

## Cross-platform path notes

`pathlib.Path` handles platform differences for you — **don't** string-concatenate with `/` or `\` yourself:

```python
from pathlib import Path

# Right — works on Windows, Mac, Linux
log = Path("logs") / "app.log"

# Wrong — breaks on Windows
log = "logs" + "/" + "app.log"
```

Useful built-ins:

```python
Path.home()                              # user home directory
Path.cwd()                               # current working directory
p.resolve()                              # absolute, with symlinks resolved
p.absolute()                             # absolute, no symlink resolution
p.exists() / p.is_file() / p.is_dir()    # checks
p.glob("*.py") / p.rglob("*.py")         # glob within / recursively
```

For temp files/dirs, **never** roll your own under `/tmp` — use `tempfile`:

```python
import tempfile

with tempfile.TemporaryDirectory() as td:
    # td is auto-cleaned up on exit
    work_path = Path(td) / "scratch.txt"
    work_path.write_text("hello")
```

`tempfile` knows the right location on every OS (Windows uses something different from `/tmp`) and handles cleanup atomically.

## Regex and strings

String methods cover the common cases:

```python
text = "  hello, world  "
print(text.strip())                      # >>> hello, world
print(text.split(","))                   # >>> ['  hello', ' world  ']
print(text.replace("world", "python"))   # >>> '  hello, python  '
print("alice".startswith("ali"))         # >>> True
print(",".join(["a", "b", "c"]))         # >>> a,b,c
```

For pattern matching, `re`:

```python
import re

text = "Order ID: 12345"
m = re.search(r"\d+", text)
print(m.group())                         # >>> 12345

# Compile once, reuse — faster in loops
ID_PATTERN = re.compile(r"order-(\d+)")
lines = ["order-100 created", "no match here", "order-201 deleted"]
for line in lines:
    if (m := ID_PATTERN.search(line)):
        order_id = m.group(1)
```

> ☕ **Java parallel:** `String` methods overlap with Python `str` methods; `java.util.regex.Pattern`/`Matcher` ≈ `re.compile(...)` + `match`/`search`/`findall`/`sub`. Basic literals, character classes, and capture groups translate cleanly; named groups (`(?P<name>...)` in Python vs `(?<name>...)` in Java), Unicode property classes, and some advanced look-around constructs differ — verify non-trivial patterns before assuming they port.

Use string methods for fixed strings (`startswith`, `endswith`, `in`, `split`). Use `re` for actual patterns. Avoid regexes for trivial fixed-string work — it's slower and less clear.

## JSON

```python
import json

# Encode / decode (in-memory)
data = {"name": "Alice", "age": 25}
text = json.dumps(data)                  # str
text_pretty = json.dumps(data, indent=2, ensure_ascii=False)
parsed = json.loads(text)                # back to dict

# Read / write files
with open("input.json") as f:
    data = json.load(f)

with open("output.json", "w") as f:
    json.dump(data, f, indent=2)
```

**`ensure_ascii=False`** is what you want when serializing non-ASCII text (it preserves Unicode characters rather than escaping them to `\uXXXX`).

**Custom serialization** for non-JSON-native types — pass a `default` function:

```python
from datetime import datetime
import json

def encode(obj):
    if isinstance(obj, datetime):
        return obj.isoformat()
    raise TypeError(f"unserializable: {type(obj)}")

print(json.dumps({"now": datetime.now()}, default=encode))
```

> ⚠️ **Pitfall:** Stdlib `json` does **no schema validation**. It happily produces `{"age": "twenty-five"}` from any dict, and parses any JSON into untyped `dict`/`list`/`str`/`int`/`float`/`bool`/`None`. For typed validation, see `pydantic` in [Part 6 § Productivity libs](06_ecosystem_and_packaging.md#productivity-libs).

> ☕ **Java parallel:** Stdlib `json` ≈ Jackson/Gson for read/write. For validating object-mapping (DTO-style with field validators), the Python equivalent is `pydantic`, not stdlib `json`.

## Configparser

For INI-style configuration files:

```python
import configparser

config = configparser.ConfigParser()
config["app"] = {"debug": "true", "port": "8080"}

with open("app.ini", "w") as f:
    config.write(f)

config.read("app.ini")
print(config["app"]["port"])             # >>> 8080
```

Useful for legacy INI files. For new projects, prefer `pyproject.toml` for tool config and environment-variable-driven settings (see [Part 6 § Settings management](06_ecosystem_and_packaging.md#settings-management)) for runtime config.

## Env vars

```python
import os

# Required: KeyError if missing
api_key = os.environ["API_KEY"]

# Optional with default
log_level = os.environ.get("LOG_LEVEL", "INFO")
log_level = os.getenv("LOG_LEVEL", "INFO")   # same thing, shorter
```

For local development, `.env` files are the common pattern — but loading them needs the third-party `python-dotenv`. See [Part 6 § Env config](06_ecosystem_and_packaging.md#env-config). For typed config with validation, see [Part 6 § Settings management](06_ecosystem_and_packaging.md#settings-management).

## Pickle

`pickle` serializes Python objects to bytes. Handy, but treat it as **dangerous on untrusted input**:

```python
import pickle

payload = {"x": 1, "y": [2, 3]}
binary = pickle.dumps(payload)
restored = pickle.loads(binary)
print(restored)                          # >>> {'x': 1, 'y': [2, 3]}
```

> ⚠️ **Pitfall:** Loading a pickle from an untrusted source can execute arbitrary code. `pickle.loads(network_data)` is a remote-code-execution sink. Use JSON for cross-system data. Use `pickle` only for trusted intra-process / intra-team caches.

`pickle` is also Python-specific — not a cross-language interchange format.

## Math, random, stats

```python
import math
print(math.sqrt(16))                     # >>> 4.0
print(math.ceil(2.3))                    # >>> 3
print(math.pi)                           # >>> 3.141592653589793
print(math.isclose(0.1 + 0.2, 0.3))      # >>> True   (float-safe equality)

import random
print(random.randint(1, 10))             # inclusive both ends
print(random.choice(["a", "b", "c"]))
print(random.sample(range(100), 5))      # k items without replacement
my_list = [1, 2, 3, 4, 5]
random.shuffle(my_list)                  # in place

# For cryptographic randomness, use SystemRandom or `secrets`:
import secrets
token = secrets.token_urlsafe(16)

import statistics
values = [1, 2, 3, 4, 5]
print(statistics.mean(values))           # >>> 3
print(statistics.median(values))         # >>> 3
print(statistics.stdev(values))          # sample stdev
```

> ⚠️ **Pitfall:** `random` is NOT cryptographically secure. Use `secrets` for tokens, passwords, session IDs, and anything an attacker shouldn't be able to predict.

> ☕ **Java parallel:** `Math` → `math`; `Random` → `random`; `SecureRandom` → `secrets` (or `random.SystemRandom()`).

## Decimal

`float` arithmetic is binary — `0.1 + 0.2 != 0.3` ([this is universal across languages](https://0.30000000000000004.com/), not a Python quirk). For money, accounting, or anywhere precision matters, use `decimal.Decimal`:

```python
from decimal import Decimal, getcontext, ROUND_HALF_UP

a = Decimal("0.1")
b = Decimal("0.2")
print(a + b)                             # >>> 0.3        (exact)

# Set precision and rounding
getcontext().prec = 6
getcontext().rounding = ROUND_HALF_UP

price = Decimal("19.995")
print(round(price, 2))                   # >>> 20.00
```

> ⚠️ **Pitfall:** Construct `Decimal` from a **string**, not a `float`. `Decimal(0.1)` gives you the binary float's true value (`0.1000000000000000055511151231257827021181583404541015625`), defeating the point. `Decimal("0.1")` is exactly 0.1.

> ☕ **Java parallel:** `BigDecimal`. Same rules: construct from string, set scale/rounding explicitly for financial code.

For arbitrary-precision integers, you already have `int` — Python ints are unbounded natively. (`BigInteger` has no separate Python type; it's just `int`.)

## Logging

The stdlib `logging` module is hierarchical, configurable, and good enough for most projects.

**Minimal setup with `basicConfig`:**

```python
import logging

logging.basicConfig(
    level=logging.INFO,
    format="%(asctime)s %(levelname)s %(name)s: %(message)s",
)

logger = logging.getLogger(__name__)
logger.info("Application started")
logger.warning("Something looks off")
```

> 💡 **Pythonic:** Use `logger.info("user %s logged in", username)` — not `logger.info(f"user {username} logged in")`. The deferred-format form skips the work when the log level is suppressed; the f-string formats unconditionally.

**Per-logger levels** — silence noisy libraries, set your own code to DEBUG:

```python
logging.getLogger("urllib3").setLevel(logging.WARNING)
logging.getLogger("myapp").setLevel(logging.DEBUG)
```

**Production-grade configuration** — use `dictConfig`. `basicConfig` is for quick scripts and only works once per process; `dictConfig` lets you wire handlers, formatters, and per-logger overrides explicitly:

```python
import logging.config

logging.config.dictConfig({
    "version": 1,
    "disable_existing_loggers": False,
    "formatters": {
        "default": {
            "format": "%(asctime)s [%(levelname)s] %(name)s: %(message)s",
        },
    },
    "handlers": {
        "console": {
            "class": "logging.StreamHandler",
            "formatter": "default",
            "level": "INFO",
        },
        "file": {
            "class": "logging.handlers.RotatingFileHandler",
            "filename": "app.log",
            "maxBytes": 10_000_000,
            "backupCount": 5,
            "formatter": "default",
            "level": "DEBUG",
        },
    },
    "root": {
        "level": "INFO",
        "handlers": ["console", "file"],
    },
    "loggers": {
        "urllib3": {"level": "WARNING"},
        "myapp":   {"level": "DEBUG"},
    },
})
```

`RotatingFileHandler` (or `TimedRotatingFileHandler`) is what you want for any long-running process — plain `FileHandler` will fill the disk eventually.

> ☕ **Java parallel:** `dictConfig` is the equivalent of a Log4j/Logback XML config — declarative, single source of truth, applied at startup. `basicConfig` is the equivalent of one-line "just print to stderr."

## Structured logging notes

Modern services typically emit **JSON logs** that go into a log aggregator (ELK, Loki, Datadog, CloudWatch). The stdlib can do this with a custom `Formatter`:

```python
import json
import logging

class JsonFormatter(logging.Formatter):
    def format(self, record):
        payload = {
            "ts": self.formatTime(record),
            "level": record.levelname,
            "logger": record.name,
            "message": record.getMessage(),
        }
        # Include any extras passed via logger.info("...", extra={...})
        if hasattr(record, "request_id"):
            payload["request_id"] = record.request_id
        return json.dumps(payload)

handler = logging.StreamHandler()
handler.setFormatter(JsonFormatter())
logging.getLogger().addHandler(handler)
logging.getLogger().setLevel(logging.INFO)
```

For richer ergonomics — structured context, contextvars integration out of the box, processor pipelines — use `structlog` (third-party). Most teams adopt `structlog` once log volume justifies the dependency.

**Request/correlation IDs** propagate via `contextvars.ContextVar` + a `logging.Filter`. The full pattern is shown in [Part 4 § Async-aware logging](04_concurrency.md#async-aware-logging). It's the same pattern for sync code — `ContextVar` works correctly with threads too.

> ☕ **Java parallel:** JSON logs ≈ Logback `JsonLayout`. Correlation-ID via `ContextVar` + `Filter` ≈ SLF4J MDC.

## Stdlib quick-wins

A handful of stdlib modules that punch above their weight. One paragraph each — reach for these before installing a third-party library.

**`itertools`** — iterator combinators. `chain` (concatenate iterables), `groupby` (group consecutive equal items), `islice` (lazy slice), `product` (Cartesian product), `combinations`, `permutations`, `accumulate` (running totals), `count` / `cycle` / `repeat` (infinite iterators):

```python
from itertools import chain, islice
print(list(chain([1, 2], [3, 4])))           # >>> [1, 2, 3, 4]
print(list(islice(range(1_000_000), 5)))     # >>> [0, 1, 2, 3, 4]
```

**`functools` (non-decorator)** — `partial` (pre-fill arguments), `reduce`, `singledispatch` (function overloading by argument type):

```python
from functools import partial
add5 = partial(lambda x, y: x + y, 5)
print(add5(3))                               # >>> 8
```

**`tempfile`** — secure temp files and dirs (see [Cross-platform path notes](#cross-platform-path-notes)). `TemporaryDirectory`, `NamedTemporaryFile`, `mkstemp`.

**`csv`** — built-in CSV reader/writer:

```python
import csv

# newline="" is the stdlib-recommended way to open CSV files —
# the csv module handles line endings internally; OS-level newline
# translation will corrupt embedded newlines (especially on Windows).
with open("data.csv", newline="") as f:
    reader = csv.DictReader(f)
    for row in reader:
        print(row["name"], row["age"])
```

For richer types (numbers, dates), bring in `pandas` ([Part 6](06_ecosystem_and_packaging.md#numerical-and-data-libs)). For simple structured data, `csv.DictReader` is enough.

**`argparse`** — declarative CLI argument parsing:

```python
import argparse
parser = argparse.ArgumentParser()
parser.add_argument("--port", type=int, default=8080)
parser.add_argument("input", help="input file path")
args = parser.parse_args()
print(args.port, args.input)
```

For richer CLIs (subcommands, colored output, autocompletion), reach for `typer` ([Part 6](06_ecosystem_and_packaging.md#productivity-libs)). `argparse` is fine when typer is overkill.

**`subprocess`** — run external commands:

```python
import subprocess

# Right — list form, no shell
result = subprocess.run(
    ["git", "log", "-n", "1"],
    capture_output=True,
    text=True,
    check=True,                          # raise on non-zero exit
)
print(result.stdout)
```

> ⚠️ **Pitfall:** Prefer the **list form** for the command. **Never** use `shell=True` with any user-controlled input — it's a shell-injection vector. If you genuinely need shell features (pipes, redirects), construct the command carefully and validate inputs.

For a teaser of `contextlib.ExitStack` (dynamic context-manager composition), see [Part 3 § Context managers](03_pythonic_idioms.md#context-managers).

## Urllib.parse

Don't string-concatenate URLs. Use `urllib.parse`:

```python
from urllib.parse import urlencode, urljoin, urlparse, urlunparse

# Build a query string
params = {"q": "python java", "page": 2}
print(urlencode(params))                # >>> q=python+java&page=2

# Safe URL joining (respects relative vs absolute)
print(urljoin("https://api.example.com/v1/", "users"))
# >>> https://api.example.com/v1/users
print(urljoin("https://api.example.com/v1/users/42", "/health"))
# >>> https://api.example.com/health

# Deconstruct a URL
p = urlparse("https://example.com:8080/path?a=1#frag")
print(p.scheme, p.hostname, p.port, p.path)
# >>> https example.com 8080 /path
```

> ☕ **Java parallel:** `java.net.URI` / `URL` / `UriBuilder`. Same role: avoid hand-crafted URL strings, get escaping right.

## Key Takeaways

- Prefer stdlib first — most "I need a library for X" needs are already `import X`.
- `pathlib` over `os.path`; use `/` over string concat; `tempfile` for any temp file/dir.
- `decimal.Decimal` for money — never floats; always construct from a string.
- Default to timezone-aware datetimes; `fromisoformat` for ISO inputs, `strptime` for arbitrary formats.
- `dictConfig` once you outgrow `basicConfig`; JSON logs via custom `Formatter` or `structlog`.
- Request/correlation IDs propagate via `contextvars` + a `logging.Filter` (the MDC pattern).
- `pickle` is unsafe for untrusted input — use JSON for cross-system data.
- `subprocess` with list-form args, never `shell=True` + untrusted input.
