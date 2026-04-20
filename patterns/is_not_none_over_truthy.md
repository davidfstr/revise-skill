# `is not None` over truthy check for Optional values

## Quick trigger

A check on an `Optional[X]` value uses truthiness (`if x:`, `if not x:`, `x if x else default`) instead of an explicit `x is not None` / `x is None` comparison.

```python
def user_runtime_dir() -> Path:
    root = _sandbox_root()                       # Path | None
    return root / "runtime" if root else Path(platformdirs.user_runtime_dir(...))
```

## Why revise

Truthy checks conflate two different questions:

- "Is this value set?" (what you usually mean)
- "Is this value non-empty / non-zero / truthy?" (what Python actually evaluates)

For types with **falsy-but-valid** states, these answers diverge and introduce silent bugs:

- `Path("")` is falsy (empty string path). A `Path("")` sandbox root would wrongly fall through to the default.
- `""` empty string, `0`, `0.0`, empty list/dict/set — all falsy but may be valid values.
- Numeric types where `0` is meaningful (counts, coordinates).
- Custom types that override `__bool__`.

Even when the current code can't produce a falsy-valid value, `is not None` is:

- **Self-documenting.** States exactly what you meant to check.
- **Robust to type widening.** If the field later gains a falsy-valid state, truthy-check code breaks silently; `is not None` continues to work.
- **Caught by typecheckers.** Type narrowing from `x is not None` is precise; truthy narrowing is weaker.

## When NOT to revise

- When the variable is `Iterable` / `Sized` and you actually *do* want "non-empty" (not just "set"). Consider `if items:` over `if items is not None and len(items) > 0:` — be explicit about what you mean.
- In `if x := thing():` walrus patterns where the truthy check is part of the idiom and the value type is unambiguously non-Optional-falsy.

## Fix

Replace `if x:` → `if x is not None:` and `if not x:` → `if x is None:` when `x` is `Optional[...]`.

For ternaries:
```python
# Before
y = x / "runtime" if x else default
# After
y = x / "runtime" if x is not None else default
```

## Example

Before:
```python
def user_runtime_dir() -> Path:
    root = _sandbox_root()
    return root / "runtime" if root else Path(platformdirs.user_runtime_dir(_APP_NAME))

def user_data_dir() -> Path:
    root = _sandbox_root()
    return root / "data" if root else Path(platformdirs.user_data_dir(_APP_NAME))
```

After:
```python
def user_runtime_dir() -> Path:
    root = _sandbox_root()
    return (
        root / "runtime" if root is not None
        else Path(platformdirs.user_runtime_dir(_APP_NAME))
    )

def user_data_dir() -> Path:
    root = _sandbox_root()
    return (
        root / "data" if root is not None
        else Path(platformdirs.user_data_dir(_APP_NAME))
    )
```
