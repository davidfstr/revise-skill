# Duplicate code

## Quick trigger

Two or more code blocks that do substantially the same thing. Either trigger is sufficient:

1. **Size:** The duplicated block is more than 3 lines.
2. **Distance:** The duplicates are far apart in the file (or in different files), making it hard to notice they exist and keep them in sync.

## Why revise

Duplicate code that must stay in sync will eventually diverge. When one copy is updated but not the other, you get subtle inconsistencies (different defaults, different edge-case handling) that are hard to diagnose because each copy looks correct in isolation.

## When NOT to revise

- When the duplication is 1-3 lines, close together, and unlikely to diverge. Inlining a tiny helper adds indirection without meaningful benefit.
- When the "duplicates" are coincidentally similar but serve different purposes and would likely evolve independently. Extracting them would create a false coupling.
- When the extraction would require passing many parameters, making the helper harder to understand than the duplicated code.

## Necessary duplication across language boundaries

Sometimes duplication is unavoidable because the copies live in different languages (e.g., a Python constant and a CSS property). In that case, **all duplicates MUST be marked with a comment** identifying every location:

- Format: `NOTE: Duplicated by X and Y`
- The comment text must be **identical** at every location (ignoring language-specific comment syntax).

```python
# NOTE: Duplicated by _BASE_FONT_SIZE and .cr-brand-header__logotext--text font-size
_BASE_FONT_SIZE = 23
# NOTE: Duplicated by _LIGHT_TEXT_COLOR and .cr-brand-header__logotext--text color
_LIGHT_TEXT_COLOR = (0, 0, 0)
```

```css
    /* NOTE: Duplicated by _BASE_FONT_SIZE and .cr-brand-header__logotext--text font-size */
    font-size: 23px;
    /* NOTE: Duplicated by _LIGHT_TEXT_COLOR and .cr-brand-header__logotext--text color */
    color: #000;
```

## Fix

1. Extract the duplicated logic into a function.
2. Name the function for what it computes/returns, not for where it's called from.
3. Replace both call sites.
4. Check whether the copies had already diverged -- if so, decide which behavior is correct and use that in the extracted function.

## Example

Before -- duplicate header logic, already diverged (different default icons):

```python
def _render_file(fd: FileDiff, idx: int) -> str:
    icon, label = _STATUS_ICONS.get(fd.status, ("⬜", fd.status.capitalize()))
    if fd.status == "renamed":
        path_display = _e(f"{fd.old_path} → {fd.new_path}")
    else:
        path_display = _e(fd.new_path or fd.old_path)
    ...

def _render_outline(file_diffs: list[FileDiff]) -> str:
    for idx, fd in enumerate(file_diffs):
        icon, label = _STATUS_ICONS.get(fd.status, ("📄", fd.status.capitalize()))
        if fd.status == "renamed":
            name = _e(f"{fd.old_path} → {fd.new_path}")
        else:
            name = _e(fd.new_path or fd.old_path)
        ...
```

After -- extracted, with consistent default:

```python
_UNKNOWN_STATUS_ICON = "❓"

def _diff_header(fd: FileDiff) -> tuple[str, str, str]:
    status_icon, status_label = _STATUS_ICONS.get(
        fd.status,
        (_UNKNOWN_STATUS_ICON, fd.status.capitalize()))
    if fd.status == "renamed":
        file_path = f"{fd.old_path} → {fd.new_path}"
    else:
        file_path = fd.new_path or fd.old_path
    return status_icon, status_label, file_path

def _render_file(fd: FileDiff, idx: int) -> str:
    status_icon, status_label, file_path = _diff_header(fd)
    ...

def _render_outline(file_diffs: list[FileDiff]) -> str:
    for idx, fd in enumerate(file_diffs):
        status_icon, status_label, file_path = _diff_header(fd)
        ...
```
