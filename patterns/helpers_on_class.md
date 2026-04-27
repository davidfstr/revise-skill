# Class-specific helpers not on the using class

## Quick trigger

A module-level helper (`def _name(...)` or `def name(...)`) whose only caller is a single class defined in the same module.

## Why revise

- **Tighter top-level surface.** One less name at module scope keeps the file's table-of-contents of identifiers focused on what matters.
- **Colocation with caller.** A reader scanning the class doesn't need to jump to a free function elsewhere in the file -- the helper lives where it's used.
- **Accurate scope signal.** Module-level `_name` advertises "private to the module" -- broader than "private to this one class." Hoisting onto the class tightens the scope to match reality.

## When NOT to revise

- The helper is called by **multiple classes or module-level functions** in the same module. Promoting it into one class creates false ownership.
- The helper is a genuine general-purpose utility that happens to have one caller today (e.g., `_strip_frontmatter(text)`). Leave it at module level; it'll likely get a second caller.
- The helper is imported from another module. Module-level is the right scope.

## Fix

- If the helper doesn't use `self`: move onto the class as `@staticmethod`.
- If it reads class state (class constants, other class methods): `@classmethod` or regular instance method.
- Keep the leading underscore to preserve "class-internal" marking.
- Update the call site from `_name(...)` to `self._name(...)`.

## Example

Before -- `_read_gvc_log` is module-level, but `TestClient._call` is its only caller:

```python
class TestClient:
    def _call(self, method: str, **kwargs: Any) -> object:
        ...
        except TimeoutError:
            gvc_log = _read_gvc_log(self._sock_path)
            raise TimeoutError(...)


def _read_gvc_log(sock_path: Path) -> str:
    """Best-effort read of the gvc log for a given test sandbox."""
    log_path = sock_path.parent.parent / "log" / "gvc.log"
    try:
        return log_path.read_text(encoding="utf-8", errors="replace")
    except FileNotFoundError:
        return f"<gvc log file not found: {log_path}>"


class GvcGuiNotDoneStarting(Exception):
    pass
```

After -- hoisted onto the class as a `@staticmethod`. Module top-level now reads as `TestClient` → exceptions → dataclass, with `TestClient`'s diagnostic helper living inside the class that owns it:

```python
class TestClient:
    def _call(self, method: str, **kwargs: Any) -> object:
        ...
        except TimeoutError:
            gvc_log = self._read_gvc_log(self._sock_path)
            raise TimeoutError(...)

    @staticmethod
    def _read_gvc_log(sock_path: Path) -> str:
        """Best-effort read of the gvc log for a given test sandbox."""
        log_path = sock_path.parent.parent / "log" / "gvc.log"
        try:
            return log_path.read_text(encoding="utf-8", errors="replace")
        except FileNotFoundError:
            return f"<gvc log file not found: {log_path}>"


class GvcGuiNotDoneStarting(Exception):
    pass
```
