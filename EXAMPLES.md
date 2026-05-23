# Examples

Before/after for each convention with real Python code.

---

## 1. Type discipline

**Before**:
```python
def get_user(id):
    user = db.query("SELECT * FROM users WHERE id = %s", (id,))
    return user[0] if user else None
```

**After**:
```python
from __future__ import annotations
from typing import Optional

def get_user(id: int) -> Optional[User]:
    user = db.query("SELECT * FROM users WHERE id = %s", (id,))
    return User.from_row(user[0]) if user else None
```

Or with Python 3.10+ union syntax:

```python
def get_user(id: int) -> User | None:
    ...
```

---

## 2. Pydantic v2 for I/O

**Before**:
```python
def create_order(data: dict):
    items = data.get("items", [])
    customer_id = data.get("customer_id")
    # ... validation scattered through code
```

**After**:
```python
from pydantic import BaseModel, Field

class CreateOrderRequest(BaseModel):
    items: list[OrderItem] = Field(min_length=1)
    customer_id: int

def create_order(data: CreateOrderRequest) -> Order:
    # data is validated by the type
    ...
```

---

## 3. pathlib over os.path

**Before**:
```python
import os

config_dir = os.path.expanduser("~/.config/myapp")
os.makedirs(config_dir, exist_ok=True)
config_file = os.path.join(config_dir, "settings.json")
if os.path.exists(config_file):
    with open(config_file) as f:
        config = json.load(f)
```

**After**:
```python
from pathlib import Path

config_file = Path.home() / ".config" / "myapp" / "settings.json"
config_file.parent.mkdir(parents=True, exist_ok=True)
if config_file.exists():
    config = json.loads(config_file.read_text())
```

---

## 4. dataclasses for data containers

**Before**:
```python
class User:
    def __init__(self, id, email, name, created_at):
        self.id = id
        self.email = email
        self.name = name
        self.created_at = created_at

    def __repr__(self):
        return f"User(id={self.id}, email={self.email})"

    def __eq__(self, other):
        return isinstance(other, User) and self.id == other.id
```

**After**:
```python
from dataclasses import dataclass
from datetime import datetime

@dataclass
class User:
    id: int
    email: str
    name: str
    created_at: datetime
```

---

## 5. subprocess.run over os.system

**Before**:
```python
import os
result = os.system(f"git log -1 {file_path}")
# result is the exit code from `man 2 wait`; not useful directly
```

**After**:
```python
import subprocess

result = subprocess.run(
    ["git", "log", "-1", file_path],
    capture_output=True,
    text=True,
    check=True,
)
print(result.stdout)
```

Note: list of args (not shell string). Avoids shell injection.

---

## 6. match for tagged-union dispatch

**Before**:
```python
def process(event):
    if isinstance(event, ClickEvent):
        return handle_click(event.x, event.y)
    elif isinstance(event, KeyEvent):
        return handle_key(event.key)
    elif isinstance(event, ResizeEvent):
        return handle_resize(event.w, event.h)
    else:
        raise ValueError(f"Unknown event: {event}")
```

**After**:
```python
def process(event: Event) -> Result:
    match event:
        case ClickEvent(x=x, y=y):
            return handle_click(x, y)
        case KeyEvent(key=key):
            return handle_key(key)
        case ResizeEvent(w=w, h=h):
            return handle_resize(w, h)
```

---

## 7. async/await end-to-end

**Before** (mixed):
```python
import requests

async def fetch_data():
    response = requests.get(URL)  # BLOCKING in async context
    return response.json()
```

**After**:
```python
import httpx

async def fetch_data() -> dict:
    async with httpx.AsyncClient() as client:
        response = await client.get(URL)
        return response.json()
```

---

## 8. TaskGroup for structured concurrency

**Before**:
```python
async def fetch_all(urls):
    tasks = [fetch_one(url) for url in urls]
    return await asyncio.gather(*tasks)
# If one fails, gather doesn't cancel siblings cleanly
```

**After** (Python 3.11+):
```python
async def fetch_all(urls: list[str]) -> list[dict]:
    results = []
    async with asyncio.TaskGroup() as tg:
        tasks = [tg.create_task(fetch_one(url)) for url in urls]
    return [t.result() for t in tasks]
# TaskGroup cancels siblings on first failure; cleaner shutdown
```

---

## 9. Lazy logging

**Before**:
```python
logger.debug(f"Processing user {user_id} with {len(items)} items")
# String interpolation runs EVEN IF logger.debug is filtered out
```

**After**:
```python
logger.debug("Processing user %s with %d items", user_id, len(items))
# Interpolation only happens if the log level is active
```

---

## 10. No print() in libraries

**Before** (in library code):
```python
def import_data(file_path):
    print(f"Loading {file_path}...")
    data = parse(file_path)
    print(f"Loaded {len(data)} records")
    return data
```

**After**:
```python
import logging
logger = logging.getLogger(__name__)

def import_data(file_path: Path) -> list[Record]:
    logger.info("Loading %s", file_path)
    data = parse(file_path)
    logger.info("Loaded %d records", len(data))
    return data
```

CLI scripts can still use print(); library code shouldn't.

---

## 11. No bare except

**Before**:
```python
try:
    risky()
except:
    log("Failed")
# Catches KeyboardInterrupt, SystemExit — bad
```

**After**:
```python
try:
    risky()
except Exception as e:
    logger.exception("Failed: %s", e)
```

Or for specific recovery:

```python
try:
    risky()
except ValueError as e:
    handle_value_error(e)
except IOError as e:
    handle_io_error(e)
```

---

## 12. No mutable default arguments

**Before**:
```python
def add_item(item, items=[]):
    items.append(item)
    return items
# `items` is shared across all calls — bug factory
```

**After**:
```python
def add_item(item: Item, items: list[Item] | None = None) -> list[Item]:
    if items is None:
        items = []
    items.append(item)
    return items
```

---

## 13. Literal over enum for string-literal sets

**Before**:
```python
from enum import Enum

class Status(Enum):
    PENDING = "pending"
    ACTIVE = "active"
    CLOSED = "closed"

def update(status: Status):
    ...
```

**After** (when you don't need the enum's class behavior):
```python
from typing import Literal

Status = Literal["pending", "active", "closed"]

def update(status: Status) -> None:
    ...
```

Lighter, no runtime cost, type-checker still catches typos.

---

## 14. Stringly-typed return → typed return

**Before**:
```python
def parse_command(s):
    if s.startswith("/"):
        return "command"
    elif s.startswith("@"):
        return "mention"
    else:
        return "text"
# Caller has to remember the exact strings; no help from the type-checker
```

**After**:
```python
from typing import Literal

CommandKind = Literal["command", "mention", "text"]

def parse_command(s: str) -> CommandKind:
    if s.startswith("/"):
        return "command"
    elif s.startswith("@"):
        return "mention"
    return "text"
```

---

## 15. ruff configuration

Standard `pyproject.toml`:

```toml
[tool.ruff]
target-version = "py311"
line-length = 100

[tool.ruff.lint]
select = [
    "E",   # pycodestyle errors
    "F",   # pyflakes
    "I",   # isort (import sorting)
    "N",   # pep8-naming
    "W",   # pycodestyle warnings
    "UP",  # pyupgrade (auto-modernize)
    "B",   # bugbear (likely bugs)
    "C4",  # comprehensions (better list/dict/set comps)
    "SIM", # simplify (anti-pattern detection)
    "RET", # return-statement issues
]
ignore = [
    "E501",  # line length — let ruff format handle it
]

[tool.ruff.format]
quote-style = "double"
indent-style = "space"
```

Run: `ruff check . --fix && ruff format .`

## Standard project bootstrap

```bash
uv init my-project
cd my-project
uv add ruff mypy pytest pydantic httpx structlog
uv add --dev pytest-cov pytest-asyncio
echo 'from __future__ import annotations' > src/my_project/__init__.py
```

Then write code that adheres to this CLAUDE.md.
