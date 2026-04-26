# Minimize try-block scope

## Quick trigger

A `try:` block contains statements that aren't themselves failable — most commonly a *success-action* placed immediately after the call that might raise.

```python
try:
    _set_window_appearance(window, appearance)
    result = {"ok": None}      # not failable; doesn't belong in `try:`
except Exception as e:
    result = {"error": str(e)}
```

The principle: **a `try:` block should contain exactly the statements that are intended to be protected by the `except` arms — no more.**

## Why revise

Widening the `try:` beyond the failable statement has two costs:

1. **Hides what's actually failable.** A reader has to figure out which line is the protected one. A tight scope reads as "this single call is the failure point."
2. **Lets unrelated exceptions get caught.** If a later refactor makes one of the trailing statements failable (e.g., a typed-dict constructor that validates), the broad handler will silently catch and report it as if it came from the protected call. Hard to spot, easy to ship.

Python's `try` statement has a built-in slot for "code that should run only when no exception was raised": the `else:` clause. Using it makes the success-only nature of the trailing code structurally explicit.

## Fix

1. Identify the *failable* statement(s) — the calls or accesses that may raise the conditions you're catching.
2. Shrink the `try:` block to contain only those statements.
3. Move trailing success-path code into an `else:` clause attached to the same `try`.

> **The `else:` is mandatory when the `except` arm falls through.** If any current `except` block does NOT exit unconditionally (no `raise` / `return` / `continue` / `break`), then code moved out of the `try:` *must* go inside `else:` — otherwise it would run *after* the exception handler swallowed the error, changing the program's behavior. Moving the `try:` start downward is easy; moving the `try:` end upward usually requires creating an `else:` clause to preserve semantics.

If the `except` arm exits unconditionally (`raise`, `return`, `continue`, `break`), the `else:` clause is optional — code following the `try:` is reachable only on success either way. Choose whichever reads more clearly for the surrounding context; `else:` is still often clearer because it visibly pairs the success-path with the `try:`.

## Example

Before — the success-result assignment is inside `try:`:

```python
elif method == "set_appearance":
    ...
    else:
        try:
            _set_window_appearance(window, appearance)
            result = {"ok": None}
        except Exception as e:
            result = {"error": str(e)}
```

After — `try:` holds only the failable call; `else:` runs only on success:

```python
elif method == "set_appearance":
    ...
    else:
        try:
            _set_window_appearance(window, appearance)
        except Exception as e:
            result = {"error": str(e)}
        else:
            result = {"ok": None}
```

Note that the `except` arm here falls through (it neither raises nor returns), so the `else:` is mandatory — without it, `result = {"ok": None}` placed after the `try` would run even when the exception path executed, overwriting the `{"error": ...}` value.

## When NOT to revise

- **Sequential failable operations under one handler.** If the second statement is also failable and the same handler is correct for both, leaving them together is appropriate — that's exactly what `try:` is for.
- **`except` exits unconditionally and the trailing code is short.** When the `except` arm always raises/returns, a flat `try` followed by trailing code is a valid alternative to an `else:` clause:
  ```python
  try:
      pid = int(pid_path.read_text().strip())
  except FileNotFoundError:
      return     # exits unconditionally; the next line is reached only on success
  do_something_with(pid)
  ```
  Either form is fine here. Don't churn working code purely to introduce `else:`.

---

Related:
- [specific_exceptions.md](specific_exceptions.md) — narrow the `except` *type* alongside narrowing the `try:` *scope*
- [silent_early_return.md](silent_early_return.md) — when an `except` swallows the error silently, that's a separate bug
