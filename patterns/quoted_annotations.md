# Unnecessarily quoted type annotations

## Quick trigger

A type annotation is written as a string literal -- `"Path | None"`, `"list[Foo]"` -- in a project where annotation evaluation is already deferred.

## Why revise

Quoted annotations exist to defer name resolution (forward references, circular imports). When the whole project already defers annotation evaluation, the quotes are dead syntax: they add visual noise and hide the annotation from tools that read `__annotations__` eagerly.

Applies to projects that require **Python 3.14+**, where PEP 649 makes annotation evaluation lazy by default, so every annotation behaves as if it were quoted.

## When NOT to revise

- When the project supports Python versions older than 3.14 and does not put `from __future__ import annotations` at the top of every file.
- When the string is doing something other than deferring evaluation (e.g., a `TypeAlias` value that is genuinely a string at runtime).
- When unquoting would require reordering imports or introducing a cycle -- the quotes are still earning their keep.

## Fix

Remove the surrounding quotes.

## Example

Before:
```python
def _enclosing_app_executable() -> "Path | None":
    ...
```

After:
```python
def _enclosing_app_executable() -> Path | None:
    ...
```
