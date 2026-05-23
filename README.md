# Python Conventions Skills

> A single `CLAUDE.md` for modern Python (3.11+) conventions. Type discipline, pathlib over os.path, async patterns, structlog, ruff + mypy strict + pytest defaults. Drop into any Python project.

## What it covers

| Layer | Rule |
|---|---|
| Types | Hints required everywhere; Pydantic v2 for I/O; TypedDict for internal; Literal for string enums |
| Stdlib | pathlib over os.path, dataclasses over manual `__init__`, subprocess.run over os.system, f-strings except in logging |
| Async | async def end-to-end, httpx not requests, TaskGroup for structured concurrency |
| Logging | structlog or stdlib with structured extras; no print() in libraries; lazy formatting |
| Tooling | ruff (lint + format), mypy --strict, pytest fixtures, uv over pip |
| Anti-patterns | Bare except, mutable defaults, stringly-typed contracts, eval on user input |

Full content: [`CLAUDE.md`](CLAUDE.md). Code-level before/after: [`EXAMPLES.md`](EXAMPLES.md).

## Install

### As a project CLAUDE.md

```bash
curl -o CLAUDE.md https://raw.githubusercontent.com/HermeticOrmus/python-conventions-skills/main/CLAUDE.md
```

### As a Claude Code skill

The same content as an installable skill: [`skills/python-conventions/`](skills/python-conventions/).

### In Cursor

See [`CURSOR.md`](CURSOR.md). Rule at [`.cursor/rules/python-conventions.mdc`](.cursor/rules/python-conventions.mdc).

## Why this exists

Modern Python (3.11+) has cleaner idioms than older Python: native union syntax (`A | B`), `match` statements, `TaskGroup`, `tomllib`, faster startup. AI-generated Python often defaults to older idioms (`Union[A, B]`, `os.path`, `time.sleep` in async code) because the training corpus is heavy on legacy code.

This file overrides the defaults with modern conventions.

## What's intentionally opinionated

- **`uv` over `pip`**. `uv` is 10-100× faster for resolution, has built-in lockfiles, and matches the workflow that most new projects are converging on.
- **`ruff` over `black + flake8 + isort`**. One tool, faster, fewer config files, same outputs.
- **`mypy --strict`**. Some teams prefer `pyright`; the rules here apply to either. Strict mode is the point — `# type: ignore` should be rare.
- **Pydantic v2 for I/O**. v1 → v2 has breaking changes; this file presumes v2. For v1 codebases, the patterns still apply with v1 syntax.

If your project disagrees with any of these defaults, override at the project level. The CLAUDE.md is a baseline, not a religion.

## See also

- [`typescript-conventions-skills`](https://github.com/HermeticOrmus/typescript-conventions-skills) — sister repo for TypeScript
- [`shell-safety-skills`](https://github.com/HermeticOrmus/shell-safety-skills) — companion for bash scripts
- [`andrej-karpathy-skills`](https://github.com/HermeticOrmus/andrej-karpathy-skills) — general coding discipline
- [PEP 8](https://peps.python.org/pep-0008/) — the canonical style guide
- [Hypermodern Python](https://cjolowicz.github.io/posts/hypermodern-python-01-setup/) — longer-form modern Python setup

## Contributing

PRs welcome — especially Django-specific patterns, FastAPI patterns, data-science patterns (numpy/pandas/polars), and ML-specific patterns (PyTorch, JAX).

## License

MIT.
