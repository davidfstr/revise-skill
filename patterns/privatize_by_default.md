# Privatize functions and methods by default

## Quick trigger

A module or class exposes a function/method as public (no leading underscore) when:

- All current callers live in the same module/class.
- There is no express intention to make the function/method publicly used from outside the module/class in the future.

## Why revise

Default-public grows the API surface of a module/class unnecessarily. Each public name is a commitment:

- **Harder to reason about.** Any public method could have external callers; changing its signature or semantics is risky. Private methods can be assumed local — reorder, rename, inline freely.
- **Harder to shrink.** Making a public method private later is a breaking change you may never take. Making a private method public when actually needed is trivial.
- **Obscures the API surface.** When most methods are public, the few that matter don't stand out. A small public API focuses attention on what the module/class is actually *for*.

This aligns with the "deep modules" principle: a good module has a small, clearly-labeled public interface backed by substantial private machinery.

## When NOT to revise

- Functions/methods that *are* called from outside the module/class (verify with a grep).
- Methods that implement a required protocol (`__enter__`, framework hooks like `setUp`, etc.) — these are public by contract.

## Fix

1. Grep for external callers of each currently-public name.
2. For each name with no external callers: add a leading `_` (Python convention).
3. Update internal callers to the new name.

## Example

Before — `call` is public but only ever used by the typed wrappers below:

```python
class TestClient:
    def __init__(self, sock_path: Path) -> None:
        self.sock_path = sock_path

    # === API ===

    def ping(self) -> dict[str, object]:
        result = self.call("ping")
        assert isinstance(result, dict)
        return result

    def list_windows(self) -> list[WindowInfo]:
        result = self.call("list_windows")
        ...

    # === Utility ===

    def call(self, method: str) -> object:
        with closing(socket.socket(...)) as s:
            ...
```

After — `call` → `_call`:

```python
class TestClient:
    def __init__(self, sock_path: Path) -> None:
        self._sock_path = sock_path

    # === API ===

    def ping(self) -> dict[str, object]:
        result = self._call("ping")
        assert isinstance(result, dict)
        return result

    def list_windows(self) -> list[WindowInfo]:
        result = self._call("list_windows")
        ...

    # === Utility ===

    def _call(self, method: str) -> object:
        with closing(socket.socket(...)) as s:
            ...
```

Note `sock_path` → `_sock_path` for the same reason: the attribute is only used internally.
