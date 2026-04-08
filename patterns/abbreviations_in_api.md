# Abbreviations in API surfaces

**Trigger:** Abbreviated names in function parameters, class fields, or other API-like surfaces (e.g., `ld_info`, `fd`, `cfg`).

**Why:** These names appear in call sites, documentation, and IDE tooltips where readers lack surrounding context to decode them. Abbreviations force readers to guess or look up what the abbreviation expands to.

**When NOT to revise:** Abbreviations are more acceptable in local variables where scope is small and context is immediate. Well-established abbreviations like `i`, `idx` in loop variables are fine.

**Fix:** Spell out the full name in parameters, fields, and other API surfaces.

Before:
```python
def render(fd: list[FileDiff], ld_info: LargeDiffInfo | None = None): ...
```

After:
```python
def render(file_diffs: list[FileDiff], large_diff_info: LargeDiffInfo | None = None): ...
```
