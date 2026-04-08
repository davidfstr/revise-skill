# Magic numbers

**Trigger:** Numeric literals used directly in logic (e.g., `max(8, min(32, size))`), where the meaning of the number isn't obvious from context.

**Why:** Magic numbers force readers to guess what the value represents. A named constant communicates intent immediately.

**Fix:** Extract to a named constant. Scope the constant to its smallest relevant context: class-level constant if only one class uses it, module-level if used across functions.

Before:
```python
def set_font_size(self, size: int) -> None:
    size = max(8, min(32, int(size)))
```

After:
```python
class AppApi:
    _MIN_FONT_SIZE = 8
    _MAX_FONT_SIZE = 32
    ...
    def set_font_size(self, size: int) -> None:
        size = max(self._MIN_FONT_SIZE, min(self._MAX_FONT_SIZE, int(size)))
```
