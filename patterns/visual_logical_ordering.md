# Definitions/parameters not in visual/logical order

**Trigger:** Function parameters (or field/variable declarations) list data elements in a different order than they appear visually in the UI, or in a different order than the conceptual data model.

**Why:** When code parameters mirror visual or logical order, readers can map between code and UI/data intuitively. Mismatches create unnecessary cognitive friction.

**Fix:** Reorder parameters (or declarations) to match the visual top-to-bottom or logical order. For UI-related code, match the order elements appear on screen.

Before:
```python
def _open_window(raw: bytes, title: str, api, prefs_loader) -> None:
    # title appears above the diff in the UI, but comes second in params
```

After:
```python
def _open_window(title: str, diff_bytes: bytes, api: AppApi, prefs_loader: _PrefsLoader) -> None:
    # title first (top of window), then diff_bytes (window contents)
```

**Note:** This pattern applies to declaration order as well as parameter order. For example, dataclass fields or local variable assignments should follow a logical ordering when one exists.

**Note:** This is not yet the most illustrative example. Look for better examples in future updates to this skill.
