# Part 6 — Ecosystem & Packaging

Goal: cross the venv/pip cognitive hurdle, then know which third-party libraries dominate which domain. Stdlib-first reflex from [Part 5](05_standard_library.md) still applies — but when you need more, here's the map.

**Prerequisites:** [Part 5 — Standard Library](05_standard_library.md) (stdlib-first reflex). **Next:** [Appendix](99_appendix_java_to_python.md).

---

## Table of Contents

- [Core differences](#core-differences)
- [Venv and packaging](#venv-and-packaging)
- [Pyproject](#pyproject)
- [Env config](#env-config)
- [Settings management](#settings-management)
- [Jupyter and IPython](#jupyter-and-ipython)
- [Numerical and data libs](#numerical-and-data-libs)
- [Visualization](#visualization)
- [HTTP clients](#http-clients)
- [HTTP production behavior](#http-production-behavior)
- [Web frameworks](#web-frameworks)
- [Middleware and DI](#middleware-and-di)
- [Auth and security](#auth-and-security)
- [Database access](#database-access)
- [Pytest](#pytest)
- [Test doubles](#test-doubles)
- [Tooling](#tooling)
- [ML/AI libs](#mlai-libs)
- [Productivity libs](#productivity-libs)
- [Key Takeaways](#key-takeaways)

---

## Core differences

| Java | Python ecosystem |
| :--- | :--- |
| Build tools (Maven, Gradle) | `pip` + `venv` baseline; Poetry / uv for richer workflows |
| Dependency declaration (`pom.xml`, `build.gradle`) | `pyproject.toml` (modern) / `requirements.txt` (classic) |
| Application server | None — your process is the server (Gunicorn/Uvicorn host it) |
| Spring Boot ecosystem | No single equivalent; mix of frameworks + libraries |
| Testing (JUnit, Mockito) | `pytest` + `unittest.mock` / `pytest-mock` |
| Static analysis (Checkstyle, SpotBugs, ErrorProne) | `ruff` + `mypy` + `black` |
| ORM (Hibernate/JPA) | SQLAlchemy 2.x |

```java
List<Integer> nums = List.of(1, 2, 3, 4);
int sum = nums.stream().mapToInt(n -> n * 2).sum();
```

```python
import numpy as np

nums = np.array([1, 2, 3, 4])
print((nums * 2).sum())                  # >>> 20
```

## Venv and packaging

**Rule:** one virtual environment per project. Always. Without `venv`, `pip install` writes to the system Python — packages from one project leak into another. The cure is older than Maven local repos, but the discipline is the same.

**Create and activate:**

```bash
# Create
python -m venv .venv

# Activate (mac/linux)
source .venv/bin/activate

# Activate (windows powershell)
.venv\Scripts\Activate.ps1

# Activate (windows cmd)
.venv\Scripts\activate.bat
```

Your shell prompt will gain a `(.venv)` prefix once active. From there, `python` and `pip` refer to the venv's copy.

**Install dependencies:**

```bash
pip install requests
pip install requests==2.31.0          # exact version
pip install "requests>=2.30,<3"       # constrained
pip install -e .                       # editable: install the current project for development
```

**Freeze and reinstall** (the classic workflow):

```bash
pip freeze > requirements.txt
# Later, in a fresh checkout:
pip install -r requirements.txt
```

> ⚠️ **Pitfall:** `pip freeze` captures everything in the venv, including transitive dependencies. The resulting `requirements.txt` is reproducible but not human-curated. Modern projects keep a hand-written list of direct deps in `pyproject.toml` and let the resolver figure out transitives.

**Higher-level tools** worth knowing:

- **uv** — Rust-based, fast. By 2026 it has grown well past "fast `pip`" into a full project + environment manager (`uv init`, `uv add`, `uv sync`, `uv.lock`, `uv run`) that rivals or replaces Poetry for many teams. The default recommendation for greenfield Python projects.
- **Poetry** — dependency resolution, lockfile, build/publish. Familiar if you've used npm/yarn. Still solid; uv is the more active alternative.
- **pip-tools** — `pip-compile` for resolved lockfiles, `pip-sync` for reproducible installs. Lighter than Poetry/uv if you already like the raw pip workflow.

For a small project, plain `venv` + `pip` + `pyproject.toml` is fine. For new team projects with locked dependencies, start with uv and only move off it if a specific need pushes you elsewhere.

> ☕ **Java parallel:** `venv` ≈ a per-project Maven local repo (but mandatory, not shared). `pip` ≈ Maven dependency download. Poetry/uv ≈ Gradle/Maven for resolution + lockfile + build. `pyproject.toml` ≈ `pom.xml` / `build.gradle`.

## Pyproject

`pyproject.toml` is the modern, standardized project metadata file. It replaces `setup.py` / `setup.cfg` / `requirements.txt` (for direct deps) / tool config scattered across multiple files:

```toml
[project]
name = "myapp"
version = "0.1.0"
description = "A small thing"
requires-python = ">=3.11"
dependencies = [
    "httpx>=0.27",
    "pydantic>=2.7",
]

[project.optional-dependencies]
dev = [
    "pytest>=8",
    "ruff",
    "mypy",
]

[project.scripts]
myapp = "myapp.cli:main"               # creates an executable `myapp` on install

[build-system]
requires = ["hatchling"]
build-backend = "hatchling.build"

[tool.ruff]
line-length = 100

[tool.mypy]
strict = true
```

`pip install -e .` installs the project itself (editable) along with its declared dependencies; `pip install -e ".[dev]"` adds the optional `dev` group.

> ☕ **Java parallel:** `pyproject.toml` ≈ `pom.xml` for a small library: declares deps, declares the build system, plus inline tool config. `project.scripts` ≈ a JAR's main class entry point — gives you a console command on install.

## Env config

For local development, the common pattern is a `.env` file with environment variables, loaded by `python-dotenv`:

```env
# .env
APP_ENV=development
DATABASE_URL=postgresql://localhost/myapp
API_KEY=secret-key
```

```python
import os
from dotenv import load_dotenv

load_dotenv()                            # reads .env into os.environ

app_env = os.getenv("APP_ENV")
db_url = os.getenv("DATABASE_URL")
```

> ⚠️ **Pitfall:** Never commit `.env` to git. Add `.env` to `.gitignore`; commit a `.env.example` template with placeholder values.

This is fine for scripts. For real applications, prefer typed settings — see the next section.

> ☕ **Java parallel:** Similar role to externalized config in Spring Boot. `.env` ≈ `application-dev.properties`. For the typed equivalent of `@ConfigurationProperties`, jump to [Settings management](#settings-management).

## Settings management

Raw `os.environ.get(...)` returns strings — every caller has to remember to parse, validate, default. `pydantic-settings` (Pydantic v2's settings extension) gives you typed, validated config objects:

```python
from pydantic_settings import BaseSettings, SettingsConfigDict
from pydantic import Field

class Settings(BaseSettings):
    model_config = SettingsConfigDict(
        env_file=".env",
        env_file_encoding="utf-8",
        env_prefix="MYAPP_",            # MYAPP_DATABASE_URL → database_url
    )

    database_url: str
    api_key: str
    debug: bool = False
    max_connections: int = Field(default=10, ge=1, le=100)
    cors_origins: list[str] = Field(default_factory=list)   # mutable defaults: factory, never []

settings = Settings()                   # reads env / .env / defaults
print(settings.database_url)
print(settings.max_connections)         # typed int, validated 1..100
```

Precedence (highest first): explicit `Settings(field=...)` → env var → `.env` file → field default.

For testing, override per-test:

```python
def test_with_debug():
    settings = Settings(debug=True)
    ...
```

Secrets management plays nicely — point at a secrets directory and fields are populated from files:

```python
class Settings(BaseSettings):
    model_config = SettingsConfigDict(secrets_dir="/run/secrets")
    db_password: str                    # read from /run/secrets/db_password
```

> ☕ **Java parallel:** `pydantic-settings.BaseSettings` is the direct analog of Spring `@ConfigurationProperties` — typed, validated, sourced from env + files + defaults, with explicit overrides in tests. `Field(ge=1, le=100)` is the JSR-380 validation equivalent.

## Jupyter and IPython

**Jupyter Notebook** — interactive document interleaving code, prose, charts, and outputs. Industry standard for data exploration, prototyping, ML experiments, demos.

**IPython** — enhanced Python REPL with history, auto-complete, magic commands (`%timeit`, `%debug`), and rich object display. Jupyter uses IPython under the hood.

```bash
pip install jupyterlab
jupyter lab                             # opens in browser
```

> ☕ **Java parallel:** No real equivalent in the Java world. Notebooks compress the compile/run loop to milliseconds and keep all output inline. For a Java engineer touching Python for any data-shaped work, this is the productivity gain that justifies switching languages.

## Numerical and data libs

**NumPy** — fast multi-dimensional arrays + vectorized operations. The foundation everything else builds on:

```python
import numpy as np

arr = np.array([1, 2, 3, 4])
print(arr * 2)                          # >>> [2 4 6 8]    (no loop)
print(arr.mean())                       # >>> 2.5
print(arr.reshape(2, 2))                # >>> [[1 2] [3 4]]
```

Vectorized ops are 10-100× faster than Python loops because they call into compiled C code.

**pandas** — DataFrames: spreadsheet/SQL-table abstraction with index-aware operations:

```python
import pandas as pd

df = pd.DataFrame({"name": ["Alice", "Bob"], "age": [25, 30]})
print(df["age"].mean())                 # >>> 27.5
print(df[df.age > 26])                  # filter rows
```

**Polars** — newer DataFrame library: faster, lower memory, columnar execution, expression-based API. Becoming the default for performance-conscious analytics:

```python
import polars as pl

df = pl.DataFrame({"name": ["Alice", "Bob"], "age": [25, 30]})
print(df.select(pl.col("age").mean()))
```

**SciPy** — scientific algorithms built on NumPy: optimization, integration, linear algebra, signal processing, statistics:

```python
from scipy import stats
print(stats.describe([1, 2, 2, 3, 4]))
```

Mental model: **NumPy** = fast arrays. **pandas** = DataFrames (default). **Polars** = faster DataFrames (newer). **SciPy** = scientific algorithms.

> ☕ **Java parallel:** Java's data-engineering equivalents are several layers of custom collections + utility code + libraries (e.g. ND4J, Smile). Python's stack is much more compact for this domain — one of the strongest reasons to reach for Python at all.

## Visualization

- **matplotlib** — foundational static plotting. The lowest common denominator:

  ```python
  import matplotlib.pyplot as plt
  plt.plot([1, 2, 3], [1, 4, 9])
  plt.show()
  ```

- **seaborn** — built on matplotlib with nicer defaults and statistical plot types.

- **plotly** — interactive charts (hover, zoom, pan). Great for notebooks and dashboards:

  ```python
  import plotly.express as px
  fig = px.line(x=[1, 2, 3], y=[1, 4, 9])
  fig.show()
  ```

Mental model: **matplotlib** = static foundation. **seaborn** = nicer statistical defaults. **plotly** = interactive.

## HTTP clients

**`requests`** — the classic synchronous client. Simple, ubiquitous:

```python
import requests

r = requests.get("https://api.example.com/users/1")
r.raise_for_status()                    # raises on 4xx/5xx
print(r.status_code)
print(r.json())
```

**`httpx`** — modern client, sync + async with one API:

```python
import httpx

# Sync
r = httpx.get("https://api.example.com/users/1")

# Async
async def fetch():
    async with httpx.AsyncClient() as client:
        r = await client.get("https://api.example.com/users/1")
        return r.json()
```

For new code that might go async, default to `httpx`. For pure-sync scripts, `requests` is still fine.

> ☕ **Java parallel:** `requests` ≈ `HttpClient` (synchronous). `httpx` ≈ `WebClient` (sync + async). Both are far lighter than typical Java HTTP setups — no DI, no builder, no client config object.

## HTTP production behavior

Default settings on `requests` and `httpx` have one critical foot-gun and several routine wins to know.

**Always set timeouts.** Default `requests.get(...)` has **no timeout** — a hung server will hang your process forever:

```python
import requests

r = requests.get("https://api.example.com/users/1", timeout=5)
r = requests.get(url, timeout=(3, 10))  # (connect, read)
```

`httpx` has a sane default (5s) but you should still set them explicitly to match your SLO.

**Reuse a session** — connection pooling + shared headers. `requests.Session` has no session-wide timeout setting — pass `timeout` on every call:

```python
session = requests.Session()
session.headers["Authorization"] = "Bearer ..."

urls = ["https://api.example.com/a", "https://api.example.com/b"]
for url in urls:
    r = session.get(url, timeout=5)     # always per-call
    r.raise_for_status()
```

For `httpx`, the equivalent is `httpx.Client(...)` / `httpx.AsyncClient(...)` — both reuse the underlying TCP connection across requests to the same host.

**Retries** — `tenacity` is the de-facto retry library:

```python
from tenacity import retry, stop_after_attempt, wait_exponential
import httpx

@retry(stop=stop_after_attempt(3), wait=wait_exponential(min=1, max=10))
def fetch(url):
    r = httpx.get(url, timeout=5)
    r.raise_for_status()
    return r.json()
```

`tenacity` composes: stop conditions, wait strategies, retry-only-on-specific-exceptions, before/after hooks. Use it whenever you talk to a flaky external service.

> ☕ **Java parallel:** `tenacity` ≈ Resilience4j or OkHttp `Interceptor` retry policies. Always-explicit-timeout is the same advice as Java; the difference is just that `requests` has no default, which surprises people.

## Web frameworks

Three options dominate, each with a different scope:

| Framework | Scope | When to pick |
| :--- | :--- | :--- |
| **Flask** | Micro — routing + Jinja templating; bring your own everything else | Small services, prototypes, you want full control of the stack |
| **FastAPI** | API-focused — strong typing, auto-generated OpenAPI docs, async-first | Typed APIs, async services, microservices |
| **Django** | Batteries-included — ORM, admin, auth, sessions, templating | Full apps, content-heavy sites, when batteries beat assembly |

```python
# Flask — minimal
from flask import Flask
app = Flask(__name__)

@app.route("/")
def home():
    return "Hello, world!"
```

```python
# FastAPI — typed, with auto-docs
from fastapi import FastAPI
app = FastAPI()

@app.get("/users/{uid}")
def get_user(uid: int) -> dict:
    return {"uid": uid, "name": "Alice"}
```

> 💡 **Pythonic:** Spring Boot has no 1:1 Python equivalent. **Django** matches Spring Boot's *scope* (batteries-included, opinionated). **Flask** and **FastAPI** are closer to a Spring Web MVC slice — the API/controller layer without the rest of the platform. Pick by project size, not by name familiarity.

## Middleware and DI

Dependency injection in Python is usually **explicit construction**, not a framework container. Pass your dependencies into constructors. For tests, pass fakes. No annotations, no scanner, no XML.

The exception is FastAPI's `Depends` — its dedicated DI primitive for per-request dependencies (DB sessions, current user, settings):

```python
from fastapi import FastAPI, Depends

def get_db():
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()

app = FastAPI()

@app.get("/users/{uid}")
def read_user(uid: int, db = Depends(get_db)):
    return db.get(User, uid)                # SQLAlchemy 2.x primary-key lookup
```

**Middleware** — the per-request hook layer:

```python
# Flask
@app.before_request
def add_request_id():
    g.request_id = generate_id()

@app.after_request
def log_response(response):
    logger.info("status=%s", response.status_code)
    return response
```

```python
# Django — middleware class with __call__
class RequestIdMiddleware:
    def __init__(self, get_response):
        self.get_response = get_response
    def __call__(self, request):
        request.request_id = generate_id()
        return self.get_response(request)
```

```python
# FastAPI — Starlette middleware
from starlette.middleware.base import BaseHTTPMiddleware

class RequestIdMiddleware(BaseHTTPMiddleware):
    async def dispatch(self, request, call_next):
        request.state.request_id = generate_id()
        return await call_next(request)
```

> ☕ **Java parallel:** `@Autowired` (constructor injection) → explicit constructor injection in Python; FastAPI `Depends` is the closest analog to a true DI container. Servlet filter / Spring `HandlerInterceptor` → framework middleware (one per framework). The Python ecosystem deliberately keeps DI lightweight — there's no Spring-like reflection-driven autowiring.

## Auth and security

There is no Python "Spring Security" — it's an assembled stack:

| Concern | Library |
| :--- | :--- |
| JWT encode/decode | `PyJWT` or `python-jose` |
| Password hashing | `passlib` (bcrypt / argon2) or `bcrypt` / `argon2-cffi` |
| OAuth / OIDC client | `Authlib` |
| Session cookies | framework-native (Flask `session`, Django sessions, FastAPI `SessionMiddleware`) |
| CSRF | Django built-in; Flask via `flask-wtf`; FastAPI APIs with token auth typically skip it |
| CORS | framework-native (Flask-CORS, Django middleware, FastAPI CORSMiddleware) |

```python
# JWT example with PyJWT
import jwt
from datetime import datetime, timedelta, timezone

payload = {"sub": "alice", "exp": datetime.now(timezone.utc) + timedelta(hours=1)}
token = jwt.encode(payload, "secret", algorithm="HS256")
decoded = jwt.decode(token, "secret", algorithms=["HS256"])
```

```python
# Password hashing with passlib
from passlib.context import CryptContext
pwd = CryptContext(schemes=["argon2"])

hashed = pwd.hash("my-password")
print(pwd.verify("my-password", hashed))   # >>> True
```

> 💡 **Pythonic:** Pick a stack, don't reinvent. JWT + passlib + framework session are the common building blocks for most auth flows. For OAuth providers, use Authlib rather than rolling your own. For full-app auth with admin panels and user management, Django's batteries are already there. Auth is a high-risk domain — borrow audited libraries, get a security review for anything beyond the basics.

## Database access

**SQLAlchemy 2.x** — the dominant ORM. Two APIs in one library:

- **ORM** — class-based, session-driven, like Hibernate/JPA.
- **Core** — query-builder, closer to JDBC with safety.

```python
# ORM example
from sqlalchemy import create_engine, String
from sqlalchemy.orm import DeclarativeBase, Mapped, mapped_column, Session

class Base(DeclarativeBase): pass

class User(Base):
    __tablename__ = "users"
    id: Mapped[int] = mapped_column(primary_key=True)
    name: Mapped[str] = mapped_column(String(64))

engine = create_engine("postgresql://localhost/myapp")

with Session(engine) as session:
    session.add(User(name="Alice"))
    session.commit()

    user = session.get(User, 1)
    print(user.name)
```

**Sessions** are unit-of-work — opened, used, committed, closed. Use a context manager and you can't leak connections.

**Async SQLAlchemy** — same shape, `AsyncSession`:

```python
from sqlalchemy.ext.asyncio import create_async_engine, AsyncSession

engine = create_async_engine("postgresql+asyncpg://localhost/myapp")

async def get_user(uid):
    async with AsyncSession(engine) as session:
        return await session.get(User, uid)
```

**Async drivers** — `asyncpg` (Postgres, fast), `aiosqlite`, `aiomysql`. Sync drivers (`psycopg`, `pymysql`) are fine when not using `async`.

**Migrations** — **Alembic** is the standard:

```bash
alembic init migrations
alembic revision --autogenerate -m "add users table"
alembic upgrade head
```

Alembic auto-detects schema changes from your SQLAlchemy models and generates migration scripts. Review the generated SQL — autogenerate isn't perfect.

**Connection pooling** — built into SQLAlchemy's `Engine`. `pool_size` and `max_overflow` cap concurrency; default behavior recycles connections.

**Lightweight alternatives** — when you don't want a full ORM:
- `psycopg` (Postgres-only, direct, very fast)
- stdlib `sqlite3` (zero dependencies, great for tools/tests)
- `peewee` (simpler ORM, smaller surface area)

> ☕ **Java parallel:** SQLAlchemy ORM ≈ Hibernate/JPA. SQLAlchemy Core ≈ jOOQ or plain JDBC + builder. Alembic ≈ Flyway / Liquibase (with `autogenerate` more like Liquibase's diff tooling).

## Pytest

`pytest` is the default Python testing framework. Lower ceremony than JUnit:

```python
# test_math.py
def add(a, b):
    return a + b

def test_add():
    assert add(2, 3) == 5

def test_add_negative():
    assert add(-1, 1) == 0
```

Run with `pytest`. Discovery is automatic — any file named `test_*.py` or `*_test.py`, any function named `test_*`.

**Fixtures** — set up state for tests. Cleaner than `@BeforeEach`:

```python
import pytest

@pytest.fixture
def sample_user():
    return {"name": "Alice", "age": 25}

def test_user_name(sample_user):
    assert sample_user["name"] == "Alice"
```

The fixture is matched by parameter name. Add it to any test's signature.

**Fixture scopes** — control lifecycle:

```python
@pytest.fixture(scope="session")        # once per test run
def database():
    db = setup_test_db()
    yield db
    db.teardown()

@pytest.fixture(scope="function")       # default: once per test (most common)
def fresh_session(database):
    s = database.session()
    yield s
    s.rollback()
```

`yield` separates setup from teardown — same pattern as `@contextmanager`. The fixture body before `yield` is setup; after `yield` is cleanup.

**`conftest.py`** — shared fixtures available to all tests in the directory and subdirectories without import:

```python
# conftest.py
@pytest.fixture
def http_client():
    return TestClient(app)
```

**Parametrize** — run one test against many inputs:

```python
@pytest.mark.parametrize("a,b,expected", [
    (1, 2, 3),
    (0, 0, 0),
    (-1, 1, 0),
])
def test_add(a, b, expected):
    assert add(a, b) == expected
```

Generates three separate test cases — pytest reports them individually.

> ☕ **Java parallel:** `pytest` ≈ JUnit 5. Fixtures ≈ `@BeforeEach` / `@BeforeAll` but **per parameter**, more composable. `@pytest.mark.parametrize` ≈ `@ParameterizedTest`. `conftest.py` ≈ shared test base classes, but file-scoped instead of inheritance-based.

## Test doubles

The Java instinct is Mockito. The Python answer is `unittest.mock` (stdlib) and `pytest-mock` (the pytest fixture wrapper around it).

**Stdlib `unittest.mock`:**

```python
from unittest.mock import MagicMock, patch

mock_db = MagicMock()
mock_db.query.return_value = [{"id": 1}]
print(mock_db.query("SELECT *"))         # >>> [{'id': 1}]

# patch — replace a name during the test
@patch("myapp.module.requests.get")
def test_fetch(mock_get):
    mock_get.return_value.json.return_value = {"ok": True}
    result = my_function()
    assert result == {"ok": True}
```

**`pytest-mock`** — wraps `unittest.mock` in a fixture (`mocker`) with automatic cleanup:

```python
def test_fetch(mocker):
    mock_get = mocker.patch("myapp.module.requests.get")
    mock_get.return_value.json.return_value = {"ok": True}
    assert my_function() == {"ok": True}
```

Prefer `pytest-mock` in pytest projects — no `with` blocks or decorators stacking.

**`AsyncMock`** for coroutines:

```python
from unittest.mock import AsyncMock

mock_fetch = AsyncMock(return_value={"data": 1})
result = await mock_fetch()              # works inside an async test
assert result == {"data": 1}
```

**`monkeypatch`** — pytest's built-in for env vars and attribute patching (lighter than full mocks):

```python
def test_with_env(monkeypatch):
    monkeypatch.setenv("API_KEY", "test-key")
    assert os.environ["API_KEY"] == "test-key"
```

> ⚠️ **Pitfall:** Over-mocking is a real anti-pattern. If your test mocks every collaborator, it's testing the mocks, not the code. Mock at **I/O boundaries** (the HTTP call, the DB session, the filesystem). For internal logic, prefer passing real objects or simple fakes (a dict for "config", a list for "events captured").

> 💡 **Pythonic:** When the code under test accepts its dependencies (constructor injection / function arguments), pass a **fake** instead of `patch()`. Patches are for code you can't restructure.

> ☕ **Java parallel:** `MagicMock` ≈ Mockito `mock()`; `.return_value` ≈ `when().thenReturn()`. `AsyncMock` ≈ Mockito for reactive types. The big philosophical difference: Python testing prefers passing fakes over patching globals — same advice you'd give a junior Mockito user.

## Tooling

The daily-driver pair (was a trio in older docs; `ruff` has subsumed the formatter role):

- **`ruff`** — extremely fast linter + formatter (Rust-based). Replaces `flake8` + plugins for linting and `black` for formatting in a single tool. Run on save in your IDE; run in CI.
- **`mypy`** — static type checker for [Part 3 type hints](03_pythonic_idioms.md#type-hints). Run in CI. Alternatives worth knowing: **`pyright`** (Microsoft, used by Pylance in VS Code) and **`basedpyright`** (community fork with stricter defaults). Pick one and stick with it.

```bash
ruff check . --fix
ruff format .          # black-compatible output; no separate `black` needed
mypy myapp/            # or: pyright myapp/
```

Configure all of them in `pyproject.toml` (see [Pyproject](#pyproject) earlier).

> ☕ **Java parallel:** `ruff` ≈ Checkstyle + SpotBugs + ErrorProne + google-java-format combined (and far faster); `mypy`/`pyright` ≈ compile-time type checking, but separate from execution. The pair is roughly the Python equivalent of a strict Java toolchain.

## ML/AI libs

This is the area where Python's ecosystem advantage is overwhelming.

- **scikit-learn** — classical ML: regression, classification, clustering, preprocessing, model evaluation. Clean, well-documented, batteries-included.
- **PyTorch** — deep learning, research-friendly, dominant in research and production at most non-Google shops.
- **TensorFlow** — deep learning, production-oriented, Google's framework.
- **Hugging Face Transformers** — LLMs and pre-trained models, pipelines, fine-tuning.

For a Java developer who needs to touch ML, this is one of the strongest reasons to learn Python — the alternatives in Java (DJL, DL4J) exist but are much smaller communities.

## Productivity libs

A handful of third-party libraries that pay back their install cost on day one:

- **`rich`** — beautiful terminal output: tables, syntax-highlighted tracebacks, progress bars, markdown rendering. `pip install rich`, then `from rich import print` for immediate gains.
- **`typer`** — CLI framework built on type hints. Lighter than `argparse` for richer CLIs; `click`-based.
- **`pydantic`** — runtime validation + serialization. The single most-used data-modeling library in modern Python. v2 is fast (Rust core).
- **`pydantic-settings`** — typed settings, see [Settings management](#settings-management) above.
- **`structlog`** — structured logging with processor pipelines, contextvars integration.
- **`sqlalchemy`** — ORM + query builder, see [Database access](#database-access).
- **`alembic`** — database migrations.
- **`tenacity`** — retry decorator, see [HTTP production behavior](#http-production-behavior).
- **`aiofiles`** — async file I/O, see [Part 4 § Async file I/O](04_concurrency.md#async-file-io).
- **`openpyxl`** — read/write Excel files.

### `pydantic` in more depth

Pydantic v2 is what you reach for whenever data crosses a trust boundary — incoming JSON from HTTP, config from env, rows from a DB, anything you want to validate before using.

```python
from pydantic import BaseModel, Field, field_validator

class User(BaseModel):
    name: str = Field(min_length=1, max_length=64)
    age: int = Field(ge=0, le=150)
    email: str

    @field_validator("email")
    @classmethod
    def email_must_have_at(cls, v: str) -> str:
        if "@" not in v:
            raise ValueError("not a valid email")
        return v

# Parse + validate
u = User.model_validate({"name": "Alice", "age": 25, "email": "alice@example.com"})

# Serialize back
print(u.model_dump())                    # >>> {'name': 'Alice', 'age': 25, ...}
print(u.model_dump_json())               # JSON string
```

**Aliases** (snake_case Python ↔ camelCase JSON):

```python
class User(BaseModel):
    user_name: str = Field(alias="userName")
    model_config = {"populate_by_name": True}
```

**Strict mode** — refuse implicit type coercion:

```python
class User(BaseModel):
    age: int
    model_config = {"strict": True}
# User(age="25")  → ValidationError (string not coerced to int)
```

> ☕ **Java parallel:** Pydantic is **Jackson (object mapping) + Bean Validation (JSR-380)** combined. `BaseModel` ≈ a Jackson-mapped DTO with validation annotations. `model_validate` ≈ `objectMapper.readValue(...)` plus running validators.

Type hints (Part 3) declare structure; pydantic enforces it at runtime. Use both together — they cover the static and runtime sides of the same concern.

## Key Takeaways

- One venv per project, always.
- `pytest` + `ruff` + `mypy` (or `pyright`) is the daily-driver toolchain; `ruff format` covers `black`'s old role.
- `pydantic` for runtime validation — type hints alone don't validate.
- `pydantic-settings` is the typed-config equivalent of Spring `@ConfigurationProperties`.
- Pick web framework by project size: Flask/FastAPI = micro / API; Django = batteries-included (Spring Boot scope).
- No Spring Security analog — assemble PyJWT + passlib + (framework session) + Authlib for OAuth.
- HTTP clients without explicit `timeout=` are a production foot-gun; use `tenacity` for retries.
- SQLAlchemy 2.x + Alembic is the Hibernate/JPA + Flyway equivalent.
