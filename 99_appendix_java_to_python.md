<!-- SKELETON — bodies filled in Phase 2 Batch 2E (LAST — depends on all anchor IDs being stable) -->

# Appendix — Java → Python Lookup

Single consolidated reference. When you remember the Java class but not the Python equivalent, look here.

For full treatment, follow the link in each row.

> 💡 **Pythonic note on mappings:** Java has rich class hierarchies; Python often spreads the same role across a built-in + a small module + a third-party lib. Some rows are 1:1; others are "closest equivalent" with caveats called out inline.

---

## Table of Contents

- [Core language](#core-language)
- [Collections](#collections)
- [Object model](#object-model)
- [Dates and time](#dates-and-time)
- [Numbers and precision](#numbers-and-precision)
- [Concurrency](#concurrency)
- [I/O and paths](#io-and-paths)
- [Exception model](#exception-model)
- [Caching and interception](#caching-and-interception)
- [Registry, DI, configuration](#registry-di-configuration)
- [Validation, serialization, ORM](#validation-serialization-orm)
- [Auth and security](#auth-and-security)
- [HTTP and URLs](#http-and-urls)
- [Streams and functional](#streams-and-functional)
- [Interop and integration](#interop-and-integration)
- [Testing](#testing)
- [Tooling](#tooling)

---

## Core language

| Java | Python | See |
|---|---|---|
| `String` | `str` | [Part 1 § str-vs-bytes](01_syntax_shock.md#str-vs-bytes) |
| triple-quoted string literal | `"""..."""` / `'''...'''` | [Part 1 § str-vs-bytes](01_syntax_shock.md#str-vs-bytes) |
| `Math` | `math` module | [Part 5 § math-random-stats](05_standard_library.md#math-random-stats) |
| `Math.pow(a, b)` | `a ** b` (built-in operator) | [Part 1 § operators](01_syntax_shock.md#operators) |
| `Math.floorMod(a, b)` | `a % b` (Python `%` already sign-follows-divisor) | [Part 1 § operators](01_syntax_shock.md#operators) |
| `a / b` (int division when ints) | `a // b` (floor division) | [Part 1 § operators](01_syntax_shock.md#operators) |
| `(double)a / b` | `a / b` (Python `/` is always float) | [Part 1 § operators](01_syntax_shock.md#operators) |
| `x++` / `x--` | `x += 1` / `x -= 1` (no `++`/`--` in Python) | [Part 1 § operators](01_syntax_shock.md#operators) |
| `cond ? a : b` (ternary) | `a if cond else b` | [Part 1 § control-flow](01_syntax_shock.md#control-flow) |
| chained `a < x && x < b` | `a < x < b` (true chaining) | [Part 1 § operators](01_syntax_shock.md#operators) |
| `null` | `None` | [Part 1 § none-and-is](01_syntax_shock.md#none-and-is) |
| `Object` | `object` | — |
| `Integer.parseInt(s)` | `int(s)` | — |
| `Double.parseDouble(s)` | `float(s)` | — |
| `Exception` | `Exception` | [Part 1 § exception-handling](01_syntax_shock.md#exception-handling) |
| `System.out.println(...)` | `print(...)` (with `sep=`, `end=`, `file=`) | [Part 1 § console-io](01_syntax_shock.md#console-io) |
| `Scanner.nextLine()` | `input(prompt)` (returns `str`; cast for numbers) | [Part 1 § console-io](01_syntax_shock.md#console-io) |
| `System.err.println(...)` | `print(..., file=sys.stderr)` | [Part 1 § console-io](01_syntax_shock.md#console-io) |

## Collections

| Java | Python | See |
|---|---|---|
| `HashMap<K,V>` | `dict` (insertion-ordered since 3.7) | [Part 2 § collections-mapping](02_java_idiom_translation.md#collections-mapping) |
| `LinkedHashMap` | `dict` (preserves order; same type) | [Part 2 § collections-mapping](02_java_idiom_translation.md#collections-mapping) |
| `TreeMap` | `dict` + `sorted(...)` at read time, or `sortedcontainers.SortedDict` (3rd-party) | [Part 2 § collections-mapping](02_java_idiom_translation.md#collections-mapping) |
| `HashSet<T>` | `set` | [Part 2 § collections-mapping](02_java_idiom_translation.md#collections-mapping) |
| `ArrayList<T>` | `list` | [Part 2 § collections-mapping](02_java_idiom_translation.md#collections-mapping) |
| `LinkedList<T>` used as `List` | `list` (Python `list` is array-backed; ~always what you want) | [Part 2 § collections-mapping](02_java_idiom_translation.md#collections-mapping) |
| `Deque<T>` / `ArrayDeque` / `LinkedList` used as queue | `collections.deque` | [Part 5 § collections-module](05_standard_library.md#collections-module) |
| immutable list/array | `tuple` | [Part 2 § tuple](02_java_idiom_translation.md#tuple) |
| `Collections.unmodifiableSet(s)` | `frozenset(s)` (true immutable copy; the unmodifiable wrapper reflects backing changes, `frozenset` does not — see notes below) | [Part 2 § frozenset](02_java_idiom_translation.md#frozenset) |
| `byte[]` | `bytes` (immutable) / `bytearray` (mutable) | [Part 2 § binary-types](02_java_idiom_translation.md#binary-types) |
| `Map<K,Integer>` frequency | `collections.Counter` | [Part 5 § collections-module](05_standard_library.md#collections-module) |
| `Map<K,List<V>>` | `collections.defaultdict(list)` | [Part 5 § collections-module](05_standard_library.md#collections-module) |
| `Map.entrySet()` / `keySet()` / `values()` | `d.items()` / `d.keys()` / `d.values()` (live views) | [Part 2 § collections-mapping](02_java_idiom_translation.md#collections-mapping) |
| `Map.getOrDefault(k, dflt)` | `d.get(k, dflt)` | [Part 2 § collections-mapping](02_java_idiom_translation.md#collections-mapping) |
| `Map.containsKey(k)` | `k in d` | [Part 2 § collections-mapping](02_java_idiom_translation.md#collections-mapping) |

> ⚠️ **Caveat on `unmodifiableSet` → `frozenset`:** Java's `unmodifiableSet` is a *view* — mutations to the backing set ARE visible through it. `frozenset` is a *copy* — independent of any source set. If you need a live read-only view in Python, expose a `MappingProxyType` (for dicts) or hand back a copy at the API boundary.

## Object model

| Java | Python | See |
|---|---|---|
| `equals(Object)` | `__eq__` | [Part 2 § java-object-model-mapping](02_java_idiom_translation.md#java-object-model-mapping) |
| `hashCode()` | `__hash__` | [Part 2 § java-object-model-mapping](02_java_idiom_translation.md#java-object-model-mapping) |
| `toString()` | `__str__` (user-facing) / `__repr__` (debug) | [Part 2 § java-object-model-mapping](02_java_idiom_translation.md#java-object-model-mapping) |
| `Comparable.compareTo` | `__lt__` / `functools.total_ordering` / `sorted(key=…)` | [Part 2 § java-object-model-mapping](02_java_idiom_translation.md#java-object-model-mapping) |
| `Cloneable` | `copy.copy` / `copy.deepcopy` | [Part 2 § java-object-model-mapping](02_java_idiom_translation.md#java-object-model-mapping) |
| `record` | `@dataclass(frozen=True)` | [Part 2 § dataclass](02_java_idiom_translation.md#dataclass) |
| implicit `super()` in subclass ctor | **explicit** `super().__init__(...)` (Python does NOT auto-call) | [Part 2 § oop-basics](02_java_idiom_translation.md#oop-basics) |
| `obj.field` requires declaration | Python lets you add `obj.field = ...` at runtime — typo-prone | [Part 2 § access-conventions](02_java_idiom_translation.md#access-conventions) |
| `StringUtils` / Apache Commons strings | built into `str` (`upper`, `lower`, `strip`, `split`, `join`, `startswith`, …) | [Part 5 § regex-and-strings](05_standard_library.md#regex-and-strings) |
| record-style validation in constructor | `__post_init__` in dataclass | [Part 2 § dataclass](02_java_idiom_translation.md#dataclass) |
| `enum` (basic) | `enum.Enum` / `IntEnum` / `StrEnum` (🐍 3.11+) | [Part 2 § enum](02_java_idiom_translation.md#enum) |
| enum constructor + per-member fields | tuple `value` + `__init__` unpack | [Part 2 § enum](02_java_idiom_translation.md#enum) |
| `Color.values()` / `valueOf("RED")` / `RED.name()` / `RED.ordinal()` | `list(Color)` / `Color["RED"]` / `RED.name` / no built-in (use `list(Color).index(RED)`) | [Part 2 § enum](02_java_idiom_translation.md#enum) |
| enum implements `Comparable` by declaration order | NOT automatic — use `IntEnum` (compares as int) or define `__lt__` | [Part 2 § enum](02_java_idiom_translation.md#enum) |
| `EnumSet` / `EnumMap` | `set` of members / `dict` keyed by member (members are hashable) | [Part 2 § enum](02_java_idiom_translation.md#enum) |
| abstract class + abstract method | `abc.ABC` + `@abstractmethod` (nominal inheritance) | [Part 2 § abstract-classes](02_java_idiom_translation.md#abstract-classes) |
| `interface` (nominal) | `abc.ABC` (nominal) OR `typing.Protocol` (structural) — see notes | [Part 3 § protocol](03_pythonic_idioms.md#protocol) |
| `final` field (per-field, static check) | `typing.Final` (mypy/pyright only; no runtime enforcement) | [Part 2 § immutable-objects](02_java_idiom_translation.md#immutable-objects) |
| `final` class / Immutables / AutoValue | `@dataclass(frozen=True)` (default) / `typing.NamedTuple` | [Part 2 § immutable-objects](02_java_idiom_translation.md#immutable-objects) |
| Singleton pattern (private ctor, static `INSTANCE`) | module (default, thread-safe via import lock) / metaclass + `Lock` (class-based, real guarantees) / `__new__` (textbook but racy) / `@cache` (per-key, NOT exactly-once) | [Part 2 § singleton](02_java_idiom_translation.md#singleton) |
| Enum-as-singleton (Bloch's Item 3) | module-level instance — Python `import` is already serialized | [Part 2 § singleton](02_java_idiom_translation.md#singleton) |

> ⚠️ **Caveat on `interface` → Python:** Java `interface` is *nominal* — implementers must declare `implements X`. Python has two answers: `abc.ABC` for the same nominal contract (subclass must inherit), or `typing.Protocol` for *structural* typing (any class with the right methods qualifies, no inheritance required). Pick `Protocol` when you care about duck typing made checkable; pick `ABC` when you want explicit declared subtyping like Java.

## Dates and time

| Java | Python | See |
|---|---|---|
| `LocalDateTime` | `datetime.datetime` | [Part 5 § datetime](05_standard_library.md#datetime) |
| `LocalDate` | `datetime.date` | [Part 5 § datetime](05_standard_library.md#datetime) |
| `ZonedDateTime` | `datetime` + `zoneinfo.ZoneInfo` | [Part 5 § datetime](05_standard_library.md#datetime) |
| `Duration` / `Period` | `datetime.timedelta` | [Part 5 § datetime](05_standard_library.md#datetime) |
| `DateTimeFormatter.parse` | `datetime.strptime(s, fmt)` | [Part 5 § date-and-time-parsing](05_standard_library.md#date-and-time-parsing) |
| `OffsetDateTime.parse` (ISO) | `datetime.fromisoformat(s)` | [Part 5 § date-and-time-parsing](05_standard_library.md#date-and-time-parsing) |

## Numbers and precision

| Java | Python | See |
|---|---|---|
| `BigDecimal` | `decimal.Decimal` (construct from string!) | [Part 5 § decimal](05_standard_library.md#decimal) |
| `int` / `long` / `BigInteger` | just `int` (arbitrary precision built in; no separate types, no overflow) | [Part 1 § operators](01_syntax_shock.md#operators) |
| OpenJDK / Eclipse OpenJ9 / Azul / GraalVM (multiple Java SE impls) | CPython / PyPy / Jython / GraalPy / IronPython / MicroPython (multiple Python impls) | [Part 1 § execution-model](01_syntax_shock.md#execution-model) |
| Run Python inside a JVM application | Jython (Py2-stuck) or GraalPy (Py3, modern) | [Part 1 § execution-model](01_syntax_shock.md#execution-model) |
| `Random` | `random` module / `random.SystemRandom` for crypto | [Part 5 § math-random-stats](05_standard_library.md#math-random-stats) |

## Concurrency

| Java | Python | See |
|---|---|---|
| `synchronized` / `ReentrantLock` | `threading.Lock` / `RLock` | [Part 4 § sync-primitives](04_concurrency.md#sync-primitives) |
| `Semaphore` | `threading.Semaphore` | [Part 4 § sync-primitives](04_concurrency.md#sync-primitives) |
| `Condition` / `wait`/`notify` | `threading.Condition` | [Part 4 § sync-primitives](04_concurrency.md#sync-primitives) |
| `CountDownLatch(1)` (simple one-shot signal) | `threading.Event` | [Part 4 § sync-primitives](04_concurrency.md#sync-primitives) |
| `CountDownLatch(N)` (count down to zero) | `threading.Barrier(N)` OR `Condition` + integer counter — see notes | [Part 4 § sync-primitives](04_concurrency.md#sync-primitives) |
| `CyclicBarrier` | `threading.Barrier` | [Part 4 § sync-primitives](04_concurrency.md#sync-primitives) |
| `BlockingQueue` | `queue.Queue` (threads) / `asyncio.Queue` (async) | [Part 4 § producer-consumer](04_concurrency.md#producer-consumer) |
| `ExecutorService` | `concurrent.futures.ThreadPoolExecutor` | [Part 4 § thread-pools](04_concurrency.md#thread-pools) |
| `CompletableFuture` | `async` + `await` | [Part 4 § async-and-await](04_concurrency.md#async-and-await) |
| `StructuredTaskScope` | `asyncio.TaskGroup` (🐍 3.11+) | [Part 4 § gather-vs-taskgroup](04_concurrency.md#gather-vs-taskgroup) |
| `ThreadLocal<T>` | `contextvars.ContextVar` (works with async; `threading.local` doesn't) | [Part 4 § contextvars](04_concurrency.md#contextvars) |
| `try-with-resources` | `with` (sync) / `async with` (coroutines) | [Part 3 § context-managers](03_pythonic_idioms.md#context-managers) |
| `Thread.interrupt()` | `Task.cancel()` + cooperative cancellation | [Part 4 § cancellation-and-timeouts](04_concurrency.md#cancellation-and-timeouts) |

> ⚠️ **Caveat on `CountDownLatch` → `Event`:** `Event` is a 1-bit flag — fine for "is the system ready yet?" (`CountDownLatch(1)`). For real countdown of N tasks, use `Barrier(N)` (all parties wait until the last arrives — best when the waiters *are* the counting parties) or an integer counter protected by `Condition` (best when watchers and counters are different threads). Don't pretend `Event` covers the general case.

## I/O and paths

| Java | Python | See |
|---|---|---|
| `java.nio.file.Path` / `Files` | `pathlib.Path` | [Part 5 § pathlib](05_standard_library.md#pathlib) |
| `BufferedReader` + try-with-resources | `with open(...) as f` | [Part 5 § pathlib](05_standard_library.md#pathlib) |
| async file I/O (project Loom virtual threads) | `aiofiles` (3rd-party) or `asyncio.to_thread(open, ...)` | [Part 4 § async-file-io](04_concurrency.md#async-file-io) |
| `Jackson` / `Gson` (read/write only) | `json` module | [Part 5 § json](05_standard_library.md#json) |
| `Jackson` (validating object mapping) | `pydantic` `BaseModel` | [Part 6 § productivity-libs](06_ecosystem_and_packaging.md#productivity-libs) |
| `Properties` file | `configparser` | [Part 5 § configparser](05_standard_library.md#configparser) |

## Exception model

| Java | Python | See |
|---|---|---|
| `throw` | `raise` | [Part 1 § exception-handling](01_syntax_shock.md#exception-handling) |
| `catch` | `except` | [Part 1 § exception-handling](01_syntax_shock.md#exception-handling) |
| `try { … } catch { … } finally { … }` | `try / except / else / finally` | [Part 1 § exception-handling](01_syntax_shock.md#exception-handling) |
| checked exceptions | (no equivalent — none enforced) | [Part 1 § exception-handling](01_syntax_shock.md#exception-handling) |
| chained: `throw new X(msg, cause)` | `raise X(msg) from cause` | [Part 1 § exception-handling](01_syntax_shock.md#exception-handling) |
| multi-catch from concurrent failures | `ExceptionGroup` + `except*` (🐍 3.11+) | [Part 4 § exception-groups](04_concurrency.md#exception-groups) |

## Caching and interception

| Java | Python | See |
|---|---|---|
| `@Cacheable` / Guava `Cache` (in-process) | `@functools.cache` / `@functools.lru_cache` | [Part 3 § decorators](03_pythonic_idioms.md#decorators) |
| cached field (compute once) | `@functools.cached_property` | [Part 3 § decorators](03_pythonic_idioms.md#decorators) |
| AOP / proxy interception | decorators (function-level) / class decorators (class-level) | [Part 3 § decorators](03_pythonic_idioms.md#decorators) |
| distributed cache (Redis-backed `@Cacheable`) | no stdlib equivalent — use `redis-py` + custom decorator, or `aiocache` | [Part 6 § productivity-libs](06_ecosystem_and_packaging.md#productivity-libs) |

## Registry, DI, configuration

| Java | Python | See |
|---|---|---|
| Spring `@Component` scanning | `__init_subclass__` registry pattern | [Part 2 § init-subclass](02_java_idiom_translation.md#init-subclass) |
| `ServiceLoader` | entry points in `pyproject.toml` | [Part 6 § pyproject](06_ecosystem_and_packaging.md#pyproject) |
| `@Autowired` (constructor / field DI) | explicit constructor injection (most code); `Depends(...)` in FastAPI | [Part 6 § middleware-and-di](06_ecosystem_and_packaging.md#middleware-and-di) |
| Spring `@ConfigurationProperties` | `pydantic-settings.BaseSettings` | [Part 6 § settings-management](06_ecosystem_and_packaging.md#settings-management) |
| `@Value("${...}")` | `os.getenv(...)` / `pydantic-settings` field | [Part 5 § env-vars](05_standard_library.md#env-vars) |
| servlet filter / Spring interceptor | framework middleware (Flask `before_request`, Django middleware, FastAPI middleware) | [Part 6 § middleware-and-di](06_ecosystem_and_packaging.md#middleware-and-di) |

## Validation, serialization, ORM

| Java | Python | See |
|---|---|---|
| Jackson `ObjectMapper` (validating) | `pydantic.BaseModel` + `model_validate` / `model_dump` | [Part 6 § productivity-libs](06_ecosystem_and_packaging.md#productivity-libs) |
| Bean Validation (Hibernate Validator, JSR-380) | `pydantic` field validators | [Part 6 § productivity-libs](06_ecosystem_and_packaging.md#productivity-libs) |
| Hibernate / JPA | `sqlalchemy` 2.x ORM | [Part 6 § database-access](06_ecosystem_and_packaging.md#database-access) |
| Flyway / Liquibase | `alembic` | [Part 6 § database-access](06_ecosystem_and_packaging.md#database-access) |
| JDBC | `sqlalchemy` Core / `psycopg` / stdlib `sqlite3` | [Part 6 § database-access](06_ecosystem_and_packaging.md#database-access) |
| reactive DB drivers (R2DBC) | async drivers: `asyncpg`, `aiosqlite`, `aiomysql` + SQLAlchemy `AsyncSession` | [Part 6 § database-access](06_ecosystem_and_packaging.md#database-access) |

## Auth and security

| Java | Python | See |
|---|---|---|
| Spring Security (full framework) | no 1:1 — assemble: PyJWT + passlib + framework session | [Part 6 § auth-and-security](06_ecosystem_and_packaging.md#auth-and-security) |
| JWT (Nimbus, jjwt) | `PyJWT` or `python-jose` | [Part 6 § auth-and-security](06_ecosystem_and_packaging.md#auth-and-security) |
| password hashing (BCrypt, Argon2) | `passlib` or `bcrypt` / `argon2-cffi` | [Part 6 § auth-and-security](06_ecosystem_and_packaging.md#auth-and-security) |
| OAuth/OIDC client | `Authlib` | [Part 6 § auth-and-security](06_ecosystem_and_packaging.md#auth-and-security) |
| session cookies | framework-native (Flask `session`, Django sessions, FastAPI `SessionMiddleware`) | [Part 6 § auth-and-security](06_ecosystem_and_packaging.md#auth-and-security) |

## HTTP and URLs

| Java | Python | See |
|---|---|---|
| `HttpClient` / OkHttp | `requests` (sync) / `httpx` (sync + async) | [Part 6 § http-clients](06_ecosystem_and_packaging.md#http-clients) |
| OkHttp retry/interceptor | `tenacity` decorator + `httpx`/`requests` event hooks | [Part 6 § http-production-behavior](06_ecosystem_and_packaging.md#http-production-behavior) |
| `URI` / `URL` builder | `urllib.parse.urlencode` / `urljoin` / `urlparse` | [Part 5 § urllibparse](05_standard_library.md#urllibparse) |
| Spring `WebClient` (reactive) | `httpx.AsyncClient` | [Part 6 § http-clients](06_ecosystem_and_packaging.md#http-clients) |

## Streams and functional

| Java | Python | See |
|---|---|---|
| `stream().map(f).toList()` | `[f(x) for x in xs]` or `list(map(f, xs))` | [Part 3 § functional-patterns](03_pythonic_idioms.md#functional-patterns) |
| `stream().filter(p).toList()` | `[x for x in xs if p(x)]` or `list(filter(p, xs))` | [Part 3 § functional-patterns](03_pythonic_idioms.md#functional-patterns) |
| `stream().reduce(0, Integer::sum)` | `sum(xs)` or `reduce(operator.add, xs, 0)` | [Part 3 § functional-patterns](03_pythonic_idioms.md#functional-patterns) |
| `stream().sorted(comparing(f))` | `sorted(xs, key=f)` | [Part 3 § functional-patterns](03_pythonic_idioms.md#functional-patterns) |
| `stream().min(comparing(f))` / `.max(...)` | `min(xs, key=f)` / `max(xs, key=f)` | [Part 3 § functional-patterns](03_pythonic_idioms.md#functional-patterns) |
| `stream().mapToInt(f).sum()` | `sum(f(x) for x in xs)` (generator expression — lazy) | [Part 3 § functional-patterns](03_pythonic_idioms.md#functional-patterns) |
| `stream().distinct()` | `set(xs)` (unordered) / `list(dict.fromkeys(xs))` (ordered) | [Part 3 § functional-patterns](03_pythonic_idioms.md#functional-patterns) |
| `Collectors.groupingBy(f)` | `defaultdict(list)` + single-pass loop (true `Map<K, List<V>>`); `itertools.groupby` only for adjacent runs | [Part 3 § functional-patterns](03_pythonic_idioms.md#functional-patterns) |
| `Collectors.partitioningBy(p)` | single-pass loop into `truthy` / `falsy` (two-comprehension form fails on iterators) | [Part 3 § functional-patterns](03_pythonic_idioms.md#functional-patterns) |
| `Collectors.joining(", ")` | `", ".join(str(x) for x in xs)` | [Part 5 § regex-and-strings](05_standard_library.md#regex-and-strings) |
| `Optional<T>.map(f)` | `f(x) if x is not None else None` | [Part 1 § none-and-is](01_syntax_shock.md#none-and-is) |
| `Optional<T>.orElse(d)` | `x if x is not None else d` — same shape; Java `orElse(d)` evaluates `d` eagerly because it's an argument | [Part 1 § none-and-is](01_syntax_shock.md#none-and-is) |
| `Optional<T>.orElseGet(() -> d())` | `x if x is not None else d()` — Python's conditional expression is lazy on the unused branch, so `d()` runs only when `x is None` | [Part 1 § none-and-is](01_syntax_shock.md#none-and-is) |
| `Function.identity()` | `lambda x: x` | [Part 3 § functions-as-first-class-objects](03_pythonic_idioms.md#functions-as-first-class-objects) |
| `Function.andThen(g)` | `lambda x: g(f(x))` for one-off; `toolz.compose` for chains (no stdlib `compose`) | [Part 3 § functional-patterns](03_pythonic_idioms.md#functional-patterns) |
| lambda for `Comparator` | `key=operator.itemgetter("x")` or `key=operator.attrgetter("x")` | [Part 3 § functional-patterns](03_pythonic_idioms.md#functional-patterns) |
| `Stream.concat(a, b)` | `itertools.chain(a, b)` | [Part 3 § functional-patterns](03_pythonic_idioms.md#functional-patterns) |
| `Stream.takeWhile(p)` / `.dropWhile(p)` | `itertools.takewhile(p, it)` / `itertools.dropwhile(p, it)` | [Part 3 § functional-patterns](03_pythonic_idioms.md#functional-patterns) |
| `BiFunction` partial application | `functools.partial(f, x)` | [Part 3 § functional-patterns](03_pythonic_idioms.md#functional-patterns) |

## Interop and integration

| Need / Java reference | Python / cross-language answer | See |
|---|---|---|
| Default cross-language boundary | HTTP/REST (FastAPI server) | [Part 7 § Choosing an interop pattern](07_interoperability.md#choosing-an-interop-pattern) |
| Typed RPC across languages | gRPC with shared `.proto` (Python via `grpcio-tools`) | [Part 7 § Python and Java](07_interoperability.md#python-and-java) |
| Java calling a Python service | HTTP (`HttpClient` / OkHttp / `WebClient`) → FastAPI | [Part 7 § Python and Java](07_interoperability.md#python-and-java) |
| Embed Python inside JVM app | **GraalPy** (modern, Py3) or **Py4J** (PySpark mechanism); avoid Jython (Py2-stuck) | [Part 7 § Python and Java](07_interoperability.md#python-and-java) |
| Call Java from Python | **JPype** (loads JVM into Python process) or **Py4J** | [Part 7 § Python and Java](07_interoperability.md#python-and-java) |
| Node.js calling a Python service | HTTP/REST or gRPC (same shape as Java) | [Part 7 § Python and Node.js](07_interoperability.md#python-and-nodejs) |
| Streaming LLM tokens to browser | **Server-Sent Events** (FastAPI `StreamingResponse`) | [Part 7 § Python and Node.js](07_interoperability.md#python-and-nodejs) |
| Bidirectional browser ↔ server | WebSocket (FastAPI / Starlette) | [Part 7 § Python and Node.js](07_interoperability.md#python-and-nodejs) |
| Run Python in the browser | **Pyodide** (CPython on WebAssembly) | [Part 7 § Python and Node.js](07_interoperability.md#python-and-nodejs) |
| Python calling a C library | `ctypes` (stdlib, no compile) or `cffi` (cleaner) | [Part 7 § Python and C and C++](07_interoperability.md#python-and-c-and-c) |
| Python calling C++ | **`pybind11`** (modern default; what PyTorch/FAISS use) | [Part 7 § Python and C and C++](07_interoperability.md#python-and-c-and-c) |
| Hot-path optimization in pure Python | **Cython** (annotate types, compile to C extension) | [Part 7 § Python and C and C++](07_interoperability.md#python-and-c-and-c) |
| Schema source of truth across languages | Pydantic → JSON Schema → TS/Java POJO; or OpenAPI from FastAPI | [Part 7 § Schema sharing across languages](07_interoperability.md#schema-sharing-across-languages) |
| Cross-language large-int round-trip | Use **string** representation (JSON numbers ≠ JS BigInt safe) or Protobuf `int64` | [Part 7 § Pitfalls](07_interoperability.md#pitfalls) |
| Cross-language binary data exchange | **Protobuf** / **Arrow** / **Parquet** — NOT `pickle` | [Part 7 § Pitfalls](07_interoperability.md#pitfalls) |
| RAG / ML service shape (production default) | Sidecar pattern — Python container behind HTTP/gRPC; polyglot frontend | [Part 7 § RAG-specific integration patterns](07_interoperability.md#rag-specific-integration-patterns) |
| Indexing in Python, queries from anywhere | Vector store with native clients per language (Pinecone / Weaviate / Qdrant / pgvector / Milvus) | [Part 7 § RAG-specific integration patterns](07_interoperability.md#rag-specific-integration-patterns) |

## Testing

| Java | Python | See |
|---|---|---|
| JUnit | `pytest` | [Part 6 § pytest](06_ecosystem_and_packaging.md#pytest) |
| `@BeforeEach` | pytest fixture (function-scope) | [Part 6 § pytest](06_ecosystem_and_packaging.md#pytest) |
| `@BeforeAll` | pytest fixture (module/session-scope) | [Part 6 § pytest](06_ecosystem_and_packaging.md#pytest) |
| `@ParameterizedTest` | `@pytest.mark.parametrize` | [Part 6 § pytest](06_ecosystem_and_packaging.md#pytest) |
| Mockito `mock()` / `when().thenReturn()` | `unittest.mock.MagicMock` / `patch` / `pytest-mock` | [Part 6 § test-doubles](06_ecosystem_and_packaging.md#test-doubles) |
| Mockito for async (`Mono`/`Flux`) | `unittest.mock.AsyncMock` | [Part 6 § test-doubles](06_ecosystem_and_packaging.md#test-doubles) |

## Tooling

| Java | Python | See |
|---|---|---|
| Maven / Gradle | `pip` + `venv` + Poetry / uv | [Part 6 § venv-and-packaging](06_ecosystem_and_packaging.md#venv-and-packaging) |
| `pom.xml` / `build.gradle` | `pyproject.toml` | [Part 6 § pyproject](06_ecosystem_and_packaging.md#pyproject) |
| Checkstyle / SpotBugs | `ruff` | [Part 6 § tooling](06_ecosystem_and_packaging.md#tooling) |
| google-java-format | `ruff format` (subsumes `black`) | [Part 6 § tooling](06_ecosystem_and_packaging.md#tooling) |
| `javac` type checks | `mypy` (run separately, not at runtime) | [Part 6 § tooling](06_ecosystem_and_packaging.md#tooling) |
| Log4j / SLF4J | `logging` (stdlib) | [Part 5 § logging](05_standard_library.md#logging) |
| `java.util.logging` | `logging` (stdlib) | [Part 5 § logging](05_standard_library.md#logging) |
| ELK-friendly JSON logs | custom `logging.Formatter` or `structlog` | [Part 5 § structured-logging-notes](05_standard_library.md#structured-logging-notes) |
| MDC (Mapped Diagnostic Context) | `contextvars.ContextVar` + `logging.Filter` | [Part 4 § async-aware-logging](04_concurrency.md#async-aware-logging) |
| Spring Boot | Django (similar scope — batteries-included); Flask/FastAPI ≈ Spring Web MVC slice | [Part 6 § web-frameworks](06_ecosystem_and_packaging.md#web-frameworks) |
