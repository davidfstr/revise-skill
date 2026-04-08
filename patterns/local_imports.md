# Local imports in non-entrypoint modules

**Trigger:** Project-specific imports placed inside functions instead of at the top of the file, in a module that is not an entrypoint.

**Why:** Standard Python convention is to place all imports at the top of the file. Local imports are harder to scan (you don't know a module's dependencies without reading every function) and can mask circular import issues.

**When NOT to revise:** In files that serve as the first module loaded when Python starts (containing a `main()` function), it is *conventional* to keep project-specific (non-stdlib) imports local to `main()` or similar functions. This minimizes startup time for commands like `--help` that don't need the full module graph.

**Fix:** Move the import to the top of the file, grouped with other imports following the project's import ordering convention.

Before:
```python
def _open_window(title: str, diff_bytes: bytes, api: AppApi) -> None:
    from gvc.diff_parser import parse
    from gvc.renderer import render
    from gvc.window_manager import create_window
    ...
```

After:
```python
from gvc.diff_parser import parse
from gvc.renderer import render
from gvc.window_manager import create_window

...

def _open_window(title: str, diff_bytes: bytes, api: AppApi) -> None:
    ...
```
