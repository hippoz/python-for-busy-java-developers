# Restructure Plan: Python for Busy Java Developers

## Goal

Rebuild `Part1.md`–`Part5.md` into a comprehensive, cohesive, coherent, DRY document set, with strong navigability for a busy Java-developer reader.

## Decisions (locked via grilling + peer review)

| # | Decision | Choice |
|---|---|---|
| Q1 | Scope | **Full restructure** (re-slice by topic axis, extract shared appendix) |
| Q2 | Part axis | **Java-developer journey** — **6 parts** (split stdlib vs ecosystem per peer review) |
| Q3 | Topic placement | **Anchor + teaser** — strict teaser format: 1 sentence + 1 link, **no code blocks, no bullets** |
| Q4 | Java↔Python comparisons | **Distributed + appendix dual** — per-part overview tables + master appendix |
| Q5 | Navigability | TOCs, README index, 3 prose callout types (`☕ Java parallel`, `⚠️ Pitfall`, `💡 Pythonic`) + `🐍 Python 3.X+` version callout, `<details>` for deep dives, **no diagrams**. Cap: **≤1 callout per `###`** (peer-review revision; was `##`). Stacked callouts banned. |
| Q6 | Gap depth | Targeted bundle: **deep** for type hints + decorators; **medium** for ~12 mid-leverage gaps (expanded from peer review); **min** for one-liners |
| Q7 | Voice | **Preserve + light edit** — keep existing prose, normalize examples and callouts |
| Q8 | Epilogues | Per-part **Key Takeaways** (3–5 concrete part-specific bullets) + global **Mindset Shift** in README |
| Q9 | Filenames | **Numbered topical**, lowercase, sortable |
| Q10 | Execution | **3-phase**: skeletons + **coverage matrix** → bodies → polish, with gates (coverage matrix added per peer review) |
| Q11 | Python baseline | **3.11 floor** (peer-review revision; was 3.10 — EOL Oct 2026), with `🐍 Python 3.12+` callouts for newer features |

## Target file layout

```
00_index.md                          README + reading paths + cross-cutting "Mindset Shift"
01_syntax_shock.md                   Part 1 — survive reading Python code
02_java_idiom_translation.md         Part 2 — translate Java reflexes (OOP, equals/hash, collections)
03_pythonic_idioms.md                Part 3 — stop writing Java in Python
04_concurrency.md                    Part 4 — pick the right model
05_standard_library.md               Part 5 — stdlib: just import
06_ecosystem_and_packaging.md        Part 6 — venv, pip, third-party libs
99_appendix_java_to_python.md        Master Java→Python lookup
RESTRUCTURE_PLAN.md                  (this file)
COVERAGE_MATRIX.md                   Phase 1 deliverable: every old §  → new home / dropped+reason
```

Old `Part1.md`–`Part5.md`: **delete** after Phase 3 (only after coverage matrix verifies nothing orphaned).

## Cross-cutting framing device (from peer review)

Throughout P2 and P3, open key sections with a **Java enterprise anti-pattern → Pythonic replacement** framing where it fits naturally. Not a structural axis (peers approved Q2 journey axis), but a section-opener device. Examples:

- `The Bean → @dataclass` (P2)
- `The Interface → typing.Protocol` (P3)
- `The Abstract Factory → first-class functions / closures` (P3)
- `ThreadLocal → contextvars.ContextVar` (P4)
- `@Cacheable / Guava Cache → @functools.cache / @lru_cache` (P3 decorators)
- `Spring component scanning → __init_subclass__ registry` (P2)
- `try-with-resources → with / async with` (P3 / P4)

Used as ~2-3 line opener, then the actual content. Sticky because it intercepts the moment a Java dev is about to write un-Pythonic code.

## Part-by-part content plan

### `01_syntax_shock.md` — survive reading Python code

Anchor for: syntax basics, variables, control flow, functions, `None`, truthiness, `is`/`==`, comprehensions (intro), exception handling, modules/imports, f-strings (min), walrus (min), `match` (basic), Unicode/`str` vs `bytes` (preserved from Part1 §3.8 — peer-flagged as critical to keep), `range`, `complex`, `zip`/`any`/`all`/`sorted`/`reversed`.

**Sources:** Part1 §1, §2, §3.6 (range), §3.7 (complex), §3.8 (Unicode), §4 (sequence utils), §5; Part3 §2, §3 (comprehensions intro), §7, §8, §9.

**New content:**
- f-string formatting depth (min, ~10 lines): `:>10`, `:.2f`, `{x=}`, nested
- Walrus `:=` (min)
- `raise from` exception chaining (min) + intro to `ExceptionGroup` / `except*` (🐍 3.11+, full in P4)
- Type hints — **starter teaser** only, full anchor in P3

**Teasers (forward pointers, 1 sentence + 1 link each):**
- `dataclass` → P2 §dataclass
- decorators → P3 §decorators
- context managers → P3 §context-managers
- type hints (full) → P3 §type-hints

**Key Takeaways:** indentation-driven syntax; `None`/`is`; truthiness as a feature; comprehensions over loops where clear; `if __name__ == "__main__"`; `str` ≠ `bytes` — encode/decode only at I/O boundaries.

---

### `02_java_idiom_translation.md` — translate Java reflexes

Anchor for: OOP (classes, inheritance, abstract classes), Java-style object model (`__str__`, `__repr__`, `__eq__`, `__hash__`, `__lt__`, `copy`/`deepcopy`), `@property`, `_protected`/`__private`, `@classmethod`/`@staticmethod`, dataclass, enum, `__slots__` (min), built-in collections vs `java.util` mapping (`HashMap`/`HashSet`/`ArrayList`/`Deque` → `dict`/`set`/`list`/`collections.deque`), mutable vs immutable, **class attr vs instance attr** (added from peer review), **MRO basics** (added), mutable default arguments pitfall, **`__init_subclass__`** (added — registry pattern, Spring-component-scan parallel, min).

**Sources:** Part1 §3 (collections — non-Unicode parts), §3.5 (binary data: `bytes`, `bytearray`, `memoryview`), §7 (OOP), §8 (special methods), §9 (class/static methods); Part3 §6 (mutable/immutable), §10 (dataclass).

**New content:**
- `enum` (medium): Enum, IntEnum, StrEnum (🐍 3.11+), `auto`, Java `enum` parallel
- `__slots__` (min): when memory matters, no Java parallel
- Composition over inheritance — promote from Part1 §7.6 footnote to subsection with concrete example
- **Class attr vs instance attr** (medium, new): a top-3 Java-dev pitfall — `class C: items = []` vs `self.items = []`
- **MRO basics** (medium, new): C3 linearization, `super()` in diamond inheritance — Java has no multi-inheritance so this is foreign
- **`__init_subclass__`** (min, new): registry hook — Spring component scan parallel
- Anti-pattern openers: `The Bean → @dataclass`, `Spring component scanning → __init_subclass__ registry`

**Teasers:** decorators (full) → P3; context managers (full) → P3.

**Key Takeaways:** dunder methods are hooks not magic; `dataclass(frozen=True)` ≈ Java `record` + equals/hashCode/toString; pick `dict` over a class for pure data; mutability is visible — design for it; class attrs are *shared*, not per-instance.

---

### `03_pythonic_idioms.md` — stop writing Java in Python

Anchor for: comprehensions (deep — dict/set/nested/generator expr), `*args`/`**kwargs` + unpacking, iterable vs iterator, generators (medium — `yield`, `yield from`, send/throw/close, generator expressions, pipelines), context managers + `@contextmanager` (medium), **`contextlib.ExitStack`** (min, added), decorators (deep), type hints (deep), `Protocol`/structural typing (medium), `match` patterns (medium — class patterns, OR, capture, guards), introspection/metaprogramming.

**Sources:** Part1 §6 (functions first-class), §11 (assert/loop-else/del/nonlocal), §12 (yield), §13 (reflection); Part3 §3 (comprehensions), §4 (args/kwargs), §5 (iterable/iterator), §11 (context managers).

**New content (the heavy lifting):**
- **Decorators (deep, new):** what they are, `@functools.wraps`, parameterized decorators, class decorators, stacking, decorator ordering pitfall, when to reach for them. Connect to `@property`, `@dataclass`, `@abstractmethod` already seen in P2. **Add: `@functools.cache` / `@lru_cache` / `@functools.cached_property`** (Guava Cache / `@Cacheable` parallel — peer-flagged "massive aha").
- **Type hints (deep, new):** beyond `def add(a:int)->int`. Expanded per peer review:
  - `Optional[X]` vs `X | None`
  - `Union` / `|`
  - `Generic` (legacy `TypeVar` form + 🐍 3.12 `def first[T](xs: list[T]) -> T` PEP 695)
  - `Protocol` for duck-typing-made-explicit (intersection with structural typing subsection)
  - `TypeAlias` / 🐍 3.12 `type X = ...`
  - `Self` (🐍 3.11+)
  - **`TypedDict`** (added)
  - **`Literal`** (added)
  - **`Callable`** (added)
  - **`Sequence` / `Mapping` vs concrete containers** (added — Java-dev habit is to declare `List<>` parameter; Python idiom is to accept the most general protocol)
  - **`cast`, `Any`** (added — escape hatches)
  - "types are not runtime validation" emphasis
  - runtime vs static checking, mypy intro
- **`@contextmanager` (medium, new):** writing your own context manager via generator.
- **`contextlib.ExitStack` (min, new):** dynamic context manager composition.
- **Generators deep (medium, expand):** `yield from`, generator expressions, pipeline pattern, lazy vs eager mental model.
- **`Protocol` (medium, new):** structural typing — the bridge between duck typing and static checking. Anti-pattern opener: `The Interface → typing.Protocol`.
- **`match` patterns (medium, expand):** class patterns, OR `|`, capture, guards.
- Anti-pattern openers: `The Interface → Protocol`, `The Abstract Factory → first-class functions / closures`, `@Cacheable → @functools.cache`.

**Key Takeaways:** comprehensions for transformation; generators for streaming; decorators for cross-cutting concerns; `Protocol` for explicit duck typing; type hints buy you mypy + IDE — they are *not* runtime validation (use pydantic for that).

---

### `04_concurrency.md` — pick the right model

Anchor for: threading + GIL, sync primitives (`Lock`, `RLock`, `Semaphore`, `Condition`, `Event`), thread safety + memory visibility, producer-consumer (`queue.Queue`), `ThreadPoolExecutor`, `async`/`await`/coroutines, `asyncio.gather`, `asyncio.Queue`, blocking vs non-blocking I/O, threads vs async vs multiprocessing chooser, mixing async + threads.

**Sources:** Part2 (entire, with dedup of GIL-in-§3 vs GIL-in-§4 — merge into single GIL section under "thread safety").

**New content (significantly expanded per peer review — async failure model was the biggest gap):**
- **Async + threads mixing (medium, new):** `asyncio.to_thread`, `run_in_executor`, when to offload blocking calls from event loop.
- **Async context managers (medium, new):** `async with`, `__aenter__` / `__aexit__`. Critical: Java devs lean on `try-with-resources`; async DB drivers and `aiohttp` require this.
- **Async iteration (min, new):** `async for`, `__aiter__` / `__anext__`, async generators.
- **Structured concurrency (medium, new):** `asyncio.TaskGroup` (🐍 3.11+) — the modern replacement for raw `gather` when failure semantics matter. Direct Java parallel: structured concurrency / `StructuredTaskScope`.
- **Cancellation & timeouts (medium, new):** `asyncio.timeout` (🐍 3.11+), `CancelledError`, cooperative cancellation, why `Thread.interrupt()` thinking doesn't translate.
- **`ExceptionGroup` / `except*` (medium, new, 🐍 3.11+):** how `TaskGroup` reports multi-task failures.
- **`contextvars.ContextVar` (min, new):** Java `ThreadLocal` parallel — also works with async (where `threading.local` does not).
- **GIL nuance + free-threaded Python (medium, expanded from min per peer review):** PEP 703 / 🐍 3.13+ optional free-threaded build. Architectural shock for Java devs is real — this changes the multi-core CPU story for Python. Cover: what changes, what doesn't, package compatibility caveats, thread-safety assumption shifts.
- Anti-pattern opener: `ThreadLocal → contextvars.ContextVar`.

**Dedup work:** GIL discussion currently appears in Part2 §3 ("When Python threads are useful") and §4 ("Thread safety in Python"). Consolidate to one anchor under thread safety; teaser from threads section.

**Key Takeaways:** threads for blocking I/O; async for non-blocking I/O; multiprocessing for CPU-bound; GIL ≠ thread safety; always synchronize shared mutable state; `TaskGroup` over raw `gather` when failures matter; cancellation is cooperative, not preemptive.

---

### `05_standard_library.md` — stdlib: just import

Anchor for: standard-library map (`pathlib`, `datetime`/`zoneinfo`, `json`, `re`, `math`/`random`/`statistics`, `logging`, `collections`, `os`/`subprocess`, `pickle` + safety warning, **`itertools`**, **`functools` utilities** beyond decorators, **`tempfile`**, **`csv`**, **`argparse`** — all added per peer review as "quick wins" subsection), env config (`os.environ`), file I/O via `pathlib` (single anchor — move from Part1 mention), JSON (single anchor — was duplicated Part1/Part4), logging config (medium — handlers, formatters, `dictConfig`).

**Sources:** Part4 (entire) + Part1 §10 (file/JSON — moved here, deduped).

**New content:**
- **`logging` config (medium):** handlers, formatters, `dictConfig`, why `basicConfig` runs out.
- **Stdlib quick-wins (medium subsection, new):** one-paragraph each with use case, no deep dive:
  - `itertools` — combinatorics, infinite iterators, `chain`/`groupby`/`islice`
  - `functools` non-decorator bits — `partial`, `reduce`, `singledispatch`
  - `tempfile` — secure temp files/dirs
  - `csv` — built-in CSV reader/writer
  - `argparse` — CLI args (when `typer` is overkill)
  - `subprocess` — safety: prefer list-form args, never `shell=True` with untrusted input
- **`contextlib.ExitStack` cross-link** (anchor in P3) — teaser only here.
- Env-config moved to P6 (`.env` + dotenv is third-party).

**Key Takeaways:** prefer stdlib first; `pathlib` over `os.path`; `dictConfig` once you outgrow `basicConfig`; `pickle` is unsafe — JSON for cross-system; `subprocess` list-form args always.

---

### `06_ecosystem_and_packaging.md` — venv, pip, third-party

Anchor for: **virtual envs + dependency management (medium, expanded per peer review)** — this is the cognitive hurdle for Java devs new to Python; deserves its own conceptual home, separate from stdlib. Then: Jupyter/IPython, NumPy, pandas, Polars, SciPy, viz (matplotlib/seaborn/plotly), HTTP (`requests`, `httpx`), web (Flask/FastAPI/Django), testing (`pytest` medium — fixtures, parametrize, conftest), tooling (`ruff`, `black`, `mypy`), ML/AI libraries, productivity libs (`rich`, `typer`, `pydantic`, `sqlalchemy`), env config via `python-dotenv` (moved from P4 in original Part4 §7).

**Sources:** Part4 §7 (env / `.env` / dotenv — moved here, third-party), Part5 (entire).

**New content:**
- **`venv` walkthrough (medium):** create, activate (mac/linux/windows differences), install, `requirements.txt`, pinning, deactivate. Why one-venv-per-project. Comparison to Maven local repo / Gradle wrappers.
- **`pyproject.toml` (medium):** modern packaging metadata, dependency groups, build backends, optional dependencies. Comparison to `pom.xml` / `build.gradle`.
- **`pip` essentials (min):** install, freeze, constraints, editable installs (`-e`).
- **Poetry / uv overview (min):** when stdlib `pip`+`venv` isn't enough, what each adds.
- **`pytest` fixtures + parametrize (medium):** beyond bare `assert`. `conftest.py`, fixture scopes, parametrize, marks. Comparison to JUnit `@BeforeEach` / `@ParameterizedTest`.
- **Stdlib-first reminder:** before reaching for a library, check P5 first.

**Key Takeaways:** one venv per project, always; `pytest` + `ruff` + `mypy` is the daily-driver trio; pick web framework by project size (Flask < FastAPI < Django); `pydantic` for runtime validation (type hints don't validate); Python ecosystem fragmentation is real but not crippling.

---

### `99_appendix_java_to_python.md` — master lookup

Single consolidated reference. Sections:

1. **Core language** — `String`→`str`, `Math`→`math`, `null`→`None`, `Object`→`object`, etc.
2. **Collections** — `HashMap`→`dict`, `HashSet`→`set`, `ArrayList`→`list`, `Deque`→`collections.deque`, etc.
3. **Object model** — `equals()`→`__eq__`, `hashCode()`→`__hash__`, `toString()`→`__str__`, `Comparable`→rich comparison / `key=`, `Cloneable`→`copy` module, `record`→`dataclass(frozen=True)`.
4. **Dates/time** — `LocalDateTime`→`datetime`, `ZonedDateTime`→`datetime` + `zoneinfo`, `Duration`→`timedelta`.
5. **Concurrency** — `synchronized`/`ReentrantLock`→`Lock`/`RLock`, `Semaphore`→`Semaphore`, `BlockingQueue`→`queue.Queue` / `asyncio.Queue`, `ExecutorService`→`ThreadPoolExecutor`, `CompletableFuture`→`async`+`await`, `StructuredTaskScope`→`asyncio.TaskGroup`, `ThreadLocal`→`contextvars.ContextVar`.
6. **I/O & paths** — `Path`/`Files`→`pathlib`, `BufferedReader`→`open(...)` context manager.
7. **Exception model** — checked vs unchecked → no checked; `throw`→`raise`, `catch`→`except`, exception chaining; multi-catch → `ExceptionGroup` / `except*`.
8. **Caching / interception** — `@Cacheable` / Guava Cache → `@functools.cache` / `@lru_cache`.
9. **Registry / scanning** — Spring component scan → `__init_subclass__` registry pattern.
10. **Tooling** — JUnit→`pytest`, Maven/Gradle→`pip`+`venv`+Poetry/uv, Checkstyle→`ruff`, formatter→`black`, compile-time checks→`mypy`, `pom.xml`/`build.gradle`→`pyproject.toml`.

Each row links to the anchor section.

---

### `00_index.md` — README

Sections:

1. **Who this is for** (1 paragraph).
2. **How to read this** — fast path (P1 + P3) vs deep path (all 6 in order). Per-part prerequisite chart.
3. **Topic map** — alphabetical: topic → file#anchor.
4. **Java-term map** — pointer to `99_appendix`.
5. **The Mindset Shift** — 5–7 cross-cutting bullets distilled from per-part Key Takeaways: favor built-ins; composition over inheritance; mutability is visible; GIL ≠ thread safety; type hints are optional but worth it; stdlib first; choose concurrency by I/O profile.
6. **Conventions** — explains the 4 callout types (`☕ Java parallel` / `⚠️ Pitfall` / `💡 Pythonic` / `🐍 Python 3.X+`), `<details>` deep-dive collapsibles, anchor naming, teaser format.

---

## Light-edit normalization rules (Q7)

Applied uniformly during Phase 2:

1. **Code examples:** show output explicitly via `print(...)` + `# >>> expected_output` comment style.
2. **"Java comparison:" prose** → `> ☕ **Java parallel:** ...` callout.
3. **"Important nuance:" / "Why this matters:"** → either `> 💡 **Pythonic:** ...` or `> ⚠️ **Pitfall:** ...` depending on intent. **At most one callout per `###` section** (peer-review revision; was `##`). Stacked callouts banned.
4. **Bullet lists that are tabular** → tables.
5. **Sectional "Practical Takeaways" + "Final Summary"** → replaced by `## Key Takeaways` (3–5 concrete part-specific bullets).
6. **Version-gated features** → `> 🐍 **Python 3.X+:** ...` callout.
7. **Teaser format (strict, peer-flagged):** one sentence + one link, no code blocks, no bullets, no examples. Example: `> See full treatment: [Decorators (P3)](03_pythonic_idioms.md#decorators).`

---

## Execution: 3 phases

### Phase 1 — Skeletons + Coverage Matrix (you review before Phase 2)

Two deliverables:

**(a) Coverage matrix (`COVERAGE_MATRIX.md`):** every existing `##`/`###` in current `Part1.md`–`Part5.md` mapped to one of:
- `MIGRATED` → new file#anchor
- `TEASER` → mentioned briefly in new file#section (with anchor target)
- `DROPPED` → explicit reason (redundant / obsolete / out of scope)

Format:

```
| Old location              | Action     | New home                                        | Notes              |
|---------------------------|------------|--------------------------------------------------|--------------------|
| Part1.md §3.5 binary data | MIGRATED   | 02_java_idiom_translation.md#binary-types       | bytes/bytearray/memoryview |
| Part1.md §3.8 Unicode     | MIGRATED   | 01_syntax_shock.md#str-vs-bytes                 | preserved per peer |
| ...                       | ...        | ...                                              | ...                |
```

**(b) Eight files (7 doc files + 1 matrix), all with:**
- Full `##` / `###` outline
- Anchor placement marked `<!-- ANCHOR -->`, teasers marked `<!-- TEASER → see <file>#<anchor> -->`
- Cross-reference links written (validated in Phase 3)
- Empty bodies
- TOCs at top
- README with reading paths skeleton

**Gate:** you review coverage matrix + skeletons before Phase 2 starts.

### Phase 2 — Bodies (batched)

Fill in prose, batch-by-batch:
- **Batch 2A:** `01_syntax_shock.md` + `02_java_idiom_translation.md`
- **Batch 2B:** `03_pythonic_idioms.md` (heaviest — new decorators + type-hints deep sections)
- **Batch 2C:** `04_concurrency.md` (heavy — async failure model new)
- **Batch 2D:** `05_standard_library.md` + `06_ecosystem_and_packaging.md`
- **Batch 2E:** `99_appendix_java_to_python.md` + `00_index.md` (last; depend on anchor IDs being stable)

Each batch: cut/paste from old `Part*.md` per coverage matrix + apply light-edit rules + add new content per gap bundle.

### Phase 3 — Polish

- Verify every cross-ref link resolves to a real anchor.
- Enforce callout cap (≤1 per `###`) and stacking ban.
- Enforce teaser format (≤1 sentence + 1 link).
- Normalize example output convention end-to-end.
- Verify coverage matrix accounts for everything in old `Part*.md`.
- Delete `Part1.md`–`Part5.md`.
- Sanity-read each part end-to-end for flow.

---

## Risks & mitigations

| Risk | Mitigation |
|---|---|
| Source material orphaned during migration | **Coverage matrix is Phase 1 gate.** Nothing deleted until matrix accounts for it. |
| Phase 1 skeleton wrong → Phase 2 wastes prose | Explicit gate at Phase 1 with your review |
| Anchor IDs unstable (heading renames break links) | Generate appendix + index **last** (Batch 2E) |
| Decorators / type-hints deep sections balloon | Cap each at ~200 lines; use `<details>` for advanced sub-cases |
| Teasers rot into mini-duplicates | Strict format enforced in Phase 3: 1 sentence + 1 link, no code/bullets |
| Existing strong voice flattened | Q7 = preserve, not rewrite |
| Callouts become noise | Cap ≤1 per `###`. No stacking. Phase 3 enforces. |
| Python version drift in examples | Q11 baseline = 3.11. Newer features get `🐍` callout. |
| Async failure model missing | Peer-review identified gap; explicit medium gaps for `TaskGroup`/`asyncio.timeout`/`ExceptionGroup`/`async with` |
| Stdlib vs ecosystem boundary blurred | Split into P5 + P6 per peer review |

## Out of scope

- Diagrams (rejected at Q5).
- Rewriting existing strong sections (rejected at Q7).
- Per-part "Final Summary" prose (rejected at Q8).
- Topic-only filenames without numbers (rejected at Q9).
- Python <3.11 compatibility (peer-review revision of Q11).
- Restructuring around "anti-patterns" as primary axis (peer-suggested; adopted as section-opener device only).

---

## Changes from peer review

Locked changes incorporated above:

1. Python 3.11 floor (was 3.10) — 3.10 EOL Oct 2026
2. Callout cap moved `##` → `###`, stacking banned
3. Strict teaser format: 1 sentence + 1 link
4. Coverage matrix added as Phase 1 gate (BLOCKER fix)
5. Async failure model: `TaskGroup` / `asyncio.timeout` / `ExceptionGroup`+`except*` / `async with` / `async for` / cancellation — all added to P4
6. GIL 3.13 free-threaded: min → medium
7. Typing additions to P3 deep section: `TypedDict`, `Literal`, `Callable`, `Sequence`/`Mapping` vs concrete, `cast`, `Any`
8. P2 additions: class attr vs instance attr (medium), MRO basics (medium), `__init_subclass__` (min)
9. P3 decorators deep section: `@functools.cache` / `@lru_cache` / `@cached_property` (Guava Cache parallel)
10. P4 additions: `contextvars.ContextVar` (min, ThreadLocal parallel)
11. P5 stdlib quick-wins subsection: `itertools`, `functools` non-decorator, `tempfile`, `csv`, `argparse`, `subprocess` safety
12. P3 additions: `contextlib.ExitStack` (min)
13. **Split P4+P5 merge back into 6 parts** — stdlib (just import) vs ecosystem (install first) is a real cognitive boundary for Java devs
14. Anti-pattern section-opener device adopted for P2/P3/P4 (Bean→dataclass, Interface→Protocol, Abstract Factory→closures, ThreadLocal→ContextVar, @Cacheable→@cache, Spring component scan→__init_subclass__)
