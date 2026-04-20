# `close()` for lifecycle teardown (not `cleanup`/`teardown`/`dispose`)

## Quick trigger

A class or module-level helper that releases acquired resources is named `cleanup()`, `teardown()`, `dispose()`, `destroy()`, `shutdown()`, or similar — when the standard Python name is `close()`.

Often appears paired with `__init__` or an `acquire`/`open`/`start` method whose counterpart is implicitly "undo this," named arbitrarily.

## Why revise

`close()` is the standard Python lifecycle verb:

- The stdlib uses it everywhere: `file.close()`, `socket.close()`, `connection.close()`, `Pool.close()`, `TemporaryDirectory.cleanup()` (the rare exception).
- `contextlib.closing()` exists specifically to turn any object-with-`close()` into a context manager. Picking a different name forfeits that interop.
- Readers scanning a class for "how do I release this?" look for `close()` first. Synonyms force a read of the class to find the teardown method.

Names like `cleanup`, `teardown`, `dispose` are noise in a language that has already standardized on `close`.

## When NOT to revise

- When the ecosystem has its own convention (e.g., pytest fixtures' `teardown_method`, Django's `tearDown`) — follow local custom.
- When `close()` is already taken by a different operation on the same class (rare, but possible when wrapping a network protocol).

## Fix

Rename to `close()`. Update all callers. If the class held resources acquired in `__init__`, also consider adding `__enter__`/`__exit__` so it can be used as a context manager (`with Foo() as f: ...`).

## Example

Before:
```python
class GvcSandbox:
    def __init__(self) -> None:
        self.root = Path(tempfile.mkdtemp(...))
        ...

    def cleanup(self) -> None:
        shutil.rmtree(self.root, ignore_errors=True)


class GvcApp:
    def teardown(self) -> None:
        ...
```

After:
```python
class GvcSandbox:
    def __init__(self) -> None:
        self.root = Path(tempfile.mkdtemp(...))
        ...

    def close(self) -> None:
        shutil.rmtree(self.root, ignore_errors=True)


class GvcApp:
    def close(self) -> None:
        ...
```

A callable returned from a `start(...)` function is the same principle — name it `close`, not `cleanup`:

```python
def start(api: AppApi) -> Callable[[], None]:
    ...
    def close() -> None:
        sock_path.unlink(missing_ok=True)
        pid_path.unlink(missing_ok=True)
    return close
```
