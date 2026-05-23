# Python for Busy Java Developers

A 6-part doc set that helps an experienced Java engineer become productive in Python without writing Java-in-Python.

---

## Who this is for

You write Java daily. You need to read, contribute to, or own Python code — for tooling, for data work, for a service rewrite, or because the team you joined uses Python. You don't have time for a from-scratch tutorial. You want to know what's the same, what's different, and where Java instincts will mislead you. This doc set is calibrated for that.

## How to read this

Two paths through the material:

- **Fast path (~1 day):** [Part 1](01_syntax_shock.md) → [Part 3](03_pythonic_idioms.md). You can read Python code and write idiomatic Python.
- **Deep path (~1 week):** [Part 1](01_syntax_shock.md) → [Part 2](02_java_idiom_translation.md) → [Part 3](03_pythonic_idioms.md) → [Part 4](04_concurrency.md) → [Part 5](05_standard_library.md) → [Part 6](06_ecosystem_and_packaging.md). You can ship Python services.

**Prerequisite chart:**

| If you read… | You should have read first… |
| :--- | :--- |
| Part 2 | Part 1 |
| Part 3 | Part 1 |
| Part 4 | Part 3 (generators, iterables, context managers all feed coroutines) |
| Part 5 | Part 1 |
| Part 6 | Part 5 (stdlib-first reflex) |

## The parts

1. **[Syntax Shock](01_syntax_shock.md)** — survive reading Python code
2. **[Java Idiom Translation](02_java_idiom_translation.md)** — translate Java reflexes (OOP, equals/hash, collections)
3. **[Pythonic Idioms](03_pythonic_idioms.md)** — stop writing Java in Python
4. **[Concurrency](04_concurrency.md)** — pick the right model
5. **[Standard Library](05_standard_library.md)** — stdlib: just import
6. **[Ecosystem & Packaging](06_ecosystem_and_packaging.md)** — venv, pip, third-party
7. **[Appendix: Java → Python Lookup](99_appendix_java_to_python.md)** — single-table reference

## Topic map

Alphabetical. Sub-bullets indicate related/grouped topics. Click through to anchors.

- **`__init_subclass__`** — [P2 § Init subclass](02_java_idiom_translation.md#init-subclass)
- **`__post_init__`** — [P2 § Dataclass](02_java_idiom_translation.md#dataclass)
- **`__slots__`** — [P2 § Slots](02_java_idiom_translation.md#slots)
- **Abstract classes (`abc.ABC`)** — [P2 § Abstract classes](02_java_idiom_translation.md#abstract-classes)
- **Access conventions (`_x`, `__x`)** — [P2 § Access conventions](02_java_idiom_translation.md#access-conventions)
- **Alembic (migrations)** — [P6 § Database access](06_ecosystem_and_packaging.md#database-access)
- **`*args` / `**kwargs`** — [P3 § Args and kwargs](03_pythonic_idioms.md#args-and-kwargs)
- **`assert`** — [P3 § Advanced control flow](03_pythonic_idioms.md#advanced-control-flow)
- **`async` / `await`** — [P4 § Async and await](04_concurrency.md#async-and-await)
- **Async context managers (`async with`/`async for`)** — [P4 § Async context managers](04_concurrency.md#async-context-managers)
- **Async file I/O (`aiofiles`)** — [P4 § Async file I/O](04_concurrency.md#async-file-io)
- **Async-aware logging (MDC equivalent)** — [P4 § Async-aware logging](04_concurrency.md#async-aware-logging)
- **Auth and security (PyJWT/passlib/Authlib)** — [P6 § Auth and security](06_ecosystem_and_packaging.md#auth-and-security)
- **`BigDecimal` parallel (`decimal.Decimal`)** — [P5 § Decimal](05_standard_library.md#decimal)
- **Binary types (`bytes`/`bytearray`/`memoryview`)** — [P2 § Binary types](02_java_idiom_translation.md#binary-types)
- **`black` (formatter, subsumed by `ruff format` in 2026)** — [P6 § Tooling](06_ecosystem_and_packaging.md#tooling)
- **Cancellation and timeouts (`asyncio.timeout`)** — [P4 § Cancellation and timeouts](04_concurrency.md#cancellation-and-timeouts)
- **Class attr vs instance attr** — [P2 § Class vs instance attributes](02_java_idiom_translation.md#class-vs-instance-attributes)
- **`@classmethod` / `@staticmethod`** — [P2 § Class and static methods](02_java_idiom_translation.md#class-and-static-methods)
- **Collections — built-ins (`dict`/`set`/`list`/`tuple`/`deque`)** — [P2 § Collections mapping](02_java_idiom_translation.md#collections-mapping)
- **`collections` module (Counter/defaultdict/deque)** — [P5 § Collections module](05_standard_library.md#collections-module)
- **`complex`** — [P1 § Complex](01_syntax_shock.md#complex)
- **Comprehensions (list/dict/set/generator)** — [P3 § Comprehensions](03_pythonic_idioms.md#comprehensions)
- **Composition over inheritance** — [P2 § Composition over inheritance](02_java_idiom_translation.md#composition-over-inheritance)
- **Concurrency chooser (threads/async/multiprocessing)** — [P4 § Concurrency chooser](04_concurrency.md#concurrency-chooser)
- **`configparser`** — [P5 § Configparser](05_standard_library.md#configparser)
- **Context managers (`with`, `@contextmanager`, `ExitStack`)** — [P3 § Context managers](03_pythonic_idioms.md#context-managers)
- **`contextvars.ContextVar` (async-safe ThreadLocal)** — [P4 § Contextvars](04_concurrency.md#contextvars)
- **Coroutines** — [P4 § Coroutines](04_concurrency.md#coroutines)
- **Cross-platform paths** — [P5 § Cross-platform path notes](05_standard_library.md#cross-platform-path-notes)
- **`@dataclass` (`frozen`/`order`/`__post_init__`)** — [P2 § Dataclass](02_java_idiom_translation.md#dataclass)
- **Database access (SQLAlchemy/Alembic/async drivers)** — [P6 § Database access](06_ecosystem_and_packaging.md#database-access)
- **Date and time (`datetime`/`zoneinfo`/`timedelta`)** — [P5 § Datetime](05_standard_library.md#datetime)
- **Date parsing (`fromisoformat`/`strptime`)** — [P5 § Date and time parsing](05_standard_library.md#date-and-time-parsing)
- **`decimal.Decimal`** — [P5 § Decimal](05_standard_library.md#decimal)
- **Decorators (incl. `@cache`/`@lru_cache`/`@cached_property`)** — [P3 § Decorators](03_pythonic_idioms.md#decorators)
- **DI (`Depends`/constructor injection)** — [P6 § Middleware and DI](06_ecosystem_and_packaging.md#middleware-and-di)
- **`dict` vs class** — [P2 § Dict vs class](02_java_idiom_translation.md#dict-vs-class)
- **Dunder methods** — [P2 § Dunder methods](02_java_idiom_translation.md#dunder-methods)
- **Encapsulation (`@property`)** — [P2 § Encapsulation](02_java_idiom_translation.md#encapsulation)
- **`enum` (`Enum`/`IntEnum`/`StrEnum`)** — [P2 § Enum](02_java_idiom_translation.md#enum)
- **Environment variables (`os.environ`)** — [P5 § Env vars](05_standard_library.md#env-vars)
- **`.env` files (`python-dotenv`)** — [P6 § Env config](06_ecosystem_and_packaging.md#env-config)
- **`ExceptionGroup` / `except*`** — [P4 § Exception groups](04_concurrency.md#exception-groups)
- **Exception handling (`try`/`except`/`raise from`)** — [P1 § Exception handling](01_syntax_shock.md#exception-handling)
- **`ExitStack`** — [P3 § Context managers](03_pythonic_idioms.md#context-managers)
- **f-strings** — [P1 § F-strings](01_syntax_shock.md#f-strings)
- **FastAPI** — [P6 § Web frameworks](06_ecosystem_and_packaging.md#web-frameworks)
- **File I/O** — [P5 § Pathlib](05_standard_library.md#pathlib)
- **Flask** — [P6 § Web frameworks](06_ecosystem_and_packaging.md#web-frameworks)
- **`frozenset`** — [P2 § Frozenset](02_java_idiom_translation.md#frozenset)
- **Functions** — [P1 § Functions](01_syntax_shock.md#functions)
- **Functions as first-class objects (lambdas, callables)** — [P3 § Functions as first-class objects](03_pythonic_idioms.md#functions-as-first-class-objects)
- **`functools.cache` / `lru_cache` / `cached_property`** — [P3 § Decorators](03_pythonic_idioms.md#decorators)
- **`asyncio.gather` vs `TaskGroup`** — [P4 § Gather vs TaskGroup](04_concurrency.md#gather-vs-taskgroup)
- **Generators (`yield`/`yield from`/generator expressions)** — [P3 § Generators](03_pythonic_idioms.md#generators)
- **GIL (incl. 3.13+ free-threaded)** — [P4 § GIL](04_concurrency.md#gil)
- **HTTP clients (`requests`/`httpx`)** — [P6 § HTTP clients](06_ecosystem_and_packaging.md#http-clients)
- **HTTP production (timeouts/retries/`tenacity`)** — [P6 § HTTP production behavior](06_ecosystem_and_packaging.md#http-production-behavior)
- **Imports** — [P1 § Modules and imports](01_syntax_shock.md#modules-and-imports)
- **Introspection (`type`/`isinstance`/`getattr`/`inspect`)** — [P3 § Introspection](03_pythonic_idioms.md#introspection)
- **`is` vs `==`** — [P1 § None and is](01_syntax_shock.md#none-and-is)
- **Iterable vs iterator** — [P3 § Iterable vs iterator](03_pythonic_idioms.md#iterable-vs-iterator)
- **`itertools`** — [P5 § Stdlib quick-wins](05_standard_library.md#stdlib-quick-wins)
- **JSON** — [P5 § JSON](05_standard_library.md#json)
- **Jupyter / IPython** — [P6 § Jupyter and IPython](06_ecosystem_and_packaging.md#jupyter-and-ipython)
- **Legacy string formatting (`.format`/`%`)** — [P1 § Legacy string formatting](01_syntax_shock.md#legacy-string-formatting)
- **`logging` (basicConfig + dictConfig)** — [P5 § Logging](05_standard_library.md#logging)
- **`match` (basic)** — [P1 § Match](01_syntax_shock.md#match)
- **`match` patterns (deep)** — [P3 § Match patterns](03_pythonic_idioms.md#match-patterns)
- **`math` / `random` / `statistics`** — [P5 § Math, random, stats](05_standard_library.md#math-random-stats)
- **Memory visibility (vs JMM)** — [P4 § Memory visibility](04_concurrency.md#memory-visibility)
- **Metaprogramming** — [P3 § Metaprogramming](03_pythonic_idioms.md#metaprogramming)
- **Middleware and DI** — [P6 § Middleware and DI](06_ecosystem_and_packaging.md#middleware-and-di)
- **ML/AI libs (scikit-learn/PyTorch/TF)** — [P6 § ML/AI libs](06_ecosystem_and_packaging.md#mlai-libs)
- **Modules and imports** — [P1 § Modules and imports](01_syntax_shock.md#modules-and-imports)
- **MRO (Method Resolution Order)** — [P2 § MRO](02_java_idiom_translation.md#mro)
- **Mutability (incl. mutable default args)** — [P2 § Mutability](02_java_idiom_translation.md#mutability)
- **`nonlocal`** — [P3 § Scope and nonlocal](03_pythonic_idioms.md#scope-and-nonlocal)
- **`None`** — [P1 § None and is](01_syntax_shock.md#none-and-is)
- **NumPy** — [P6 § Numerical and data libs](06_ecosystem_and_packaging.md#numerical-and-data-libs)
- **OOP basics** — [P2 § OOP basics](02_java_idiom_translation.md#oop-basics)
- **pandas / Polars / SciPy** — [P6 § Numerical and data libs](06_ecosystem_and_packaging.md#numerical-and-data-libs)
- **`pathlib`** — [P5 § Pathlib](05_standard_library.md#pathlib)
- **`pickle` (with safety warning)** — [P5 § Pickle](05_standard_library.md#pickle)
- **Producer-consumer (`queue.Queue`/`asyncio.Queue`)** — [P4 § Producer-consumer](04_concurrency.md#producer-consumer)
- **`Protocol` (structural typing)** — [P3 § Protocol](03_pythonic_idioms.md#protocol)
- **`pydantic`** — [P6 § Productivity libs](06_ecosystem_and_packaging.md#productivity-libs)
- **`pydantic-settings`** — [P6 § Settings management](06_ecosystem_and_packaging.md#settings-management)
- **`pyproject.toml`** — [P6 § Pyproject](06_ecosystem_and_packaging.md#pyproject)
- **`pytest` (fixtures, parametrize, conftest)** — [P6 § Pytest](06_ecosystem_and_packaging.md#pytest)
- **`range`** — [P1 § Range](01_syntax_shock.md#range)
- **Regex (`re`)** — [P5 § Regex and strings](05_standard_library.md#regex-and-strings)
- **`requests` / `httpx`** — [P6 § HTTP clients](06_ecosystem_and_packaging.md#http-clients)
- **`ruff`** — [P6 § Tooling](06_ecosystem_and_packaging.md#tooling)
- **Scope and `nonlocal`** — [P3 § Scope and nonlocal](03_pythonic_idioms.md#scope-and-nonlocal)
- **Sequence utilities (`zip`/`any`/`all`/`sorted`/`reversed`)** — [P1 § Sequence utilities](01_syntax_shock.md#sequence-utilities)
- **Settings management (`pydantic-settings`)** — [P6 § Settings management](06_ecosystem_and_packaging.md#settings-management)
- **SQLAlchemy** — [P6 § Database access](06_ecosystem_and_packaging.md#database-access)
- **`str` vs `bytes`** — [P1 § Str vs bytes](01_syntax_shock.md#str-vs-bytes)
- **Structured logging** — [P5 § Structured logging notes](05_standard_library.md#structured-logging-notes)
- **`subprocess`** — [P5 § Stdlib quick-wins](05_standard_library.md#stdlib-quick-wins)
- **Sync primitives (Lock/RLock/Semaphore/Condition/Event/Barrier)** — [P4 § Sync primitives](04_concurrency.md#sync-primitives)
- **`TaskGroup` (structured concurrency)** — [P4 § Gather vs TaskGroup](04_concurrency.md#gather-vs-taskgroup)
- **`tempfile` / `csv` / `argparse`** — [P5 § Stdlib quick-wins](05_standard_library.md#stdlib-quick-wins)
- **Test doubles (`unittest.mock`/`pytest-mock`/`AsyncMock`)** — [P6 § Test doubles](06_ecosystem_and_packaging.md#test-doubles)
- **Thread pools (`ThreadPoolExecutor`)** — [P4 § Thread pools](04_concurrency.md#thread-pools)
- **Thread safety** — [P4 § Thread safety](04_concurrency.md#thread-safety)
- **Threading** — [P4 § Threading](04_concurrency.md#threading)
- **Tooling (`ruff`/`mypy`/`pyright`)** — [P6 § Tooling](06_ecosystem_and_packaging.md#tooling)
- **Truthiness** — [P1 § Truthiness](01_syntax_shock.md#truthiness)
- **Tuple** — [P2 § Tuple](02_java_idiom_translation.md#tuple)
- **Type hints (incl. `Protocol`/`TypedDict`/generics)** — [P3 § Type hints](03_pythonic_idioms.md#type-hints)
- **`urllib.parse`** — [P5 § Urllib.parse](05_standard_library.md#urllibparse)
- **`venv` and packaging** — [P6 § Venv and packaging](06_ecosystem_and_packaging.md#venv-and-packaging)
- **Visualization (matplotlib/seaborn/plotly)** — [P6 § Visualization](06_ecosystem_and_packaging.md#visualization)
- **Walrus `:=`** — [P1 § Walrus](01_syntax_shock.md#walrus)
- **Web frameworks (Flask/FastAPI/Django)** — [P6 § Web frameworks](06_ecosystem_and_packaging.md#web-frameworks)

## Java-term map

When you remember the Java class but not the Python equivalent — see **[Appendix: Java → Python Lookup](99_appendix_java_to_python.md)**. Single consolidated reference, organized by domain (Core language, Collections, Object model, Concurrency, Auth, Tooling, etc.).

## The Mindset Shift

Distilled from the per-part Key Takeaways — these are the cross-cutting shifts a Java developer needs to internalize. The first three are biggest.

- **Favor built-ins; reach for stdlib before third-party.** Python's standard library is a strong default. `pathlib`, `datetime`, `json`, `collections`, `logging`, `itertools` cover the vast majority of "where's the library for X?" questions. See [Part 5](05_standard_library.md).
- **Mutability is visible — design around it.** Mutable defaults, class-vs-instance attributes, shallow vs deep copy: Python makes reference semantics part of everyday code in a way Java masks behind types. See [P2 § Mutability](02_java_idiom_translation.md#mutability) and [P2 § Class vs instance attributes](02_java_idiom_translation.md#class-vs-instance-attributes).
- **Choose concurrency by I/O profile, not by habit.** Threads for blocking I/O. Async for non-blocking I/O. Multiprocessing for CPU-bound. The GIL is an interpreter detail, not a thread-safety guarantee. See [P4 § Concurrency chooser](04_concurrency.md#concurrency-chooser).
- **Composition > inheritance — even more than in Java.** Duck typing and `Protocol` mean you rarely *need* a shared base class. Reach for `@dataclass` + plain functions before reaching for an abstract base. See [P2 § Composition over inheritance](02_java_idiom_translation.md#composition-over-inheritance).
- **Type hints are optional but worth it — and aren't runtime validation.** Annotate, run `mypy` in CI, get IDE support. For runtime validation, use `pydantic`. They cover different concerns; both are good. See [P3 § Type hints](03_pythonic_idioms.md#type-hints) and [P6 § Pydantic](06_ecosystem_and_packaging.md#productivity-libs).
- **`str` ≠ `bytes`; encode/decode only at I/O boundaries.** Don't think of strings as bytes. Java did this transition; Python makes it visible. See [P1 § Str vs bytes](01_syntax_shock.md#str-vs-bytes).
- **Decorators are the cross-cutting tool.** `@functools.cache`, `@dataclass`, `@property`, `@retry`, `@pytest.fixture` — the AOP-style behavior you'd reach for an interceptor or annotation processor in Java is a decorator in Python. See [P3 § Decorators](03_pythonic_idioms.md#decorators).

## Conventions

This doc set uses four callout types:

- `> ☕ **Java parallel:** …` — quick Java analog for orientation
- `> ⚠️ **Pitfall:** …` — common Java-dev mistake
- `> 💡 **Pythonic:** …` — idiomatic Python preference
- `> 🐍 **Python 3.X+:** …` — version-gated feature

**Discipline rules:**
- At most one callout per `###` section under normal circumstances.
- Stacked callouts (two in a row) are discouraged but allowed when they address genuinely different angles — e.g., a `☕ Java parallel` followed by a `⚠️ Pitfall`, or a `⚠️ Pitfall` followed by a `🐍 Python 3.X+` version note. Each adds information the other doesn't.
- Teasers (forward pointers) are strictly one sentence + one link — no code, no bullets.

**Baseline:** Python 3.11. Features requiring 3.12+ are flagged with the `🐍` callout. Anything you write in 2026 should target 3.11 minimum.

**Anchors:** All section IDs are derived from the GitHub auto-slug of the heading. To deep-link to any section, click the section heading in GitHub's rendered view and copy the URL.

---

Ready? Start with **[Part 1 — Syntax Shock](01_syntax_shock.md)**.
