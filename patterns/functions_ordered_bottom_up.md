# Functions ordered bottom-up (callees before callers)

**Trigger:** In a file, helper/callee functions are defined *above* the higher-level functions that call them. The reader encounters implementation details before understanding the big picture.

**Why:** AIs naturally write callees first (they need to know the helper's name to write the caller). But readers want the opposite: start at the high-level entry point, then drill down into details as needed. Files should read top-down like a newspaper article: headline first, details below.

**Fix:** Reorder so that high-level functions appear first, with callees defined below their callers. Calls should go "downward" in the file.

Before:
```python
def _build_title(args: list[str]) -> str:
    ...

def main() -> None:
    ...
    title = _build_title(args)
    ...

if __name__ == "__main__":
    main()
```

After:
```python
def main() -> None:
    ...
    title = _build_title(args)
    ...

def _build_title(args: list[str]) -> str:
    ...

if __name__ == "__main__":
    main()
```

**How to apply:** The best tool depends on what's being reordered.

Use `mcp__revise__move_string_in_file` to efficiently reorder items that have **unique boundary lines**, without requiring a manual copy-and-paste. Works well for:
- **Functions and classes** — `def` and `class` signatures are usually unique
- **`# === Header ===` sections**

Use `Write` (full rewrite) when reordering items with **non-unique boundaries** or when merging sections. In particular use `Write` with:
- **`# ---...\n# Header` sections** -- the delimiter line is identical everywhere; each move leaves orphaned delimiters behind, requiring a cleanup Edit pass

For files under ~300 lines where you already have the content in context a full `Write` is often one operation vs. many moves + edits.

To verify the result, use `mcp__revise__outline_file` — it shows definitions and section headings at all indent levels, similar to VS Code's folded view, without the noise of a full file read.

**Among siblings: match call order.** When a caller invokes multiple helpers, define those helpers in the order they're invoked. The file then reads in execution order.

Before:
```python
def main() -> None:
    ...
    _build_icns()                                # called 1st
    ...
    if args.editable:
        _symlink_editable_app_to_source_tree()   # called 2nd

def _symlink_editable_app_to_source_tree() -> None:   # defined 1st — out of order
    ...

def _build_icns() -> None:                            # defined 2nd
    ...
```

After:
```python
def main() -> None:
    ...
    _build_icns()
    ...
    if args.editable:
        _symlink_editable_app_to_source_tree()

def _build_icns() -> None:                            # matches call order
    ...

def _symlink_editable_app_to_source_tree() -> None:
    ...
```

**Exceptions:** Some definitions must stay at the top or bottom of a file due to technical constraints. For example, `if __name__ == "__main__":` must always remain at the bottom. Decorators must be defined before any function they decorate. Do not reorder past these constraints.
