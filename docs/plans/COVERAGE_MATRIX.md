# Coverage Matrix — old `Part*.md` → new files

Every existing `##`/`###` in `Part1.md`–`Part5.md` mapped to one of:
- **MIGRATED** → moved to new file#anchor
- **TEASER** → mentioned briefly in new file with link to its anchor home
- **SPLIT** → divided between two new homes
- **DROPPED** → removed, with reason

Old files are deleted only after Phase 3 verifies this matrix accounts for everything.

> **Grouping convention:** Rows below are listed at the granularity that's useful for migration. **A parent `##` row implicitly covers all of its `###` sub-headings unless a sub-heading appears as its own explicit row.** Trivial sub-headings like "Example", "Java comparison", "Why this matters", and "Common pitfall" are intentionally not separately rowed — they're prose-internal scaffolding that moves with the parent body. The "Minimal comparison" code-pair that opens each old part is preserved inside each new part's `## Core differences` section (re-scoped to the new part's topic).

---

## Part1.md → new homes

| Old section | Action | New home | Notes |
|---|---|---|---|
| §1 Core Differences at a Glance | MIGRATED | `01_syntax_shock.md#core-differences` | Re-scoped to syntax/exec topics only |
| §1 Minimal comparison (code pair) | MIGRATED | `01_syntax_shock.md#core-differences` | Lives inside Core differences |
| §2 Static vs dynamic typing | MIGRATED | `01_syntax_shock.md#typing-basics` | + teaser to `03#type-hints` |
| §2 Functions and multiple return values | MIGRATED | `01_syntax_shock.md#functions` | |
| §2 Execution model | MIGRATED | `01_syntax_shock.md#execution-model` | |
| §2 Entry point | MIGRATED | `01_syntax_shock.md#entry-point` | |
| §3.1 dict vs HashMap (+ sub: TreeMap?, When matters, Iterating) | MIGRATED | `02_java_idiom_translation.md#collections-mapping` | Sub-headings covered by parent |
| §3.1.1 Set vs HashSet/TreeSet | MIGRATED | `02_java_idiom_translation.md#collections-mapping` | |
| §3.2 Tuple (+ sub: single-element quirk, list→tuple, immutability) | MIGRATED | `02_java_idiom_translation.md#tuple` | |
| §3.3 Set (+ sub: empty set quirk) | MIGRATED | `02_java_idiom_translation.md#collections-mapping` | |
| §3.4 frozenset | MIGRATED | `02_java_idiom_translation.md#frozenset` | |
| §3.5 Binary data (+ sub: bytes, bytearray, memoryview) | MIGRATED | `02_java_idiom_translation.md#binary-types` | |
| §3.6 range | MIGRATED | `01_syntax_shock.md#range` | |
| §3.7 complex | MIGRATED | `01_syntax_shock.md#complex` | |
| §3.8 Strings, Unicode, Encodings (+ sub: encode, decode, file enc, pitfall, Java cmp) | MIGRATED | `01_syntax_shock.md#str-vs-bytes` | Peer-flagged critical |
| §4 Sequence Utilities (+ sub: zip, any/all, sorted/reversed) | MIGRATED | `01_syntax_shock.md#sequence-utilities` | All sub-fns covered by parent body |
| §5 Common keyword differences | MIGRATED | `01_syntax_shock.md#control-flow` | |
| §5 None | MIGRATED | `01_syntax_shock.md#none-and-is` | |
| §5 Match-case | MIGRATED | `01_syntax_shock.md#match` | + teaser to `03#match-patterns` |
| §6 Functions as First-Class (+ sub: lambda, nested+scope) | MIGRATED | `03_pythonic_idioms.md#functions-as-first-class-objects` | nonlocal subset → `03#scope-and-nonlocal` |
| §7.1 Basic class | MIGRATED | `02_java_idiom_translation.md#oop-basics` | |
| §7.2 dict not a class | MIGRATED | `02_java_idiom_translation.md#dict-vs-class` | |
| §7.3 Abstract classes | MIGRATED | `02_java_idiom_translation.md#abstract-classes` | |
| §7.4 Encapsulation the Pythonic way | MIGRATED | `02_java_idiom_translation.md#encapsulation` | |
| §7.5 _protected and __private | MIGRATED | `02_java_idiom_translation.md#access-conventions` | |
| §7.6 Composition over inheritance | MIGRATED | `02_java_idiom_translation.md#composition-over-inheritance` | Promoted from one-liner |
| §8 Special Methods (+ Vector example) | MIGRATED | `02_java_idiom_translation.md#dunder-methods` | Vector example moves with parent |
| §8 hashCode/equals/Comparable/Cloneable mappings (+ all subs: __str__, __eq__, __hash__, Comparable, Cloneable, shallow/deep, mental model, __repr__) | MIGRATED | `02_java_idiom_translation.md#java-object-model-mapping` | All sub-headings covered |
| §8 dataclass(order=True, frozen=True) | MIGRATED | `02_java_idiom_translation.md#dataclass` | |
| §9 classmethod/staticmethod | MIGRATED | `02_java_idiom_translation.md#class-and-static-methods` | |
| §10 File handling with `with open(...)` | MIGRATED | `05_standard_library.md#pathlib` | Consolidated with pathlib |
| §10 JSON load/dump | MIGRATED | `05_standard_library.md#json` | Was duplicated Part1/Part4 |
| §11 assert | MIGRATED | `03_pythonic_idioms.md#advanced-control-flow` | + early warning in `01#exception-handling` |
| §11 break | DROPPED | n/a | Standard Java idiom; no Python-specific story |
| §11 while…else | MIGRATED | `03_pythonic_idioms.md#advanced-control-flow` | |
| §11 for…else | MIGRATED | `03_pythonic_idioms.md#advanced-control-flow` | |
| §11 del | MIGRATED | `03_pythonic_idioms.md#advanced-control-flow` | |
| §11 nonlocal | MIGRATED | `03_pythonic_idioms.md#scope-and-nonlocal` | |
| §12 yield + generator + read-in-batches + Generator vs Coroutine | MIGRATED | `03_pythonic_idioms.md#generators` | Sub-table links to `04#coroutines` |
| §13 type/isinstance/dir/getattr/setattr/hasattr + inspect | MIGRATED | `03_pythonic_idioms.md#introspection` | |
| §13 Metaprogramming (dynamic methods, type()) | MIGRATED | `03_pythonic_idioms.md#metaprogramming` | |
| §14 Practical Takeaways | DROPPED | n/a | Redundant; per-part Key Takeaways (Q8) |
| §15 Final Summary | DROPPED | n/a | Redundant; per Q8 |

## Part2.md → new homes

| Old section | Action | New home | Notes |
|---|---|---|---|
| §1 Core Differences (+ Minimal comparison) | MIGRATED | `04_concurrency.md#core-differences` | Scoped to concurrency |
| §2 Coroutine | MIGRATED | `04_concurrency.md#coroutines` | |
| §2 async | MIGRATED | `04_concurrency.md#async-and-await` | |
| §2 await | MIGRATED | `04_concurrency.md#async-and-await` | |
| §2 When to use async/await | MIGRATED | `04_concurrency.md#when-to-use-async` | |
| §2 asyncio.gather | MIGRATED | `04_concurrency.md#gather-vs-taskgroup` | Pairs with new TaskGroup |
| §3 Multi-threading basics + When threads useful | MIGRATED | `04_concurrency.md#threading` | |
| §3 GIL caveat | MIGRATED | `04_concurrency.md#gil` | Consolidated with §4 GIL |
| §3 Common pitfalls of Python threads (all 4 sub-items) | MIGRATED | `04_concurrency.md#threading-pitfalls` | |
| §4 Thread safety in Python | MIGRATED | `04_concurrency.md#thread-safety` | |
| §4 Python memory model vs JMM + Lock/visibility + Safe/Unsafe + Mental shortcut | MIGRATED | `04_concurrency.md#memory-visibility` | |
| §4 Better alternatives (Event example) | MIGRATED | `04_concurrency.md#sync-primitives` | |
| §4 Race-condition lock fix example | MIGRATED | `04_concurrency.md#thread-safety` | |
| §5 Threads vs async coroutines + Blocking vs non-blocking I/O | MIGRATED | `04_concurrency.md#blocking-vs-non-blocking-io` | |
| §5 Thread vs async vs multiprocessing chooser | MIGRATED | `04_concurrency.md#concurrency-chooser` | |
| §5 Relationship between threads and async | MIGRATED | `04_concurrency.md#mixing-async-and-threads` | + new `asyncio.to_thread` content |
| §6 Lock/RLock/Semaphore/Condition/Event | MIGRATED | `04_concurrency.md#sync-primitives` | + new `Barrier` |
| §7 Threaded P-C with queue.Queue | MIGRATED | `04_concurrency.md#producer-consumer` | |
| §7 queue.Queue vs shared list+Lock | MIGRATED | `04_concurrency.md#producer-consumer` | |
| §7 queue.Queue vs asyncio.Queue | MIGRATED | `04_concurrency.md#producer-consumer` | |
| §7 Async P-C with asyncio.Queue | MIGRATED | `04_concurrency.md#producer-consumer` | |
| §8 ThreadPoolExecutor + vs raw Thread | MIGRATED | `04_concurrency.md#thread-pools` | |
| §9 Practical Takeaways | DROPPED | n/a | Redundant |
| §10 Final Summary | DROPPED | n/a | Redundant |

## Part3.md → new homes

| Old section | Action | New home | Notes |
|---|---|---|---|
| §1 Core Differences (+ Minimal comparison) | MIGRATED | `03_pythonic_idioms.md#core-differences` | Scoped to idioms |
| §2 try/except/else/finally + Raising + Custom + Java mental model | MIGRATED | `01_syntax_shock.md#exception-handling` | Basics = Survive level. + teaser to `04#exception-groups` |
| §3 Comprehensions (list/filter/dict/set + Java cmp) | MIGRATED | `03_pythonic_idioms.md#comprehensions` | |
| §4 *args/**kwargs/unpacking/star assignment | MIGRATED | `03_pythonic_idioms.md#args-and-kwargs` | |
| §5 Iterable/Iterator/why-matters | MIGRATED | `03_pythonic_idioms.md#iterable-vs-iterator` | |
| §6 Mutable vs Immutable + Mutable default args pitfall | MIGRATED | `02_java_idiom_translation.md#mutability` | |
| §7 Truthiness | MIGRATED | `01_syntax_shock.md#truthiness` | |
| §8 is vs == | MIGRATED | `01_syntax_shock.md#none-and-is` | Merged into single None+is anchor |
| §9 Modules, Packages, Imports | MIGRATED | `01_syntax_shock.md#modules-and-imports` | |
| §10 dataclass | MIGRATED | `02_java_idiom_translation.md#dataclass` | Single anchor (was in Part1 §8 too) |
| §11 Context Managers Beyond Files | MIGRATED | `03_pythonic_idioms.md#context-managers` | |
| §12 Practical Takeaways | DROPPED | n/a | Redundant |
| §13 Final Summary | DROPPED | n/a | Redundant |

## Part4.md → new homes

| Old section | Action | New home | Notes |
|---|---|---|---|
| §1 Core Differences (+ Minimal comparison) | MIGRATED | `05_standard_library.md#core-differences` | Scoped to stdlib |
| §2 Core Language Utilities (Common mappings table) | MIGRATED | `99_appendix_java_to_python.md#core-language` | Bulk to appendix |
| §2 Example (str ops) | DROPPED | n/a | Covered in `01#str-vs-bytes` |
| §2 Important difference (function-oriented) | MIGRATED | `05_standard_library.md#stdlib-philosophy` | |
| §3 Built-in collections | DROPPED | n/a | Anchored in `02#collections-mapping` |
| §3 collections module (+ Counter, defaultdict, deque, Java cmp) | MIGRATED | `05_standard_library.md#collections-module` | |
| §4 datetime (+ date, timedelta, zoneinfo, Java cmp) | MIGRATED | `05_standard_library.md#datetime` | |
| §5 pathlib (+ read/write, mkdir, shutil.copy, Java cmp) | MIGRATED | `05_standard_library.md#pathlib` | |
| §6 String ops + re (+ Java cmp) | MIGRATED | `05_standard_library.md#regex-and-strings` | |
| §7 JSON | MIGRATED | `05_standard_library.md#json` | |
| §7 Config files (configparser) | MIGRATED | `05_standard_library.md#configparser` | Minor mention |
| §7 Env vars (os.environ) | MIGRATED | `05_standard_library.md#env-vars` | |
| §7 .env files + python-dotenv | MIGRATED | `06_ecosystem_and_packaging.md#env-config` | Third-party split |
| §7 Pickle (+ safety warning) | MIGRATED | `05_standard_library.md#pickle` | |
| §8 math/random/statistics | MIGRATED | `05_standard_library.md#math-random-stats` | |
| §9 Logging | MIGRATED | `05_standard_library.md#logging` | Expanded medium (dictConfig added) |
| §9 Runtime inspection | DROPPED | n/a | Anchored in `03#introspection` |
| §10 Concurrency stdlib modules summary | MIGRATED | `04_concurrency.md#stdlib-modules-summary` | Brief pointer |
| §11 Practical Takeaways | DROPPED | n/a | Redundant |
| §12 Final Summary | DROPPED | n/a | Redundant |

## Part5.md → new homes

| Old section | Action | New home | Notes |
|---|---|---|---|
| §1 Core Differences (+ Minimal comparison) | MIGRATED | `06_ecosystem_and_packaging.md#core-differences` | Scoped to ecosystem |
| §2 Jupyter + IPython (+ Java cmp) | MIGRATED | `06_ecosystem_and_packaging.md#jupyter-and-ipython` | |
| §3 NumPy + pandas + Polars + SciPy (+ Java cmp) | MIGRATED | `06_ecosystem_and_packaging.md#numerical-and-data-libs` | |
| §4 matplotlib + seaborn + plotly (+ Java cmp) | MIGRATED | `06_ecosystem_and_packaging.md#visualization` | |
| §5 requests + httpx (+ Java cmp) | MIGRATED | `06_ecosystem_and_packaging.md#http-clients` | |
| §6 Flask + FastAPI + Django (+ Java cmp) | MIGRATED | `06_ecosystem_and_packaging.md#web-frameworks` | |
| §7 pytest + black + ruff + mypy (+ Java cmp) | MIGRATED | `06_ecosystem_and_packaging.md#pytest` + `06#tooling` | pytest in its own anchor; rest in tooling |
| §8 pip/venv/Poetry/uv | MIGRATED | `06_ecosystem_and_packaging.md#venv-and-packaging` | Expanded medium |
| §9 scikit-learn + pytorch + tensorflow | MIGRATED | `06_ecosystem_and_packaging.md#mlai-libs` | |
| §10 rich + typer + pydantic + openpyxl + sqlalchemy | MIGRATED | `06_ecosystem_and_packaging.md#productivity-libs` | sqlalchemy also gets dedicated treatment in `06#database-access` |
| §11 Practical Takeaways | DROPPED | n/a | Redundant |
| §12 Final Summary | DROPPED | n/a | Redundant |

---

## New content (not from any old part)

| New section | Home | Reason |
|---|---|---|
| f-string formatting depth | `01#f-strings` | Q6 min |
| Legacy string formatting (`.format`, `%`) | `01#legacy-string-formatting` | Peer (Java devs hit it in legacy + logging) |
| Walrus `:=` | `01#walrus` | Q6 min (peer: lead with assign-in-comprehension) |
| `raise from` chaining | `01#exception-handling` | Q6 min |
| `assert` early warning callout | `01#exception-handling` | Peer (devs see asserts early) |
| `match` "not Java switch" warning | `01#match` | Peer |
| Match deep (class/OR/capture/guards) | `03#match-patterns` | Q6 medium |
| Type hints deep (Optional/Union/Generic/Protocol/TypedDict/Literal/Callable/Sequence-Mapping/cast/Any/Self/PEP 695) + type-pragmas | `03#type-hints` | Q6 deep + peer |
| Decorators deep + `@cache`/`@lru_cache`/`@cached_property` | `03#decorators` | Q6 deep |
| `@contextmanager` + `contextlib.ExitStack` | `03#context-managers` | Q6 medium + min |
| `Protocol` + structural typing (paired with `abc.ABC` nominal contrast) | `03#protocol` | Q6 medium + peer correctness |
| `enum` (Enum/IntEnum/StrEnum/auto) | `02#enum` | Q6 medium |
| `__slots__` | `02#slots` | Q6 min |
| `__post_init__` for dataclass validation | `02#dataclass` (within dataclass section) | Peer |
| Class attr vs instance attr | `02#class-vs-instance-attributes` | Q6 medium + peer |
| MRO basics | `02#mro` | Q6 medium + peer |
| `__init_subclass__` | `02#init-subclass` | Q6 min + peer |
| Immutable objects (`@dataclass(frozen=True)` / `NamedTuple` / `__setattr__` override) | `02#immutable-objects` | User request — important Java concept (final / record / Immutables) |
| Singleton (module / `__new__` / `@cache` / Enum member) | `02#singleton` | User request — important Java pattern (private ctor / enum singleton / DCL) |
| Operators (`/` / `//` / `%` / `**` / bitwise / no `++` / comparison chaining) | `01#operators` | User request via `ref/` Day 02 — Java devs hit these in hour one |
| Console I/O (`input` / `print(sep=, end=, file=)`) | `01#console-io` | User request via `ref/` Day 01 |
| Triple-quoted strings | `01#str-vs-bytes` (within) | User request via `ref/` Day 02 |
| Ternary expression (`x if c else y`) | `01#control-flow` (within) | User request via `ref/` Day 03 |
| Default parameter values (feature, not just trap) | `01#functions` (within) | User request via `ref/` Day 04 |
| Dict view methods (`.items` / `.keys` / `.values` / `.get`) | `02#collections-mapping` (within) | User request via `ref/` Day 03 |
| `super().__init__()` explicit + inheritance example | `02#oop-basics` (within) | User request via `ref/` Day 04 |
| Dynamic attribute addition (`obj.foo = bar`) | `02#access-conventions` (within) | User request via `ref/` Day 04 |
| Pass-by-value-of-reference explicit framing | `02#mutability` (within, Java-parallel callout) | User request via `ref/` Day 04 |
| String interning / `is` on literals SyntaxWarning | `01#none-and-is` (within, pitfall callout) | User request via `ref/` Day 02 |
| String methods catalog (`upper`/`lower`/`title`/`strip`/`startswith`/`removeprefix`…) | `05#regex-and-strings` (within) | User request via `ref/` Day 01 |
| Functional patterns (`operator` module / `functools.reduce`+`partial` examples / Stream→Python map / `itertools` combinators / generator pipelines / Pythonic-FP framing) | `03#functional-patterns` | User request — honest gap in FP framing for Java/Streams audience |
| Python-specific keywords cheat sheet (`pass`/`del`/`is`/`in`/`with`/`yield`/`nonlocal`/`global`/`elif`/`for-else`/`assert`/`match`/`raise from`/`except*`) | `01#python-specific-keywords` | User request — Java devs want one-page reference of "shocking" keywords |
| `int` is unbounded (no `long` / `BigInteger`; `2**1000` works) | `01#operators` (within, new `### Integers are unbounded` subsection) | User request — Java distinction of `int`/`long`/`BigInteger` collapses to one type |
| Enum deepening: per-member values via tuple + `__init__`; lookup (`Color["RED"]` / `Color(1)`); iteration (`list(Color)`); Java-feature map (constructor/abstract methods/Comparable/EnumSet); functional API; ordering not automatic pitfall | `02#enum` (within, extended) | User request — Java enums are much richer than named constants |
| Python implementations (CPython / PyPy / Jython / GraalPy / IronPython / MicroPython) — what each is + GIL/bytecode/C-ext consequences + Java-parallel framing | `01#execution-model` (within, new `### Python implementations` subsection) | User request — Java devs know about Jython/GraalPy as JVM bridges; "Python is slow" is a CPython statement, not a language one |
| Custom exceptions with non-string fields (int, dict, etc.) via `__init__` + `super().__init__(message)`; explanation of `exc.args` tuple | `01#exception-handling` (within) | User-asked question — original example was just `pass`; real-world exceptions carry status codes / headers / context |
| Interoperability: full Part 7 — pattern chooser; Python↔Java (Py4J as local socket bridge vs JPype/GraalPy/Jython as true in-process); Python↔Node (HTTP/SSE/WebSocket/Pyodide/python-shell); Python↔C/C++ (ctypes/cffi/pybind11/Cython); RAG-specific patterns (Python-as-service / shared vector store / streaming / embedding service); production concerns (timeouts/retries/circuit breakers/tracing/auth/rate limits/schema versioning/inference batching/cancellation); schema sharing (Pydantic→JSON Schema, OpenAPI from FastAPI); cross-language pitfalls (pickle / int round-trip / encoding / datetime / GIL / Pyodide cost / venv-deployment for in-process bridges) | `07_interoperability.md` (NEW FILE) | User request — integrating Python RAG systems with Java/Node/C++ is the dominant 2026 production pattern; existing docs only covered Python-internal concerns |
| Anti-pattern openers (Bean/Interface/AbsFactory/ThreadLocal/Cacheable/SpringScan) | embedded P2/P3/P4 | Peer device |
| `async with` / `async for` / async ctx mgrs | `04#async-context-managers` | Q6 medium + peer |
| Async file I/O (`aiofiles` / `to_thread`) | `04#async-file-io` | Peer |
| `asyncio.TaskGroup` (structured concurrency) | `04#gather-vs-taskgroup` | Q6 medium + peer |
| `asyncio.timeout` + cancellation | `04#cancellation-and-timeouts` | Q6 medium + peer |
| `ExceptionGroup` / `except*` | `04#exception-groups` | Q6 medium + peer |
| `asyncio.to_thread` / `run_in_executor` | `04#mixing-async-and-threads` | Q6 medium |
| `contextvars.ContextVar` | `04#contextvars` | Q6 min + peer |
| Async-aware logging (ContextVar + Filter) | `04#async-aware-logging` | Peer |
| `threading.Barrier` (true `CountDownLatch(N)`) | `04#sync-primitives` | Peer correctness |
| GIL nuance + free-threaded 3.13+ | `04#gil` | Bumped min→medium (peer) |
| Date parsing (`fromisoformat`, `strptime`) | `05#date-and-time-parsing` | Peer |
| Cross-platform path notes | `05#cross-platform-path-notes` | Peer |
| `decimal.Decimal` (BigDecimal parallel) | `05#decimal` | Peer |
| Structured logging notes (JSON, correlation IDs) | `05#structured-logging-notes` | Peer |
| `urllib.parse` | `05#urllibparse` | Peer |
| `itertools` / `functools` non-decorator / `tempfile` / `csv` / `argparse` / `subprocess` safety | `05#stdlib-quick-wins` | Q6 medium + peer |
| `logging` `dictConfig` + handlers + formatters | `05#logging` | Expanded medium |
| Settings management (`pydantic-settings`) | `06#settings-management` | Peer |
| Middleware + DI (Flask `before_request` / Django middleware / FastAPI `Depends`) | `06#middleware-and-di` | Peer |
| Auth and security (PyJWT / passlib / Authlib) | `06#auth-and-security` | Peer |
| Database access (SQLAlchemy 2.x ORM + Core, Alembic, async drivers, pooling) | `06#database-access` | Peer |
| HTTP production behavior (timeouts, retries, `tenacity`, sessions, pooling) | `06#http-production-behavior` | Peer |
| Test doubles (`unittest.mock` / `pytest-mock` / `AsyncMock` / monkeypatch) | `06#test-doubles` | Peer |
| `pytest` fixtures + parametrize + conftest | `06#pytest` | Expanded medium |
| `venv` walkthrough | `06#venv-and-packaging` | Q6 medium |
| `pyproject.toml` | `06#pyproject` | Q6 medium |

---

## Verification checklist (Phase 3 will re-run)

- [ ] Every old `##` (or grouped `###`) appears in this matrix exactly once
- [ ] Every MIGRATED row's new anchor exists in the target file and matches GitHub auto-slug
- [ ] Every SPLIT row's two new anchors both exist
- [ ] Every TEASER row has a teaser block in the source file pointing to the right anchor
- [ ] Every DROPPED row has an explicit reason
- [ ] No header in any new file contains punctuation that would mangle the GitHub auto-anchor (no `/`, `,`, parens with content)
