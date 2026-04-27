# Branch-scoped comment placed above the conditional

## Quick trigger

A comment whose scope is *one branch* of a conditional is placed *above the whole conditional*, forcing the reader to map each sentence back to whichever branch it describes.

Applies to both unprefixed comments and prefixed ones (`# NOTE:`, `# TODO:`, `# FIXME:`, etc.).

```python
def user_runtime_dir() -> Path:
    # Unix socket paths have a 104-char limit on macOS, so use a short flat
    # subdir of the sandbox root rather than the full platformdirs path.
    root = _sandbox_root()
    return root / "runtime" if root is not None else Path(platformdirs.user_runtime_dir(...))
```

The `104-char limit` rationale explains only the `root / "runtime"` branch — but it sits above the whole ternary, where it looks like prose about the function as a whole.

## Why revise

A comment placed above a conditional is read as commentary on the *entire decision* — all branches, plus the condition itself. When the rationale actually applies to only one branch:

- The reader wastes effort reassigning each sentence to the right branch.
- If the branch is later removed or restructured, the comment becomes stale but still visible above the remaining branches, misleading maintainers.
- Side-by-side branches lose their local justification — the reader scanning the *other* branch can't tell whether the comment applies there too.

Placing the comment inside the branch it describes makes the comment's scope unambiguous: it belongs to this branch, and it disappears if the branch disappears.

## When NOT to revise

- **Commentary that genuinely applies to the whole conditional** (rationale for the condition itself, explanation of why this choice is being made at all). Keep it above the `if`/`match`.
- **Very short labels** (`# default:` above the `else` branch) where the placement is conventionally understood.
- **Comments on a guard clause** (`if not x: return`) — those describe what the whole guard does, not one branch.

## Fix

Move the comment inside the branch it describes. For multi-line ternaries, this may require breaking the expression onto multiple lines:

```python
return (
    # rationale for this branch
    root / "runtime" if root is not None
    else Path(platformdirs.user_runtime_dir(...))
)
```

For `if/elif/else` chains, place the comment on the first line inside the relevant branch:

```python
if status == "retry":
    # NOTE: don't re-increment the attempt counter on transient errors
    retry()
elif status == "fail":
    give_up()
```

For `match`:

```python
match event:
    case NetworkError():
        # Handled centrally by the reconnect loop — don't log here
        pass
    case AuthError():
        raise
```

## Example

Before — 104-char reason applies only to the sandbox branch, but sits above the whole ternary:

```python
def user_runtime_dir() -> Path:
    # Unix socket paths have a 104-char limit on macOS, so use a short flat
    # subdir of the sandbox root rather than the full platformdirs path.
    root = _sandbox_root()
    return root / "runtime" if root is not None else Path(platformdirs.user_runtime_dir(_APP_NAME))
```

After — comment is scoped to the branch it justifies:

```python
def user_runtime_dir() -> Path:
    root = _sandbox_root()
    return (
        # Use a short flat subdir of the sandbox root rather than the full platformdirs path
        # because Unix socket paths have a 104-char limit on macOS
        root / "runtime" if root is not None
        else Path(platformdirs.user_runtime_dir(_APP_NAME))
    )
```

Before — `NOTE:` applies to the `kill` branch only, but sits above the whole check:

```python
# NOTE: SIGTERM may race with the process's own exit handler; see #421
if process.is_alive():
    process.terminate()
else:
    log.info("process already exited cleanly")
```

After:

```python
if process.is_alive():
    # NOTE: SIGTERM may race with the process's own exit handler; see #421
    process.terminate()
else:
    log.info("process already exited cleanly")
```
