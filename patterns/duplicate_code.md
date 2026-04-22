# Duplicate code

## Quick trigger

Two or more code blocks that do substantially the same thing. Either trigger is sufficient:

1. **Size:** The duplicated block is more than 3 lines.
2. **Distance:** The duplicates are far apart in the file (or in different files), making it hard to notice they exist and keep them in sync.

## Why revise

Duplicate code that must stay in sync will eventually diverge. When one copy is updated but not the other, you get subtle inconsistencies (different defaults, different edge-case handling) that are hard to diagnose because each copy looks correct in isolation.

## When NOT to revise

- When the duplication is 1-3 lines, close together, and unlikely to diverge. Inlining a tiny helper adds indirection without meaningful benefit.
- When the "duplicates" are coincidentally similar but serve different purposes and would likely evolve independently. Extracting them would create a false coupling.
- When the extraction would require passing many parameters, making the helper harder to understand than the duplicated code.

## Necessary duplication: mark every copy with `NOTE: Duplicated in ...`

Sometimes duplication is unavoidable. Common reasons:

- **Language boundaries:** a Python TypedDict/dataclass and its TypeScript equivalent; a Python constant and a CSS value.
- **Build vs. runtime:** a value set in a build-system spec (PyInstaller `.spec`, Dockerfile, Vite config) that is also needed at runtime.
- **Process boundaries:** logic that must run on the backend and the frontend (e.g., rate-limit enforcement, form validation).
- **Parallel structures within a file:** two near-identical functions whose signatures differ enough that extracting a helper would be more confusing than the duplication.

In all of these, mark each copy with a `NOTE:` comment identifying the other copies so a maintainer who touches one can find the rest. Omitting the marker is the real bug: a future reader changes one copy without knowing the others exist.

### Format

The default form is **bidirectional: `NOTE: Duplicated in X and Y`**. Place it at every copy. This is what you should reach for in almost every case.

A rare **directional variant, `NOTE: Duplicates [structure of] X`**, is used only when one side genuinely cannot carry a matching marker:

- The "source" is third-party code you cannot modify -- e.g., a Django class whose logic your application duplicates, or a framework constant you're hand-mirroring.
- The "source" is a framework value being hand-expanded in a context where referencing it directly isn't possible (e.g., a SASS variable hand-expanded inside a Vue `<script>` `linear-gradient()` call).

Placed only at the derivative; the un-markable source is referenced by name.

### What `X` and `Y` should be

Any identifier that lets a maintainer grep their way to the other copy. Pick whichever is most findable:

- **File basenames** when the duplicate is a whole block or type: `teacher_home/types.ts and lesson_list.py`, `protocol.py and protocol.d.ts`.
- **Qualified symbol names (with parens for functions)** when the duplicate is a specific function or method: `UserCodeAssignment.owners() and UserAssignment.can_edit_after_open()`, `proxy_http() and enforceOrRecord()`.
- **Conceptual locations** when there's no single file or symbol: `"app_local" and "app" images`, `backend proxy and rate_limiting.js`.

Use the comment syntax of the surrounding language (`#`, `//`, `/* */`).

**The comment text must be identical at every copy** (ignoring language-specific comment syntax) so that a maintainer who finds one marker can grep the exact string and discover the rest. Pick one form of `X` and `Y` -- e.g. `protocol.py and protocol.d.ts` -- and use that same wording at every site, rather than mirroring the names into each language's local conventions.

### Examples

Cross-language type definition (bidirectional, file-based):

```python
# NOTE: Duplicated in protocol.py and protocol.d.ts
class Problem(TypedDict):
    ...
```

```ts
// NOTE: Duplicated in protocol.py and protocol.d.ts
interface Problem {
    ...
}
```

Cross-process logic (bidirectional, symbol-based):

```python
# NOTE: Duplicated in proxy_http() and enforceOrRecord().
if _ENABLE_RATE_LIMIT:
    ...
```

Derivative copy of a framework value (directional):

```ts
// NOTE: Duplicates structure of linear-gradient() from $ts-gradient
background: linear-gradient(...)
```

Build-vs.-runtime duplication (bidirectional, symbol + file):

```python
# NOTE: Duplicated in gvc.spec and _configure_app_identity()
info_plist={
    "CFBundleName": "gvc",
    ...
}
```

Cross-module workaround that can't be cleanly extracted (different error semantics at each site):

```python
# src/gvc/ipc.py
def receive(conn: socket.socket) -> Path:
    # Force the conn into Python's timeout mode so recv() goes through
    # poll()/select() before the syscall. ...
    # NOTE: Duplicated in _handle_request (testmode.py) and receive (ipc.py)
    conn.settimeout(3.0)
    ...
```

```python
# src/gvc/testmode.py
def _handle_request(conn: socket.socket, api: AppApi) -> None:
    ...
    # Force the conn into Python's timeout mode so recv() goes through
    # poll()/select() before the syscall. ...
    # NOTE: Duplicated in _handle_request (testmode.py) and receive (ipc.py)
    conn.settimeout(3.0)
    ...
```

### Bracket the duplicated block with blank lines

When the duplicated region is a sub-section of a larger function, bracket it with blank lines so the *extent* of the duplicated zone is visually obvious. Without the brackets, a reader updating one copy can't tell where "must stay in sync" ends and function-local logic begins.

```python
def _handle_request(conn: socket.socket, api: AppApi) -> None:
    started_at = time.monotonic()  # capture
    with closing(conn):
        # ... multi-line comment explaining the workaround ...
        # NOTE: Duplicated in _handle_request (testmode.py) and receive (ipc.py)
        conn.settimeout(3.0)
                                                 # <-- blank line: end of duplicated zone
        chunks: list[bytes] = []                 # local to this function
        try:
            while chunk := conn.recv(4096):
                chunks.append(chunk)
        ...
```

## Fix

1. Extract the duplicated logic into a function.
2. Name the function for what it computes/returns, not for where it's called from.
3. Replace both call sites.
4. Check whether the copies had already diverged -- if so, ask/decide which behavior is correct and use that in the extracted function.

## Example

Before -- duplicate header logic, already diverged (different default icons):

```python
def _render_file(fd: FileDiff, idx: int) -> str:
    icon, label = _STATUS_ICONS.get(fd.status, ("⬜", fd.status.capitalize()))
    if fd.status == "renamed":
        path_display = _e(f"{fd.old_path} → {fd.new_path}")
    else:
        path_display = _e(fd.new_path or fd.old_path)
    ...

def _render_outline(file_diffs: list[FileDiff]) -> str:
    for idx, fd in enumerate(file_diffs):
        icon, label = _STATUS_ICONS.get(fd.status, ("📄", fd.status.capitalize()))
        if fd.status == "renamed":
            name = _e(f"{fd.old_path} → {fd.new_path}")
        else:
            name = _e(fd.new_path or fd.old_path)
        ...
```

After -- extracted, with consistent default:

```python
_UNKNOWN_STATUS_ICON = "❓"

def _diff_header(fd: FileDiff) -> tuple[str, str, str]:
    status_icon, status_label = _STATUS_ICONS.get(
        fd.status,
        (_UNKNOWN_STATUS_ICON, fd.status.capitalize()))
    if fd.status == "renamed":
        file_path = f"{fd.old_path} → {fd.new_path}"
    else:
        file_path = fd.new_path or fd.old_path
    return status_icon, status_label, file_path

def _render_file(fd: FileDiff, idx: int) -> str:
    status_icon, status_label, file_path = _diff_header(fd)
    ...

def _render_outline(file_diffs: list[FileDiff]) -> str:
    for idx, fd in enumerate(file_diffs):
        status_icon, status_label, file_path = _diff_header(fd)
        ...
```
