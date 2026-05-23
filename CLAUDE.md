# CLAUDE.md

Python conventions for modern (3.11+) projects. Apply to every `.py`, `pyproject.toml`, and `requirements*.txt`.

**Tradeoff**: bias toward strict-but-modern defaults. For one-off scripts or legacy codebases (3.8-, untyped, old Django), use judgment.

## Type discipline

- **Type hints required** on every function signature, public attribute, and return value
- `from __future__ import annotations` at top of files — gives forward-reference types and PEP 604 union (`A | B`) on older runtimes
- **Pydantic v2** for any external/IO model — never plain `dict[str, Any]` at API boundaries
- `TypedDict` for internal dict shapes that don't need validation
- `Literal["a", "b"]` for string-literal enums (lighter than `enum.Enum`)

## Stdlib preferences

- `pathlib.Path` over `os.path` — composable, type-safe
- `dataclasses` (or Pydantic) over manually-written `__init__` for data containers
- `subprocess.run()` over `os.system` — actual return-code handling
- `f-strings` over `%` or `.format()` — except for logging, where lazy formatting matters
- `match` statements (3.10+) for tagged-union dispatch — cleaner than chained `isinstance`

## Async

- `async def` end-to-end — don't mix sync `requests` into an async stack; use `httpx`
- `asyncio.gather` for parallel; `asyncio.TaskGroup` (3.11+) for structured concurrency with proper cancellation
- Never `time.sleep()` in async code — `await asyncio.sleep()` only

## Logging

- `structlog` if available in the project; else stdlib `logging` with structured extras
- **Never `print()`** in library code — only allowed in CLIs / scripts where stdout is the deliverable
- Lazy formatting: `logger.info("user %s logged in", user_id)` not `logger.info(f"user {user_id}...")` — avoids string interpolation when log level is filtered

## Tooling defaults

- **`ruff`** for lint + format (replaces `black` + `flake8` + `isort`)
- **`mypy --strict`** for type checking when the codebase tolerates it
- **`pytest`** with fixtures over class-based setup/teardown
- **`uv`** over `pip` for new projects (faster, lockfile by default)

## Anti-patterns

- Bare `except:` clauses (catches `KeyboardInterrupt`, `SystemExit` — almost never what you want)
- Mutable default arguments (`def f(x=[]):` — use `None` and assign inside)
- Catching exception just to re-raise (`except Exception: raise` — pointless)
- Stringly-typed return contracts (return enum/Literal/dataclass, not arbitrary strings)
- `eval()` / `exec()` on user input
- `from foo import *`

## Project setup

A new Python project starts with:

```toml
# pyproject.toml
[project]
name = "your-project"
version = "0.1.0"
requires-python = ">=3.11"

[tool.ruff]
target-version = "py311"
line-length = 100

[tool.ruff.lint]
select = ["E", "F", "I", "N", "W", "UP", "B", "C4", "SIM", "RET"]
ignore = ["E501"]  # line length — ruff format handles it

[tool.mypy]
strict = true
python_version = "3.11"

[tool.pytest.ini_options]
testpaths = ["tests"]
```

```bash
# Initialize
uv init
uv add ruff mypy pytest
uv add --dev pytest-cov
```

---

**License**: MIT.
