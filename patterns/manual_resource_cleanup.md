# Manual resource cleanup

**Trigger:** A resource (file, socket, connection, etc.) is opened/created in one place and cleaned up with an explicit `close()`, `unlink()`, or similar call later -- often at the end of a function or after a `try/except`.

**Why:** Manual cleanup steps are easy to skip on exception paths, leading to resource leaks. AI-drafted code frequently uses this pattern because it's straightforward to write linearly.

**Fix (in priority order):**
1. **Use the resource as a context manager** if it supports the `__enter__`/`__exit__` protocol (e.g., `with open(...) as f:`).
2. **Use `contextlib.closing()`** if the resource has a `close()` method but doesn't support the context manager protocol (e.g., `with closing(socket.socket(...)) as sock:`).
3. **Use `try/finally`** for cleanup that doesn't fit a context manager (e.g., `sock_path.unlink()`).

**When NOT to revise:** When the resource's creation and cleanup happen in different functions or different contexts (e.g., a resource created in one method and cleaned up in a callback). Structured cleanup only works when creation and cleanup are in the same scope.

Before:
```python
server_sock = socket.socket(socket.AF_UNIX, socket.SOCK_STREAM)
server_sock.bind(str(sock_path))
server_sock.listen(backlog=5)
...
webview.start()
server_sock.close()
sock_path.unlink(missing_ok=True)
```

After:
```python
with closing(socket.socket(socket.AF_UNIX, socket.SOCK_STREAM)) as server_sock:
    server_sock.bind(str(sock_path))
    try:
        server_sock.listen(backlog=5)
        ...
        webview.start()
    finally:
        sock_path.unlink(missing_ok=True)
```
