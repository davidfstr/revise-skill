# Vague or generic names

**Trigger:** A function, variable, class, or file uses a name that describes *how* it works (mechanism) rather than *what* it represents (domain concept). Common AI patterns: `tmp_file`, `data`, `result`, `process_items`, `handle_request`.

**Why:** Vague names force readers to read the implementation to understand purpose. A precise name communicates intent immediately. This is often the simplest, lowest-risk revision available.

**Fix:** Rename to reflect the domain concept. Propagate the rename through all callers and documentation.

Before:
```python
def write_tmp_file(raw: bytes, title: str) -> Path: ...
def read_tmp_file(path: Path) -> tuple[bytes, str]: ...
```

After (minimal rename, without further restructuring):
```python
def write_gui_request_file(raw: bytes, title: str) -> Path: ...
def read_gui_request_file(path: Path) -> tuple[bytes, str]: ...
```
