# Missing `assert_never` on union dispatch

**Trigger:** An `isinstance` chain or `match` statement that dispatches on a union type and exhibits either of these antipatterns:

1. **Last variant handled by unchecked `else`:** The final `else` assumes the remaining type without an explicit `isinstance`/`case` check. New variants silently fall through.
2. **No `else` branch at all:** The chain handles all current variants but has no fallback. New variants silently do nothing.

Both are unsafe: adding a new variant to the union later won't produce a type error at dispatch sites.

**Fix:** Make every variant an explicit `isinstance` check (or `case`), then add a final `else: assert_never(x)` branch. Import `assert_never` from `typing`.

Before (antipattern 1 -- unchecked else):
```python
type Shape = Triangle | Square | Circle

shape: Shape = ...
if isinstance(shape, Triangle):
    ...
elif isinstance(shape, Square):
    ...
else:  # Circle -- unsafe: silently swallows new variants
    ...
```

Before (antipattern 2 -- no else):
```python
if isinstance(shape, Triangle):
    ...
elif isinstance(shape, Square):
    ...
elif isinstance(shape, Circle):
    ...
# no else -- unsafe: new variants silently do nothing
```

After:
```python
if isinstance(shape, Triangle):
    ...
elif isinstance(shape, Square):
    ...
elif isinstance(shape, Circle):
    ...
else:
    assert_never(shape)
```
