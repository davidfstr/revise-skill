# Bare `dict` at serialization boundaries

## Quick trigger

A function returns `dict` (or accepts `dict`) at a boundary where data crosses between systems -- Python to JavaScript, module to module, or process to process. The shape of the dict is implicit, known only by reading the code that constructs or consumes it.

## Why revise

A bare `dict` return type hides the data contract. Callers must read the implementation to know what keys exist and what types the values have. A `TypedDict` makes the contract explicit and type-checkable: if a key is added or renamed, the type checker catches mismatches.

## When NOT to revise

- When the dict shape is genuinely dynamic (arbitrary user-provided keys, plugin data).
- When the dict is consumed immediately in the same scope and never passed across a boundary.

## Fix

1. Define a `TypedDict` describing the expected shape.
2. Have the data-owning class - if applicable - provide a `to_dict()` method returning the TypedDict.
3. Update the boundary function's return type to use the TypedDict.

## Example

Before -- bare dict constructed in the API layer:

```python
class AppApi:
    def get_prefs(self) -> dict:
        return {
            "font_size": self._prefs.font_size,
        }
```

After -- TypedDict owned by the data class:

```python
# prefs.py
class PrefsDict(TypedDict):
    font_size: int

@dataclass
class Prefs:
    font_size: int = 13

    def to_dict(self) -> PrefsDict:
        return PrefsDict({"font_size": self.font_size})

# app_api.py
class AppApi:
    def get_prefs(self) -> PrefsDict:
        return self._prefs.to_dict()
```
