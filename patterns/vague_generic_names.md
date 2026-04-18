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

---

**Vague action verbs in function names:** Verbs like `make_X`, `do_X`, `handle_X`, `process_X`, `setup_X` hide what the function actually does. Two problems compound:

1. **Mechanism is hidden** — the reader has to open the function to see how `make_X` works.
2. **Scope is overclaimed** — `make_X` sounds like this function *is* the making-of-X. If the work is actually split across multiple places (e.g. a config flag set in `main()` plus this function's step), the name misleads the reader into thinking they've found the full picture here.

**Fix:** Rename to a concrete verb that describes *what this function does*, not the larger goal it participates in.

Before:
```python
# main() sets GVC_NOARCHIVE=1, the PyInstaller spec reads it and skips
# embedding .pyc files, THEN this function swaps in a symlink. "make_editable"
# claims ownership of the whole process, but is actually just the last step.
def _make_editable() -> None:
    bundle_gvc = _ROOT / "dist/gvc.app/Contents/Resources/gvc"
    shutil.rmtree(bundle_gvc)
    bundle_gvc.symlink_to(_ROOT / "src/gvc")
```

After:
```python
def _symlink_editable_app_to_source_tree() -> None:
    bundle_gvc = _ROOT / "dist/gvc.app/Contents/Resources/gvc"
    shutil.rmtree(bundle_gvc)
    bundle_gvc.symlink_to(_ROOT / "src/gvc")
```

The new name states exactly this function's step. Readers looking for "how does editable mode work?" can now find *each* contributing piece by name rather than assuming one function owns it all. Length is fine — a long, precise private helper name beats a short ambiguous one.
