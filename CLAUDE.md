# Python for Busy Java Developers — Project Notes

A documentation-only repo. 6-part Markdown set + appendix + index + coverage matrix.
No source code, no build, no tests — only prose and cross-linked Markdown.

## Repo layout

- `00_index.md` — README, topic map, mindset shift, conventions
- `01_syntax_shock.md` → `06_ecosystem_and_packaging.md` — the 6 core parts
- `07_interoperability.md` — optional Part 7 on Python ↔ Java/Node/C++ integration (RAG use case)
- `99_appendix_java_to_python.md` — Java→Python lookup tables
- `docs/plans/RESTRUCTURE_PLAN.md` — 11-decision plan with peer-review changelog
- `docs/plans/COVERAGE_MATRIX.md` — every old `§` from `Part{1..5}.md` mapped to its new home; touched whenever new content is added

## Conventions (canonical: `00_index.md § Conventions`)

- **Baseline:** Python 3.11. Features needing 3.12+ get a `🐍 Python 3.12+` callout.
- **Callout types (4 only):** `☕ Java parallel`, `⚠️ Pitfall`, `💡 Pythonic`, `🐍 Python 3.X+`. Cap ≤1 per `###`; stacking discouraged but allowed when angles differ.
- **Teaser format (strict):** one sentence + one link. No code, no bullets.
- **Anchor hygiene:** NEVER put parenthetical content, slashes, or commas in headers — GitHub auto-slugs them into mangled IDs. Move qualifiers into the first body sentence. Round-2 peer review caught ~28 broken anchors from this exact mistake.

## Cross-ref verification

Run after any edit that adds/renames a section or adds a cross-file link:

```bash
broken=0
while IFS= read -r link; do
  file="${link%%#*}"; anchor="${link##*#}"
  [ ! -f "$file" ] && { echo "MISSING: $file"; broken=$((broken+1)); continue; }
  found=$(awk -v t="$anchor" '/^##+ /{h=$0;sub(/^##+ /,"",h);h=tolower(h);gsub(/ /,"-",h);gsub(/[^a-z0-9_-]/,"",h);if(h==t){print"F";exit}}' "$file")
  [ -z "$found" ] && { echo "BROKEN: $link"; broken=$((broken+1)); }
done < <(grep -hoE '[0-9]+_[a-z_]+\.md#[a-z0-9-]+' 0*.md 99*.md | sort -u)
echo "Broken: $broken"
```

Target: 0 broken before commit.

## Peer review workflow

Use `/hippoz-claude:consult codex,gemini <prompt>` for substantive content reviews.
Session files live under `.hippoz/consult/<session-id>/` (globally gitignored).
Reuse the same session-id across rounds for continuity; compress `session.md` between rounds when it exceeds ~8 KB (raw responses always preserved in `output/`).

## Workflow for adding new content

1. **Identify home** — which part owns this concept? Default: a topic lives in exactly one anchor section; other parts use a 1-sentence teaser pointing to it.
2. **Write with conventions** — clean header, ≤1 callout per `###`, code examples must run as written.
3. **Update `docs/plans/COVERAGE_MATRIX.md`** — add a row to the "New content" table with home + reason.
4. **Update `00_index.md` topic map** + `99_appendix_java_to_python.md` if there's a Java parallel.
5. **Run cross-ref verification** (script above).
6. **Stage explicitly** (`git add <files>`) — NOT `git add -u`, which sweeps unrelated deletions.
7. **Peer review** via `/hippoz-claude:consult` for non-trivial additions.

## Gotchas (from real review rounds)

- **`git add -u` sweeps unrelated deletions.** Stage doc files by name. Bit the project in `2a0e4ac` (Optional fix accidentally also deleted `ref/`).
- **Code examples must run as written.** Round 6 caught undefined `username` / `lines` / `my_list` / `urls`. Round 8 caught `itemgetter("city", ...)` against data missing `"city"`. Don't leave placeholder names unbound.
- **`assert` is NOT Python-specific.** Java has it too — opt-in (`-ea`) vs Python opt-out (`-O`). Don't list it as a Python-only surprise. (Round 8, user-flagged.)
- **`@functools.cache` is NOT exactly-once.** Two threads hitting cold cache can both build the value. Don't frame it as a thread-safe singleton factory.
- **`Sequence[int]` vs `Iterable[int]`** — `Sequence` does NOT accept generators (needs `len`/indexing). Use `Iterable` for parameters that only iterate.
- **`functools.reduce` examples must justify reduce.** If the example reduces to `sum`/`max`/`min`/`math.prod`/`sum(..., start=)`, it's an anti-pattern — pick a case with no built-in (set intersection, dict merge).
- **Anchor IDs are auto-derived.** Header `## File I/O via pathlib` becomes `#file-io-via-pathlib`, NOT `#pathlib`. Either rename the header to match or accept the auto-slug everywhere it's referenced.
- **Don't claim "double-checked" without writing real DCL.** Outer unlocked check → lock → inner re-check. Lock-then-check is a synchronized bottleneck, not DCL. (Round 7.)
- **Python conditional `a if c else b` is lazy on the unused branch.** Java `Optional.orElse(d)` is eager only because `d` is an argument (evaluated before the call), not because the conditional is eager. Don't conflate. (Round 9.)
- **Python `int` has NO machine-int fast path.** Every `int` is a `PyLong` with variable-length digits — small values just use one digit. Don't claim CPython "switches to arbitrary precision when needed"; it was arbitrary-precision the whole time. (Round 10.)
- **Py4J is a local socket bridge, NOT in-process.** Two CPython processes (Python + JVM) talking via TCP/Unix socket. The JVM-embeds-Python options are GraalPy (Python on JVM) and Jython (Python-as-JVM-bytecode, Py2-stuck). The Python-embeds-JVM option is JPype. Don't lump Py4J with in-process. (Round 10–11.)
- **GraalPy's GIL is for C-extension compatibility, not "PEP 703 phase-out."** PEP 703 is CPython-specific. GraalPy keeps its GIL waiting on CPython's PEP 703 rollout + ecosystem catch-up. Don't misattribute. (Round 10.)
- **GIL invisibility ≠ "requests served serially per-process."** A single ASGI worker handles concurrent I/O-bound requests via asyncio just fine; the GIL only blocks parallel Python *bytecode*. Workers (`uvicorn --workers N`) are for CPU-bound parallelism. (Round 15.)
- **"Every public LLM API uses SSE" is false.** OpenAI and Anthropic chat-completions APIs use SSE for streaming. Gemini uses gRPC streaming. Others vary. Narrow these claims. (Round 10–11.)
- **Stale-text-after-rename:** when renaming a section (e.g., Sidecar → Python-as-service), `grep -r <old-term>` across all files including `00_index.md`, `docs/plans/COVERAGE_MATRIX.md`, `99_appendix_java_to_python.md`, and Key Takeaways. Each post-Part-7-rename round caught 2-5 lingering refs in places I'd forgotten. (Rounds 11, 13, 14.)
- **Converge loops take more rounds than expected.** Part 7 needed 7 review rounds to land clean — each fix can introduce or reveal adjacent stale text. Always run a final cross-file `grep` after a significant rename, and budget for multiple verification rounds on non-trivial additions.

## Out of scope for this repo

- No source code, no Python interpreter required to build.
- No CI — review is gate-by-peer-review, not gate-by-tests.
- No `requirements.txt` / `pyproject.toml` — this is prose only.
