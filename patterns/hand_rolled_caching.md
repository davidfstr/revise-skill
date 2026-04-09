# Hand-rolled caching

## Quick trigger

Module-level sentinel variables (`_FOO: str | None = None`) with a guard function (`if _FOO is None: compute; return _FOO`), sometimes accompanied by assert chains to satisfy the type checker after the guard.

## Why revise

Hand-rolled caching adds boilerplate (sentinel declarations, None checks, asserts) that obscures the actual computation. `functools.cache` or `functools.lru_cache` express the same intent in one line and are well-understood by readers.

## When NOT to revise

- When the cache needs explicit invalidation (clearing specific entries, TTL-based expiry). `@cache` has `.cache_clear()` but no per-key control.
- When the cached function has mutable arguments (unhashable types). `@cache` requires hashable arguments.
- When memory is a concern and you need bounded cache size -- use `@lru_cache(maxsize=N)` instead of `@cache`.

## Fix

1. Remove the module-level sentinel variables.
2. Add `@cache` (or `@lru_cache`) to the function that computes the values.
3. Remove the guard check and any assert chains.

## Example

Before -- hand-rolled cache with sentinels and asserts:

```python
_CSS: str | None = None
_JS: str | None = None
_HTML_TEMPLATE: str | None = None

def _assets() -> tuple[str, str, str]:
    global _CSS, _JS, _HTML_TEMPLATE
    if _CSS is None:
        _CSS = _load_asset("diff.css")
        _JS = _load_asset("diff.js")
        _HTML_TEMPLATE = _load_asset("diff.html")
    else:
        assert _CSS is not None
        assert _JS is not None
        assert _HTML_TEMPLATE is not None
    return _CSS, _JS, _HTML_TEMPLATE
```

After -- `@cache`:

```python
from functools import cache

@cache
def _assets() -> tuple[str, str, str]:
    _CSS = _load_asset("diff.css")
    _JS = _load_asset("diff.js")
    _HTML_TEMPLATE = _load_asset("diff.html")
    return _CSS, _JS, _HTML_TEMPLATE
```
