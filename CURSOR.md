# Using this repo with Cursor

This project includes a Cursor project rule with file-scope to Python files.

## In this repository

1. Open the folder in Cursor.
2. The rule [`.cursor/rules/python-conventions.mdc`](.cursor/rules/python-conventions.mdc) is scoped to `**/*.py`, `**/pyproject.toml`, `**/requirements*.txt`.
3. Confirm under **Settings → Rules**.

## Use the same discipline in another project

**Cursor**: Copy `.cursor/rules/python-conventions.mdc` into that project's `.cursor/rules/`.

**Other AI tools**: Copy [`CLAUDE.md`](CLAUDE.md) to the project root.

## Pair with ruff for mechanical enforcement

```bash
pip install ruff
ruff check . --fix && ruff format .
```

Add to CI:

```yaml
- run: ruff check .
- run: ruff format --check .
```
