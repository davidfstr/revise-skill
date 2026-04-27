# Failable operation not named `try_X`

**Trigger:** An operation that returns `None` on failure rather than raising an exception, but its name doesn't signal this behavior.

**Why:** The `try_` prefix signals to readers that a `None` return is expected and normal, not an error. Without it, callers might not realize they need to check for `None`, or might confuse the function with a version that raises.

**Fix:** Add the `try_` prefix to the function name.

Before:
```python
def parse(diff_bytes: bytes) -> LargeDiffInfo | None: ...
```

After:
```python
def try_parse(diff_bytes: bytes) -> LargeDiffInfo | None: ...
```
