# Part 4 — Concurrency

Goal: pick the right concurrency model for the workload, then know enough of each to use it safely. Cover threads, async, and multiprocessing — and where each fits.

**Prerequisites:** [Part 3 — Pythonic Idioms](03_pythonic_idioms.md) (generators, iterables, context managers all feed coroutines). **Next:** [Part 5 — Standard Library](05_standard_library.md).

---

## Table of Contents

- [Core differences](#core-differences)
- [Threading](#threading)
- [Threading pitfalls](#threading-pitfalls)
- [GIL](#gil)
- [Thread safety](#thread-safety)
- [Memory visibility](#memory-visibility)
- [Sync primitives](#sync-primitives)
- [Producer-consumer](#producer-consumer)
- [Thread pools](#thread-pools)
- [Coroutines](#coroutines)
- [Async and await](#async-and-await)
- [When to use async](#when-to-use-async)
- [Gather vs TaskGroup](#gather-vs-taskgroup)
- [Async context managers](#async-context-managers)
- [Async file I/O](#async-file-io)
- [Cancellation and timeouts](#cancellation-and-timeouts)
- [Exception groups](#exception-groups)
- [Contextvars](#contextvars)
- [Async-aware logging](#async-aware-logging)
- [Blocking vs non-blocking I/O](#blocking-vs-non-blocking-io)
- [Mixing async and threads](#mixing-async-and-threads)
- [Reactive programming](#reactive-programming)
- [Concurrency chooser](#concurrency-chooser)
- [Stdlib modules summary](#stdlib-modules-summary)
- [Key Takeaways](#key-takeaways)

---

## Core differences

| Topic | Java View | Python View |
| :--- | :--- | :--- |
| Threading | General-purpose concurrency primitive | Good for I/O-bound; limited by GIL for CPU-heavy work |
| Async coroutines | Roughly similar to `CompletableFuture` workflows | Event-loop based, lightweight, best for non-blocking I/O |
| Blocking I/O | Common with threads | Also common in Python threads |
| Non-blocking I/O | Async frameworks / NIO style | Natural fit for `async` / `await` |
| Synchronization | `synchronized`, `Lock`, `Semaphore`, `Condition` | `Lock`, `RLock`, `Semaphore`, `Condition`, `Event`, `Barrier`, `Queue` |
| Thread pools | `ExecutorService` | `concurrent.futures.ThreadPoolExecutor` |
| Producer-consumer | `BlockingQueue` | `queue.Queue` / `asyncio.Queue` |
| Structured concurrency | `StructuredTaskScope` (Loom, recent) | `asyncio.TaskGroup` (🐍 3.11+) |
| Memory model | Formal JMM + happens-before | No directly-comparable language model; rely on sync primitives |

```java
ExecutorService pool = Executors.newFixedThreadPool(3);
```

```python
from concurrent.futures import ThreadPoolExecutor

with ThreadPoolExecutor(max_workers=3) as executor:
    pass
```

## Threading

Python's `threading` module gives you real OS threads:

```python
import threading
import time

def worker(name):
    print(f"Worker {name} running")
    time.sleep(1)
    print(f"Worker {name} done")

t1 = threading.Thread(target=worker, args=("A",))
t2 = threading.Thread(target=worker, args=("B",))

t1.start()
t2.start()
t1.join()
t2.join()
```

**When Python threads are useful** — they shine when work is **waiting** rather than computing:

- network requests
- socket I/O
- file I/O
- database calls
- shelling out to external commands
- background monitoring / housekeeping

So yes: Python threads still use blocking I/O APIs much like Java threads. When one thread is blocked on a socket read, others can run. That's why threads remain useful for I/O-bound work even with the GIL.

## Threading pitfalls

### 1. Expecting CPU-bound work to scale

In Java, more threads → more CPU throughput on multi-core. In CPython, CPU-bound Python code does **not** scale that way because of the GIL ([next section](#gil)). If your work is pure-Python computation, threads won't help — reach for `multiprocessing` or a native library that releases the GIL.

### 2. Shared mutable state

Threads share memory. The usual concurrency bugs all apply — races, lost updates, inconsistent reads, deadlocks. The GIL does **not** protect you:

```python
counter = 0

def increment():
    global counter
    for _ in range(100_000):
        counter += 1                    # NOT atomic: read, add, write
```

Run this from 4 threads and the final `counter` is usually less than 400,000. Fix it with a `Lock` (see [Sync primitives](#sync-primitives)).

### 3. Forgetting `join()`

If you start workers and exit `main` without joining them, the process may end with work unfinished or in surprising states. Always `join()` or use a `ThreadPoolExecutor` context manager that handles it for you.

### 4. Framework assumptions

Many libraries (UI toolkits in particular) assume "this object is only touched from one thread." That's true in Python too. Don't mutate widgets or session-bound DB connections from random threads.

## GIL

CPython has a **Global Interpreter Lock**: only one thread executes Python bytecode at a time within a single interpreter process. This is an implementation detail of CPython — not a language guarantee.

Practical consequences:
- I/O-bound work scales fine (threads release the GIL while waiting on I/O syscalls).
- CPU-bound pure-Python work does **not** scale with threads — you'll see one core's worth of work no matter how many threads.
- Native code (NumPy, Pillow, cryptography, many compiled extensions) often releases the GIL during heavy compute, so vectorized work *can* parallelize.

> ⚠️ **Pitfall:** The GIL is an **interpreter execution lock**, not application-level mutual exclusion. It does NOT make your code thread-safe. Treating "I have the GIL" as a substitute for a `Lock` is the most common Java-to-Python concurrency mistake — see [Thread safety](#thread-safety).

> 🐍 **Python 3.13+:** PEP 703 introduces an optional **free-threaded** CPython build (`python3.13t`), where the GIL can be disabled. It's experimental: many C extensions are not yet free-threading-safe, single-thread performance drops modestly, and "GIL = atomicity" assumptions in third-party code can manifest as new races. For now, treat it as a forward-looking option, not the default. The thread-safety rules in the rest of this part don't change — they were never the GIL's job.

## Thread safety

The misunderstanding to avoid:

> "Python has a GIL, so thread safety is not my problem."

Wrong. The GIL serializes bytecode execution, not multi-step operations like `counter += 1` (which is three bytecodes: load, add, store). Race conditions on shared mutable state are real.

**Unsafe** — race condition example:

```python
import threading

counter = 0

def increment():
    global counter
    for _ in range(100_000):
        counter += 1

threads = [threading.Thread(target=increment) for _ in range(4)]
for t in threads: t.start()
for t in threads: t.join()

print(counter)                          # often < 400000; varies by interpreter and load
```

On modern CPython this race won't always reproduce — the GIL can let `counter += 1` finish before a context switch in some runs. Repeat the loop many times, or widen the critical section (e.g. add a brief computation between the read and write), and the lost updates become visible. The point isn't that the demo *always* fails — it's that correctness *can't depend* on what the GIL happens to do.

**Safe** — protect the read-modify-write with a `Lock`:

```python
import threading

counter = 0
lock = threading.Lock()

def increment():
    global counter
    for _ in range(100_000):
        with lock:
            counter += 1

threads = [threading.Thread(target=increment) for _ in range(4)]
for t in threads: t.start()
for t in threads: t.join()

print(counter)                          # 400000, always
```

> ☕ **Java parallel:** "GIL ≠ `synchronized`." Treat the GIL as a CPython performance characteristic, not a correctness guarantee. Shared mutable state still requires explicit synchronization.

The most-used rules:
- Immutable data is safe to share.
- Mutable shared state needs a lock or a thread-safe container.
- Atomic-*looking* operations (`x += 1`, `dict.setdefault`, list append-then-pop) are usually not atomic at the Python level.

## Memory visibility

In Java you reason in terms of the Java Memory Model: visibility, ordering, `volatile`, `synchronized`, happens-before edges. Python does not expose an equivalent everyday model.

The practical Python question is simpler:

> **Have I used a synchronization primitive to coordinate access and communication between threads?**

Whenever you use `Lock`, `Condition`, `Event`, or `Queue`, you're not just preventing simultaneous access — you're establishing a coordination boundary. That's the Pythonic substitute for thinking about memory-visibility rules.

**Unsafe pattern** — unsynchronized flag:

```python
ready = False
value = 0

def writer():
    global ready, value
    value = 42
    ready = True

def reader():
    while not ready:
        pass
    print(value)
```

This may appear to work but it's the kind of thing Java developers would (rightly) distrust without a happens-before edge.

**Safe pattern with `Event`:**

```python
import threading

ready = threading.Event()
value = 0

def writer():
    global value
    value = 42
    ready.set()

def reader():
    ready.wait()
    print(value)
```

> 💡 **Pythonic:** If your Java instinct is "I need a happens-before relationship here," your Python instinct should become "I need a proper synchronization primitive here." `Event` for one-time signals, `Queue` for handoff, `Condition` for state transitions, `Lock` for critical sections.

## Sync primitives

`threading` offers the same family of primitives you'd expect from `java.util.concurrent`.

### `Lock`

Basic mutual exclusion. Reentrant by the same thread? No — see `RLock`.

```python
import threading

lock = threading.Lock()
with lock:
    # critical section
    ...
```

> ☕ **Java parallel:** No direct counterpart — Java's `synchronized` and `ReentrantLock` are both reentrant. The closest equivalent is a `Semaphore(1)` used as a non-reentrant binary lock. See the next subsection for why Python's default differs.

### `RLock`

Re-entrant — the holding thread can `acquire()` it again without deadlocking.

```python
lock = threading.RLock()

def outer():
    with lock:
        inner()

def inner():
    with lock:                          # safe re-entry
        ...
```

**Rule of thumb: prefer `Lock`; reach for `RLock` only when you must.** `RLock` looks like the safer default — it can't deadlock against itself — but it carries real costs:

- **Runtime overhead** — every acquire/release checks the owning thread and bumps a recursion counter. `Lock` skips both. The gap is small per call but compounds in hot paths.
- **Architectural smell** — needing reentrancy usually means a locked public method is calling another locked public method on the same object. The cleaner fix is the **locked-public / unlocked-private** split: public methods acquire the lock and delegate to a `_locked_*` helper that assumes the lock is already held. Plain `Lock` forces you to notice.
- **Predictability** — `Lock` makes critical sections explicit and short. `RLock` invites sprawling, recursively-locked code paths that are harder to reason about.

Reach for `RLock` only when recursion is genuinely unavoidable — e.g., walking a tree where each node's method locks itself and recurses into children, and the locked/unlocked split would force you to thread the lock through every signature.

> ☕ **Java parallel:** `Lock` ≈ a non-reentrant primitive (Java doesn't ship one in `java.util.concurrent` — you'd build it from `Semaphore(1)`). `RLock` ≈ `ReentrantLock`. The convention is inverted, though: Java treats reentrancy as the default — `synchronized` is reentrant, `ReentrantLock` is the canonical `Lock` implementation, and most code reaches for them without a second thought. In Python, the idiomatic default is the non-reentrant `Lock`; `RLock` is a deliberate exception.

### `Semaphore`

Limits how many threads can be in a region at once:

```python
import threading
sem = threading.Semaphore(2)            # at most 2 concurrent

def worker(name):
    with sem:
        ...                             # only 2 workers in here at a time
```

> ☕ Direct analog of `java.util.concurrent.Semaphore`.

### `Condition`

Wait until a state predicate becomes true; signal others when it does. Combines a wait/notify with a lock.

```python
import threading

cond = threading.Condition()
items = []

def consumer():
    with cond:
        while not items:
            cond.wait()
        item = items.pop(0)

def producer():
    with cond:
        items.append("task")
        cond.notify()
```

> ☕ Conceptually similar to `wait`/`notify` on a monitor, or `Condition` paired with a `Lock`.

### `Event`

A simple one-bit signal. One thread sets; others wait.

```python
import threading

ready = threading.Event()

def waiter():
    ready.wait()
    print("Go!")

def signaler():
    ready.set()
```

> ☕ Closest to `CountDownLatch(1)`.

### `Barrier`

Synchronizes N threads — they all block until N have arrived.

```python
import threading

barrier = threading.Barrier(parties=3)

def worker(name):
    print(f"{name} at barrier")
    barrier.wait()                      # blocks until 3 threads here
    print(f"{name} past barrier")
```

> ☕ Direct analog of `CyclicBarrier`. **And** the right primitive for the general `CountDownLatch(N)` case where `Event` is not enough. (For `CountDownLatch(1)` use `Event`. For arbitrary counts where parties all need to wait until everyone arrives, use `Barrier`. For a counter that hits zero and signals once, build it with a `Condition` + integer.)

## Producer-consumer

A classic threading pattern. `queue.Queue` is thread-safe and handles all the wait/notify for you.

```python
import queue
import threading
import time

q = queue.Queue()

def producer():
    for item in range(5):
        print(f"Producing {item}")
        q.put(item)
        time.sleep(0.5)
    q.put(None)                         # sentinel: "done"

def consumer():
    while True:
        item = q.get()
        if item is None:
            q.task_done()
            break
        print(f"Consuming {item}")
        time.sleep(1)
        q.task_done()

t_p = threading.Thread(target=producer)
t_c = threading.Thread(target=consumer)
t_p.start(); t_c.start()
t_p.join(); q.join(); t_c.join()
```

**Why not a shared `list` + `Lock`?** Because you'd be reinventing all of: synchronization, waiting for non-empty, wake-up logic, shutdown signaling. `Queue` already solves these correctly.

> 💡 **Pythonic:** When a standard-library concurrency primitive already models your problem, use it. Don't reinvent low-level coordination unless you genuinely need to.

### `queue.Queue` vs `asyncio.Queue`

- `queue.Queue` — for threads (blocking `get`/`put`).
- `asyncio.Queue` — for coroutines (`await q.get()` / `await q.put()`).

**Same producer-consumer pattern, async flavor:**

```python
import asyncio

async def producer(q):
    for item in range(5):
        await q.put(item)
        await asyncio.sleep(0.2)
    await q.put(None)

async def consumer(q):
    while True:
        item = await q.get()
        if item is None:
            q.task_done()
            break
        print(f"Consuming {item}")
        await asyncio.sleep(0.5)
        q.task_done()

async def main():
    q = asyncio.Queue()
    await asyncio.gather(producer(q), consumer(q))

asyncio.run(main())
```

> ☕ Both are the `BlockingQueue` shape — just one for threads, one for coroutines.

## Thread pools

If you don't want to spin up threads manually, use `concurrent.futures.ThreadPoolExecutor`. This is closest to Java's `ExecutorService` and is the practical default for many workloads.

```python
from concurrent.futures import ThreadPoolExecutor
import time

def fetch(name):
    time.sleep(1)
    return f"done: {name}"

with ThreadPoolExecutor(max_workers=3) as ex:
    futures = [ex.submit(fetch, n) for n in ["A", "B", "C"]]
    for fut in futures:
        print(fut.result())
```

**Raw `Thread` vs `ThreadPoolExecutor`:**

- Raw `Thread` — fine when you have a small number of long-lived custom workers and want explicit control.
- `ThreadPoolExecutor` — preferred for "I have a batch of similar tasks to run with bounded concurrency." Cleaner lifecycle, results via futures, exits cleanly on `with` block exit.

For mixed CPU/IO bulk work, `concurrent.futures.ProcessPoolExecutor` has the same shape but uses processes (bypassing the GIL).

## Coroutines

A coroutine is a function that can suspend and resume later. In modern Python, you define one with `async def`:

```python
async def greet():
    return "hello"
```

Calling a coroutine function **does not run the body** — it returns a coroutine object. To actually run it, you `await` it from inside another coroutine, or hand it to an event loop:

```python
import asyncio

async def main():
    result = await greet()
    print(result)

asyncio.run(main())                     # >>> hello
```

`asyncio.run(...)` is the standard top-level entry point — creates the event loop, runs the coroutine, cleans up.

## Async and await

`async def` defines an async function (returns a coroutine when called). `await` pauses the current coroutine until another awaitable completes, yielding control to the event loop to do other work meanwhile.

```python
import asyncio

async def fetch_data():
    await asyncio.sleep(1)              # simulates I/O
    return "done"

async def main():
    result = await fetch_data()
    print(result)

asyncio.run(main())                     # >>> done (after ~1 second)
```

> ⚠️ **Pitfall:** `await` is **not** "make blocking I/O magically non-blocking." It only works correctly when the awaited operation is genuinely non-blocking. Calling `time.sleep(2)` (blocking!) inside an `async def` freezes the entire event loop for two seconds — see [Mixing async and threads](#mixing-async-and-threads) for the fix.

> ☕ **Java parallel:** Roughly similar in intent to chained `CompletableFuture` workflows, but with `async`/`await` built into the language.

## When to use async

Async wins when:
- Tasks spend most of their time **waiting** on I/O.
- You want many lightweight concurrent operations (hundreds, thousands).
- You don't want to dedicate one OS thread per task.
- Your libraries are async-friendly (`aiohttp`, `httpx.AsyncClient`, `asyncpg`, etc.).

Async does **not** help CPU-bound work — the event loop is single-threaded. For CPU-bound, use `multiprocessing` or native vectorized libraries.

## Gather vs TaskGroup

To run multiple coroutines concurrently, there are two patterns:

**`asyncio.gather`** — fire-and-await-all:

```python
import asyncio

async def fetch(name, delay):
    await asyncio.sleep(delay)
    return f"done: {name}"

async def main():
    results = await asyncio.gather(
        fetch("A", 1),
        fetch("B", 2),
        fetch("C", 1),
    )
    print(results)                      # >>> ['done: A', 'done: B', 'done: C']

asyncio.run(main())
```

`gather` returns when all tasks complete. By default, an exception in any one task does **not** cancel the others — you'll get the exception, but the siblings keep running. This is a footgun.

**`asyncio.TaskGroup` (🐍 3.11+)** — structured concurrency:

```python
import asyncio

async def main():
    async with asyncio.TaskGroup() as tg:
        t1 = tg.create_task(fetch("A", 1))
        t2 = tg.create_task(fetch("B", 2))
        t3 = tg.create_task(fetch("C", 1))
    # all tasks have completed (or all were cancelled) by here
    print(t1.result(), t2.result(), t3.result())
```

When a `TaskGroup` task raises, **siblings are cancelled** and the group raises an `ExceptionGroup` containing all the errors. This is the failure model you actually want for most concurrent operations.

> ☕ **Java parallel:** `TaskGroup` is the async analog of `StructuredTaskScope` from Project Loom. Use it as your default for "run these together; if any fails, abort." Use raw `gather` only when you genuinely want fire-and-forget independence.

## Async context managers

The `async with` statement is the async sibling of `with`. The protocol uses `__aenter__` and `__aexit__`, both coroutines:

```python
class AsyncResource:
    async def __aenter__(self):
        print("acquire")
        return self
    async def __aexit__(self, exc_type, exc, tb):
        print("release")

async def main():
    async with AsyncResource() as r:
        ...
```

This is essential for async DB drivers, HTTP clients, and connection pools:

```python
import httpx

async def fetch_user(uid):
    async with httpx.AsyncClient() as client:
        r = await client.get(f"https://api.example.com/users/{uid}")
        return r.json()
```

**`async for`** iterates over async iterables (`__aiter__` / `__anext__`):

```python
async def main():
    async for record in db_stream():
        await process(record)
```

**Async generators** combine `async def` with `yield`:

```python
async def db_stream():
    async with db.connect() as conn:
        async for row in conn.execute("SELECT ..."):
            yield row
```

> ☕ **Java parallel:** "try-with-resources → async with." Same scoped-cleanup guarantee, but for resources you acquire/release via `await`.

## Async file I/O

Standard `open()` is **blocking**. Inside an async function, calling `open(...)` or `f.read()` will stall the event loop until the OS finishes the syscall. For low-volume one-offs that's tolerable; for anything sustained it's not.

Two practical answers:

**1. `aiofiles` (third-party, common):**

```python
import aiofiles

async def read_log(path):
    async with aiofiles.open(path, "r") as f:
        contents = await f.read()
        return contents
```

**2. Offload to a thread with `asyncio.to_thread`:**

```python
import asyncio

async def read_log(path):
    return await asyncio.to_thread(_read_sync, path)

def _read_sync(path):
    with open(path) as f:
        return f.read()
```

Use `aiofiles` if file I/O is a meaningful fraction of your workload. Use `to_thread` for occasional blocking calls inside an otherwise async app.

## Cancellation and timeouts

In Java, `Thread.interrupt()` is a polite request. In asyncio, **cancellation is cooperative** — a coroutine is told to cancel, and the cancellation propagates only at `await` points by raising `asyncio.CancelledError`.

**`asyncio.timeout`** (🐍 3.11+) — wraps a block with a timeout:

```python
import asyncio

async def main():
    try:
        async with asyncio.timeout(2.0):
            await slow_operation()
    except TimeoutError:
        print("timed out")
```

Inside the `async with`, if 2 seconds elapse before completion, the task is cancelled and a `TimeoutError` is raised.

**Manual cancellation** of a task:

```python
import asyncio

async def main():
    task = asyncio.create_task(slow_operation())
    await asyncio.sleep(1)
    task.cancel()
    try:
        await task
    except asyncio.CancelledError:
        print("cancelled")
```

> ⚠️ **Pitfall:** A coroutine with no `await` between long blocks is uncancellable until it reaches the next `await`. Don't write big synchronous chunks inside `async def`. If you need to do work that takes a while, either break it up with `await asyncio.sleep(0)` or offload it with `asyncio.to_thread`.

> ⚠️ **Don't swallow `CancelledError`.** In Python 3.8+ `asyncio.CancelledError` inherits from `BaseException`, so a bare `except Exception:` will NOT catch it (good). But a bare `except:` clause, `except BaseException:`, or libraries that catch broadly *can* — and if you accidentally swallow it, you break `TaskGroup` and `asyncio.timeout` semantics. If you catch it, re-raise.

## Exception groups

`ExceptionGroup` (🐍 3.11+) is how `TaskGroup` reports multiple concurrent failures. You handle it with the `except*` syntax:

```python
import asyncio

async def fail_a():
    raise ValueError("bad A")

async def fail_b():
    raise TypeError("bad B")

async def main():
    try:
        async with asyncio.TaskGroup() as tg:
            tg.create_task(fail_a())
            tg.create_task(fail_b())
    except* ValueError as eg:
        for e in eg.exceptions:
            print("got ValueError:", e)
    except* TypeError as eg:
        for e in eg.exceptions:
            print("got TypeError:", e)
```

`except*` matches each exception in the group separately. Each branch only sees the matching ones; the rest re-raise (possibly grouped again).

> ☕ **Java parallel:** Java traditionally collapses multi-task failures into the first one, dropping the rest. `ExceptionGroup` + `except*` is the model that preserves all of them and lets you handle by type without losing siblings.

You can also raise `ExceptionGroup` yourself when aggregating errors outside of `TaskGroup`:

```python
raise ExceptionGroup("validation failed", [
    ValueError("name too short"),
    ValueError("age out of range"),
])
```

## Contextvars

`threading.local` gives thread-local storage. But coroutines aren't tied to threads — many coroutines share one thread. `contextvars.ContextVar` is the async-safe replacement that also works correctly with threads:

```python
import contextvars

request_id: contextvars.ContextVar[str] = contextvars.ContextVar("request_id")

async def handle_request(rid):
    request_id.set(rid)
    await process()                     # downstream calls see the same rid

async def process():
    print(f"Processing request {request_id.get()}")
```

Each `Task` gets its own copy of the context, so values don't leak between concurrent requests. Each thread also has its own copy.

> ☕ **Java parallel:** "ThreadLocal → contextvars.ContextVar." Use `ContextVar` whenever you'd reach for `ThreadLocal` in Java — and especially in async code, where `threading.local` is wrong. Request IDs, tenant IDs, trace contexts, logging MDC — all `ContextVar`.

## Async-aware logging

Loggers work fine in coroutines — but if you want request-scoped fields (correlation IDs, user IDs, trace IDs) to propagate, store them in a `ContextVar` and inject them via a `logging.Filter`. This is the Python analog of Java's MDC (Mapped Diagnostic Context):

```python
import logging
import contextvars

request_id = contextvars.ContextVar("request_id", default="-")

class RequestIdFilter(logging.Filter):
    def filter(self, record):
        record.request_id = request_id.get()
        return True

handler = logging.StreamHandler()
handler.setFormatter(logging.Formatter(
    "%(asctime)s [%(request_id)s] %(levelname)s %(message)s"
))
handler.addFilter(RequestIdFilter())

logger = logging.getLogger(__name__)
logger.addHandler(handler)
logger.setLevel(logging.INFO)

# in a request handler:
request_id.set("req-42")
logger.info("processing")
# >>> 2026-05-23 ... [req-42] INFO processing
```

The full `logging` configuration story (handlers, formatters, `dictConfig`) lives in [Part 5 § Logging](05_standard_library.md#logging). What this section adds is: in async or threaded code, use `ContextVar` + a `logging.Filter` to thread per-request context through your logs.

## Blocking vs non-blocking I/O

This is the model that decides whether threads or async fits.

**Blocking I/O** — the thread waits on the kernel; the OS can schedule other threads, but this thread is stuck:

```python
import time
print("before")
time.sleep(2)                           # blocks this thread
print("after")
```

If this happens inside a Python thread, other threads keep running. That's why **threads pair naturally with blocking I/O libraries** — old synchronous DB drivers, classic `requests`, file reads.

**Non-blocking I/O** — the program asks the OS to start an operation and yields control immediately; the event loop runs other work until the operation is ready:

```python
import asyncio

async def main():
    print("before")
    await asyncio.sleep(2)              # yields to event loop
    print("after")

asyncio.run(main())
```

While `asyncio.sleep` is pending, other coroutines on the same event loop run. That's the entire async value proposition: **one thread, many concurrent waiting tasks**.

## Mixing async and threads

The two models compose. Two common moves:

**1. From async, offload a blocking call to a thread** with `asyncio.to_thread`:

```python
import asyncio
import time

async def main():
    # time.sleep is blocking; offload it
    await asyncio.to_thread(time.sleep, 2)

asyncio.run(main())
```

Use this when a function you need to call is blocking and there's no async alternative. The blocking call runs on a background thread (default thread pool); your event loop stays responsive.

**2. From async, dispatch to a custom executor** with `loop.run_in_executor`:

```python
import asyncio
from concurrent.futures import ThreadPoolExecutor

executor = ThreadPoolExecutor(max_workers=4)

async def main():
    loop = asyncio.get_running_loop()
    result = await loop.run_in_executor(executor, expensive_blocking_fn, arg)
```

Use a custom executor when you need to bound the concurrency of the blocking calls separately from asyncio's default pool.

> 💡 **Pythonic:** Async is not "faster threads." It's a different concurrency model. Mix them deliberately at the boundaries — blocking calls offloaded to threads from async, async tasks scheduled into a loop from threads (`asyncio.run_coroutine_threadsafe`).

## Reactive programming

If you came from **Project Reactor** (Spring WebFlux), **RxJava**, or any reactive-streams stack, this is the section that tells you where `Flux<T>` went. Short answer: Python doesn't have a Reactor analog in the standard library, and most of the use cases that drive you to Reactor on the JVM are covered in Python by **`asyncio` + async iterators + `asyncio.Queue`** — pieces you've already seen in this part.

**Reactor → Python conceptual map:**

| Reactor concept | Python idiom | Notes |
| :--- | :--- | :--- |
| `Mono<T>` (0–1 value) | `Awaitable[T]` — usually a `Coroutine` returned by `async def` | `await foo()` is your "subscribe + get the single value." For `Mono.empty()`, use `Awaitable[Optional[T]]` (or `Awaitable[T \| None]` on 3.10+). |
| `Flux<T>` (Reactor) / RxJava `Flowable<T>` (the backpressured one) | `AsyncIterator[T]` / `AsyncGenerator[T, None]` | Consume with `async for x in src:`. RxJava `Observable<T>` is **not** backpressured — only `Flowable` is, which is what maps cleanly here. |
| `flux.map(f)` | Small async-generator wrapper: `async def mapped(src): async for x in src: yield f(x)` | No fluent chain — compose by passing one generator into the next |
| `flux.filter(p)` | Same shape, with `if p(x): yield x` | No method chain — just compose generators |
| `flux.concatMap(f)` (sequential flatten) | Nested `async for`: `async for x in src: async for y in f(x): yield y` | This is **sequential** — one inner stream at a time, in order |
| `flux.flatMap(f)` (concurrent merge) | `asyncio.TaskGroup` driving inner generators concurrently and feeding a bounded `asyncio.Queue`; or `aiostream.stream.flatmap(..., task_limit=N)` | The nested-`for` shortcut is **wrong** for `flatMap` — `flatMap` interleaves inner publishers with prefetch/concurrency. The Python equivalent has to spawn tasks. |
| `Mono.zip(a, b, ...)` / `Flux.zip(...)` (combine all) | `asyncio.gather(a, b, ...)` for one-shot fan-in; `aiostream.stream.ziplatest` for stream-of-streams | `gather` raises on first exception unless `return_exceptions=True` |
| `Flux.merge(...)` (interleave as values arrive) | `asyncio.as_completed(...)` for futures; `aiostream.stream.merge(...)` for async iterables | Stdlib has no built-in async-iterable merge |
| `onErrorResume(fn)` / `onErrorReturn(v)` / `retry(n)` / `retryWhen(...)` | `try / except` inside the consumer loop; `tenacity.retry` decorator on the operation; or just `for attempt in tenacity.Retrying(...):` | No fluent error operators — error handling is structured, not chained. See [Cancellation and timeouts](#cancellation-and-timeouts) for `asyncio.timeout` and [Part 6 § HTTP production behavior](06_ecosystem_and_packaging.md#http-production-behavior) for `tenacity` patterns. |
| `subscribeOn(boundedElastic())` (run the source on a thread pool) | `asyncio.to_thread(...)` / `loop.run_in_executor(...)` to push the blocking step off the event loop | See [Mixing async and threads](#mixing-async-and-threads) |
| `publishOn(parallel())` (hop downstream operators to another thread) | **No direct equivalent** — Python has one event loop per thread, so "hop the rest of the pipeline to another thread" isn't a thing. If a single pipeline step is CPU- or block-heavy, wrap *that step* in `asyncio.to_thread` / `run_in_executor`. | Don't try to translate `publishOn` 1:1 — the Python model is "stay on the loop except where you have to leave." |
| Reactive backpressure (`request(n)` signals, prefetch buffers) | Pull-based `async for` is naturally backpressured (one item per `__anext__`); bounded `asyncio.Queue(maxsize=N)` for buffered/decoupled pipelines | No `request(n)` protocol — the consumer's `__anext__` call **is** the demand signal. Note Reactor also lets you tune prefetch and choose `onBackpressureBuffer`/`Drop`/`Latest`; the Python equivalent is choosing your `Queue` policy (size, full-handling) explicitly. |
| Hot vs cold streams | Async generators are **cold** (a fresh stream per consumer); for hot/multicast, run a producer task that fans out into one `asyncio.Queue` per consumer | Stdlib has no built-in multicast. `asyncio.Event` is a 1-bit flag with no payload — useful for "ready yet?" signals, not for delivering values to multiple consumers. |
| Schedulers (`Schedulers.parallel()`, `boundedElastic()`) | The event loop itself (cooperative scheduling of coroutines) + an executor pool for blocking work | One loop per thread; no first-class scheduler abstraction per-operator |
| `flux.window(...)` / `buffer(...)` / `debounce(...)` | Hand-rolled with `asyncio.sleep` + accumulators, or reach for `aiostream` | Stdlib doesn't ship operator combinators |

**A minimal "Flux pipeline" in Python** — three async generators composed by iteration, no operator chain:

```python
import asyncio

async def numbers(n):                           # source: Flux<Integer>
    for i in range(n):
        await asyncio.sleep(0.01)               # simulate I/O
        yield i

async def squared(src):                         # operator: map
    async for x in src:
        yield x * x

async def even_only(src):                       # operator: filter
    async for x in src:
        if x % 2 == 0:
            yield x

async def main():
    pipeline = even_only(squared(numbers(10)))  # compose by nesting
    async for value in pipeline:
        print(value)

asyncio.run(main())
# >>> 0
# >>> 4
# >>> 16
# >>> 36
# >>> 64
```

Each `async for` step both consumes and produces — that's the backpressure. The `numbers` generator only advances when `squared` calls `__anext__`, which only happens when `even_only` calls `__anext__`, which only happens when `main` iterates. There's no buffered demand signal because there's no buffer.

**Libraries exist for operator-chain syntax, but they aren't idiomatic.** If you genuinely need Rx-style operators in Python:

- **[`RxPY`](https://github.com/ReactiveX/RxPY)** — the official ReactiveX port. Closest to RxJava `Observable` syntax. Niche, and **not a Reactive-Streams backpressure equivalent** — RxPY 3.x dropped built-in backpressure support. Reach for it for operator-chain ergonomics, not for `Flowable`/`Flux`-style flow control.
- **[`aiostream`](https://github.com/vxgmichel/aiostream)** — operator combinators (`map`, `filter`, `merge`, `zip`, `chunks`, `timeout`) over `asyncio` streams. The most "Pythonic" of the reactive-style libraries.
- **[`streamz`](https://streamz.readthedocs.io/)** — data-pipeline focused, integrates with pandas/Dask.

Most Python codebases don't reach for these — they compose async generators and `asyncio.Queue` directly.

> ⚠️ **Pitfall:** Don't import `aiostream`/`RxPY` just to recreate the fluent operator-chain mental model. The Python way is to compose by `async for` and small async-generator helpers, not by stringing `.map(...).filter(...).window(...)` together. Reaching for the chain syntax fights the language and isolates your code from idiomatic readers — keep that escape hatch for cases where the operators themselves (sliding windows, debounce, time-based batching) are actually what you need, not the syntax.

> ☕ **Java parallel:** Reactor's value proposition on the JVM is partly *concurrency* (composing async work without callback hell) and partly a *programming model* (declarative, operator-driven, with backpressure semantics). Python solved the first half with `async`/`await` — you get the composition without needing a stream type. The second half (operators, schedulers, multicast) lives in third-party libraries when you need it, but most teams find they don't.

**Cross-refs:** [Part 3 § Generators](03_pythonic_idioms.md#generators) (cold pull-based streams via `yield`), [Part 3 § Iterable vs iterator](03_pythonic_idioms.md#iterable-vs-iterator) (sync version of the same protocol), [Part 7 § RAG-specific integration patterns](07_interoperability.md#rag-specific-integration-patterns) (streaming LLM responses across services).

## Concurrency chooser

| Situation | Best fit | Why |
| :--- | :--- | :--- |
| Many blocking I/O tasks using sync libraries | **threading** | Easy integration with existing blocking APIs |
| Many concurrent I/O tasks with async-friendly libraries | **async** | Scales beyond one-thread-per-task |
| CPU-bound heavy computation | **multiprocessing** | Bypasses the GIL via separate processes |
| Moderate background work that needs shared in-memory state | **threading** | Threads share memory naturally |
| High-concurrency network server | **async** | Event loop handles many waiting tasks |
| Need real multi-core parallelism for Python compute | **multiprocessing** or native lib | The only way around the GIL today (free-threaded 3.13+ aside) |

Quick model:
- **threading** = good for blocking I/O.
- **async** = good for non-blocking I/O.
- **multiprocessing** = good for CPU-bound.

Pick by workload, not by habit.

## Stdlib modules summary

Concurrency in Python is spread across several stdlib modules — there's no single `java.util.concurrent` umbrella:

| Module | Use |
| :--- | :--- |
| `threading` | Threads, locks, events, conditions, semaphores |
| `queue` | Thread-safe queues (`Queue`, `LifoQueue`, `PriorityQueue`) |
| `concurrent.futures` | `ThreadPoolExecutor`, `ProcessPoolExecutor`, `Future` |
| `multiprocessing` | Process-based parallelism, shared memory, pipes |
| `asyncio` | Event loop, coroutines, async primitives |
| `contextvars` | Context-local storage (async-safe `ThreadLocal`) |

For the Java→Python module mapping in lookup form, see [the Concurrency section of the Appendix](99_appendix_java_to_python.md#concurrency).

## Key Takeaways

- Threads for blocking I/O; async for non-blocking I/O; multiprocessing for CPU-bound.
- GIL ≠ thread safety. The GIL is an interpreter detail; race conditions are real and need explicit synchronization.
- `TaskGroup` over raw `gather` when one failure should cancel siblings; `ExceptionGroup` + `except*` to handle them.
- Cancellation is cooperative — design `await` points so cancellation can propagate.
- `contextvars.ContextVar` is the async-safe `ThreadLocal` — use it for request IDs in logging (Python's MDC).
- Don't call blocking code inside `async def` — offload with `asyncio.to_thread`.
