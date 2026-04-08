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

**How to apply:** Use the `mcp__revise__move_string_in_file` tool to efficiently move function/class definitions around without needing to retype them. This is much less error-prone than cut-and-paste via the standard editing tools.

**Exceptions:** Some definitions must stay at the top or bottom of a file due to technical constraints. For example, `if __name__ == "__main__":` must always remain at the bottom. Decorators must be defined before any function they decorate. Do not reorder past these constraints.
