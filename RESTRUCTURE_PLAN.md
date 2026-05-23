# Restructure Plan: Python for Busy Java Developers

## Goal

Rebuild `Part1.md`–`Part5.md` into a comprehensive, cohesive, coherent, DRY document set, with strong navigability for a busy Java-developer reader.

## Decisions (locked via grilling)

| # | Decision | Choice |
|---|---|---|
| Q1 | Scope | **Full restructure** (re-slice by topic axis, extract shared appendix) |
| Q2 | Part axis | **Java-developer journey** — 5 parts |
| Q3 | Topic placement | **Anchor + teaser** — single source of truth, forward-pointing 1-line teasers allowed |
| Q4 | Java↔Python comparisons | **Distributed + appendix dual** — per-part overview tables + master appendix |
| Q5 | Navigability | TOCs, README index, 3 prose callout types (`☕ Java parallel`, `⚠️ Pitfall`, `💡 Pythonic`), `<details>` for deep dives, **no diagrams** |
| Q6 | Gap depth | Targeted bundle: **deep** for type hints + decorators; **medium** for ~9 mid-leverage gaps; **min** for one-liners |
| Q7 | Voice | **Preserve + light edit** — keep existing prose, normalize examples and callouts |
| Q8 | Epilogues | Per-part **Key Takeaways** (3–5 part-specific bullets) + global **Mindset Shift** in README |
| Q9 | Filenames | **Numbered topical**, lowercase, sortable |
| Q10 | Execution | **3-phase**: skeletons → bodies → polish, with gates |
| Q11 | Python baseline | **3.10**, with `🐍 Python 3.11+ / 3.12+` callouts for newer features |

## Target file layout

```
00_index.md                          README + reading paths + cross-cutting "Mindset Shift"
01_syntax_shock.md                   Part 1
02_java_idiom_translation.md         Part 2
03_pythonic_idioms.md                Part 3
04_concurrency.md                    Part 4
05_stdlib_and_ecosystem.md           Part 5
99_appendix_java_to_python.md        Master Java→Python lookup
RESTRUCTURE_PLAN.md                  (this file)
```

Old `Part1.md`–`Part5.md`: **delete** after migration.

## Part-by-part content plan

### `01_syntax_shock.md` — survive reading Python code

Anchor for: syntax basics, variables, control flow, functions, `None`, truthiness, `is`/`==`, comprehensions (intro), exception handling, modules/imports, f-strings (min), walrus (min), `match` (basic).

**Sources:** Part1 §1, §2, §5; Part3 §2, §3 (comprehensions intro), §7, §8, §9.

**New content:**
- f-string formatting depth (min, ~10 lines)
- Walrus `:=` (min)
- `raise from` exception chaining (min)
- Type hints — **starter teaser**, full anchor in Part 3

**Teasers (forward pointers):** dataclass → P2; decorators → P3; context managers → P3; type hints full → P3.

**Key Takeaways:** 3–5 bullets covering: indentation-driven syntax, `None`/`is`, truthiness as a feature, comprehensions over loops where clear, `if __name__ == "__main__"`.

---

### `02_java_idiom_translation.md` — translate Java reflexes

Anchor for: OOP (classes, inheritance, abstract classes), Java-style object model (`__str__`, `__repr__`, `__eq__`, `__hash__`, `__lt__`, `copy`/`deepcopy`), `@property`, `_protected`/`__private`, `@classmethod`/`@staticmethod`, dataclass, enum, `__slots__` (min), built-in collections vs `java.util` mapping, mutable vs immutable.

**Sources:** Part1 §3 (collections), §7 (OOP), §8 (special methods), §9 (class/static methods); Part3 §6 (mutable/immutable), §10 (dataclass).

**New content:**
- `enum` (medium): Enum, IntEnum, StrEnum (🐍 3.11+), `auto`, Java `enum` parallel
- `__slots__` (min): when memory matters, Java parallel = none
- Composition over inheritance — promote from Part1 §7.6 footnote to real subsection with a concrete example

**Teasers:** decorators full → P3; context managers full → P3.

**Key Takeaways:** dunder methods are hooks not magic; `dataclass(frozen=True)` ≈ Java record + equals/hashCode/toString; pick `dict` over a class for pure data; mutability is visible — design for it.

---

### `03_pythonic_idioms.md` — stop writing Java in Python

Anchor for: comprehensions (deep — dict/set/nested/generator expr), `*args`/`**kwargs` + unpacking, iterable vs iterator, generators (medium — `yield`, `yield from`, send/throw/close, generator expressions, pipelines), context managers + `@contextmanager` (medium), decorators (deep), type hints (deep — `Optional`, `Union`/`|`, `Generic`, `Protocol`, `TypeAlias`, `Self`, mypy), `Protocol`/structural typing (medium), `match` patterns (medium — class patterns, OR, capture, guards), introspection/metaprogramming.

**Sources:** Part1 §6 (functions first-class), §11 (assert/loop-else/del/nonlocal), §12 (yield), §13 (reflection); Part3 §3 (comprehensions), §4 (args/kwargs), §5 (iterable/iterator), §11 (context managers).

**New content (the heavy lifting):**
- **Decorators (deep, new):** what they are, `@functools.wraps`, parameterized decorators, class decorators, stacking, when to reach for them. Connect to `@property`, `@dataclass`, `@abstractmethod` already seen in P2.
- **Type hints (deep, new):** beyond `def add(a:int)->int`. `Optional[X]` vs `X | None`, generics, `Protocol` for duck-typing-made-explicit, `TypeAlias`, `Self`, runtime vs static checking, mypy intro, 🐍 3.12 `type X = ...` and `def first[T](xs: list[T]) -> T`.
- **`@contextmanager` (medium, new):** writing your own context manager via generator.
- **Generators deep (medium, expand):** `yield from`, generator expressions, pipeline pattern, lazy vs eager mental model.
- **`Protocol` (medium, new):** structural typing — the bridge between duck typing and static checking.
- **`match` patterns (medium, expand):** class patterns, OR `|`, capture, guards.

**Key Takeaways:** comprehensions for transformation, generators for streaming, decorators for cross-cutting, `Protocol` for explicit duck typing, type hints buy you mypy + IDE.

---

### `04_concurrency.md` — pick the right model

Anchor for: threading + GIL, sync primitives (`Lock`, `RLock`, `Semaphore`, `Condition`, `Event`), thread safety + memory visibility, producer-consumer (`queue.Queue`), `ThreadPoolExecutor`, `async`/`await`/coroutines, `asyncio.gather`, `asyncio.Queue`, blocking vs non-blocking I/O, threads vs async vs multiprocessing chooser, mixing async + threads.

**Sources:** Part2 (entire, with dedup of GIL-in-§3 vs GIL-in-§4 — merge into single GIL section under "thread safety").

**New content:**
- **Async + threads mixing (medium, new):** `asyncio.to_thread`, `run_in_executor`, when to offload blocking calls from event loop.
- **GIL nuance + free-threaded Python (min, new):** PEP 703 / 3.13+ optional free-threaded build, what changes and what doesn't.
- Concurrency chooser stays as table (no mermaid per Q5).

**Dedup work:** GIL discussion currently appears in Part2 §3 (within "When Python threads are useful") and Part2 §4 ("Thread safety in Python"). Consolidate to one anchor under thread safety, teaser from threads section.

**Key Takeaways:** threads for blocking I/O, async for non-blocking I/O, multiprocessing for CPU-bound, GIL ≠ thread safety, always synchronize shared mutable state.

---

### `05_stdlib_and_ecosystem.md` — what to reach for

Anchor for: standard-library map (`pathlib`, `datetime`/`zoneinfo`, `json`, `re`, `math`/`random`/`statistics`, `logging`, `collections`, `os`/`subprocess`, `pickle` + safety warning), env config (`os.environ` + `.env`), file I/O via `pathlib` (move from Part1 mention to here), JSON (single anchor, was duplicated Part1/Part4), logging config (medium — handlers, formatters, dictConfig), Jupyter/IPython, NumPy, pandas, Polars, SciPy, viz (matplotlib/seaborn/plotly), HTTP (`requests`, `httpx`), web (Flask/FastAPI/Django), testing (`pytest` medium — fixtures, parametrize, conftest), tooling (`ruff`, `black`, `mypy`), packaging + envs (medium — `venv` walkthrough, `requirements.txt`, intro to `pyproject.toml`, Poetry/uv mention), ML/AI libraries, productivity libs (`rich`, `typer`, `pydantic`, `sqlalchemy`).

**Sources:** Part4 + Part5 (merged into one part; current split is artificial — both are "the libraries you'll reach for").

**Note on merge:** Part4 (stdlib) and Part5 (ecosystem) are pedagogically the same topic ("here's the toolbox"). The boundary between "stdlib" and "third-party" matters less than reader thinks. One part with clear stdlib-first / ecosystem-second sections is more navigable.

**New content:**
- **`venv` walkthrough (medium):** create, activate (mac/linux/windows differences), install, `requirements.txt`, pinning, deactivate.
- **`pytest` fixtures + parametrize (medium):** beyond bare `assert`.
- **`logging` config (medium):** handlers, formatters, `dictConfig`, why `basicConfig` runs out.
- **Packaging (min):** what `pyproject.toml` is, when to publish.

**Key Takeaways:** prefer stdlib first; `pathlib` over `os.path`; `pytest` + `ruff` + `mypy` is the daily-driver trio; `venv` per project always; pick web framework by project size.

---

### `99_appendix_java_to_python.md` — master lookup

Single consolidated reference. Sections:

1. **Core language** — `String`→`str`, `Math`→`math`, `null`→`None`, `Object`→`object`, etc.
2. **Collections** — `HashMap`→`dict`, `HashSet`→`set`, `ArrayList`→`list`, `Deque`→`collections.deque`, etc.
3. **Object model** — `equals()`→`__eq__`, `hashCode()`→`__hash__`, `toString()`→`__str__`, `Comparable`→rich comparison / `key=`, `Cloneable`→`copy` module, record→`dataclass(frozen=True)`.
4. **Dates/time** — `LocalDateTime`→`datetime`, `ZonedDateTime`→`datetime` + `zoneinfo`, `Duration`→`timedelta`.
5. **Concurrency** — `synchronized`/`ReentrantLock`→`Lock`/`RLock`, `Semaphore`→`Semaphore`, `BlockingQueue`→`queue.Queue` / `asyncio.Queue`, `ExecutorService`→`ThreadPoolExecutor`, `CompletableFuture`→async + `await`.
6. **I/O & paths** — `Path`/`Files`→`pathlib`, `BufferedReader`→`open(...)` context manager.
7. **Exception model** — checked vs unchecked → no checked; `throw`→`raise`, `catch`→`except`, exception chaining.
8. **Tooling** — JUnit→`pytest`, Maven/Gradle→`pip`+`venv`+Poetry/uv, Checkstyle→`ruff`, formatter→`black`, compile-time checks→`mypy`.

Each row links to the anchor section (`[full treatment](03_pythonic_idioms.md#decorators)`).

---

### `00_index.md` — README

Sections:

1. **Who this is for** (1 paragraph).
2. **How to read this** — fast path (just P1+P3) vs deep path (all 5 in order).
3. **Topic map** — alphabetical: topic → file#anchor. (e.g., "decorators → 03 §decorators".)
4. **Java-term map** — pointer to `99_appendix`.
5. **The Mindset Shift** — 5–7 cross-cutting bullets distilled from per-part Key Takeaways: favor built-ins; composition over inheritance; mutability is visible; GIL ≠ thread safety; type hints are optional but worth it; stdlib first; choose concurrency by I/O profile.
6. **Conventions** — explains the 4 callout types (`☕ Java parallel` / `⚠️ Pitfall` / `💡 Pythonic` / `🐍 Python 3.X+`), the `<details>` deep-dive collapsibles, anchor naming.

---

## Light-edit normalization rules (Q7)

Applied uniformly during Phase 2:

1. **Code examples:** show output explicitly via `print(...)` + `# >>> expected_output` style comment. Inconsistent in current docs.
2. **"Java comparison:" prose** → `> ☕ **Java parallel:** ...` callout.
3. **"Important nuance:" / "Why this matters:"** → either `> 💡 **Pythonic:** ...` or `> ⚠️ **Pitfall:** ...` depending on intent. At most **one callout per `##` section** (discipline).
4. **Bullet lists that are tabular** → tables (e.g., current Part5 §6 web frameworks).
5. **Sectional "Practical Takeaways" + "Final Summary"** → replaced by `## Key Takeaways` (3–5 bullets, part-specific only).
6. **Version-gated features** → `> 🐍 **Python 3.11+:** ...` callout.

---

## Execution: 3 phases

### Phase 1 — Skeletons (single deliverable, you review)

Produce all 7 files with:
- Full `##` / `###` outline
- Anchor placement marked `<!-- ANCHOR -->`, teasers marked `<!-- TEASER → see <file>#<anchor> -->`
- Cross-reference links written (validate in Phase 3)
- Empty bodies
- TOCs at top
- README with reading paths skeleton

**Gate:** you approve the structure before Phase 2.

### Phase 2 — Bodies (parallelizable, batched)

Fill in prose, batch-by-batch:
- **Batch 2A:** `01_syntax_shock.md` + `02_java_idiom_translation.md`
- **Batch 2B:** `03_pythonic_idioms.md` (heaviest — new decorators + type-hints deep sections)
- **Batch 2C:** `04_concurrency.md` + `05_stdlib_and_ecosystem.md`
- **Batch 2D:** `99_appendix_java_to_python.md` + `00_index.md` (last, depends on anchor IDs being stable)

Each batch: cut/paste from old `Part*.md` per source mapping above + apply light-edit rules + add new content per gap bundle.

### Phase 3 — Polish

- Verify every cross-ref link resolves to a real anchor.
- Dedupe callouts (rule: ≤1 per `##`).
- Normalize example output convention end-to-end.
- Delete `Part1.md`–`Part5.md`.
- Sanity-read each part end-to-end for flow.

---

## Risks & mitigations

| Risk | Mitigation |
|---|---|
| Phase 1 skeleton wrong → Phase 2 wastes prose | Explicit gate at Phase 1 with your review |
| Anchor IDs unstable (heading renames break links) | Generate appendix + index **last** in Phase 2 (Batch 2D) |
| Decorators / type-hints sections balloon | Cap each at ~150 lines; use `<details>` for advanced sub-cases |
| Existing strong voice flattened | Q7 = preserve, not rewrite. Edits limited to normalization rules only. |
| Callouts become noise | Rule: ≤1 callout per `##`. Enforced in Phase 3. |
| Python version drift in examples | Q11 baseline = 3.10. Any newer feature gets `🐍` callout. |

## Out of scope

- Diagrams (rejected at Q5).
- Rewriting existing strong sections (rejected at Q7).
- Per-part "Final Summary" prose (rejected at Q8).
- Topic-only filenames without numbers (rejected at Q9).
- Python <3.10 compatibility (rejected at Q11).

---

## Open items for peer review

The grill resolved 11 decisions. Peer review should stress-test:

1. **Q2 axis (Java-developer journey) vs. concept-layers (A) or skill-ladder (C).** Did we pick the right axis for "busy Java developer"?
2. **Q3 anchor+teaser** — does this actually beat single-home? Risk: teaser+anchor still creates 2 places to maintain.
3. **Part4/Part5 merge into `05_stdlib_and_ecosystem.md`.** Stdlib and 3rd-party arguably *should* stay split (different install/governance story). Worth defending or splitting back?
4. **Gap bundle (Q6 = B).** Did we under-cover anything survival-critical? Over-cover anything?
5. **Per-part Key Takeaways (Q8 = C)** — does 3–5 bullets really beat killing the section entirely?
6. **Execution staging (Q10 = C, 3-phase).** Is the Phase 1 skeleton gate overhead worth it, or is the doc small enough that staged-by-file (B) is better?
