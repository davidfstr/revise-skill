# Data clump passed through free functions

**Trigger:** Two or more standalone functions that accept or return the same bundle of parameters (e.g., `raw: bytes, title: str`).

**How to recognize:** A repeated parameter group appears across multiple function signatures. Supporting signals include tightly-coupled function pairs (read/write, serialize/deserialize), but the parameter bundle is the primary trigger.

**When NOT to revise:** When the functions are loosely related and share only one parameter. A single shared parameter is not a data clump.

**Fix:** Introduce a `@dataclass` that holds the shared parameters and convert the free functions into methods on that class.

Before:
```python
def write_gui_request_file(raw: bytes, title: str) -> Path:
    meta = json.dumps({"title": title})
    ...

def read_gui_request_file(path: Path) -> tuple[bytes, str]:
    ...
    return raw, meta.get("title", "gvc")
```

After:
```python
@dataclass(frozen=True)
class GuiRequest:
    title: str
    diff_bytes: bytes

    def write_to(self, filepath: Path) -> None: ...
    def write_to_temp_file(self) -> Path: ...

    @staticmethod
    def read_from(filepath: Path) -> GuiRequest: ...
```
