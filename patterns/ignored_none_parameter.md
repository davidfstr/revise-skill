# Ignored None parameter

## Quick trigger

A function accepts `T | None` (or `Optional[T]`) and starts by checking `if x is None: return`. The function silently does nothing when given `None`, even though its name promises an action.

```python
def _run_js_in_window_titled(api: AppApi, title: str | None, js: str) -> None:
    if title is None:
        return   # silently refuses to do what the name says
    for w in api.open_windows():
        if w.title == title:
            w.evaluate_js(js)
            return
```

## Why revise

A function whose name promises an action but silently no-ops is an anti-pattern. Callers can't tell from the call site whether the action ran or was skipped — debugging requires reading the function body. The contract should reflect what's actually true: the function only operates on a non-`None` value, and **deciding what to do when no value is available is the caller's problem, not the function's**.

This is a Type Design fix: narrow the contract to what the implementation actually requires. The `None` check moves *outward* to each call site, where the surrounding context can decide whether to fall back, raise, log, or skip — instead of every caller getting the same hardcoded "do nothing" behavior baked into the helper.

## When NOT to revise

- **The function name signals optionality.** `try_X`, `apply_if_set`, `clear_if_present` — these are explicit "may no-op" contracts. See [`try_X` naming](try_x_naming.md).
- **The function is part of a fluent/chainable API** where `None`-passthrough is the documented behavior (e.g., `Optional.map(f)` in Java, `?.` in JS).
- **The function returns a meaningful value on `None`** (e.g., `len(x or "")`) — that's not a silent no-op, it's a documented identity.

## Fix

1. Remove the `if x is None: return` block from the function.
2. Tighten the parameter type from `T | None` to `T`.
3. At each call site, decide explicitly what to do when the value would have been `None`. Common choices:
   - **Skip the call entirely**: wrap in `if x is not None:` at the call site.
   - **Provide a default**: `f(x if x is not None else default)` or `f(x) if x is not None else default`.
   - **Raise**: if `None` here would be a bug, let the type system catch it earlier.

For the related search-loop case where the function silently completes without finding a match, prefer `raise` over silent return — see [silent_early_return.md](silent_early_return.md).

## Example

Before — helper silently refuses on `None`; callers can't see this from the call site:

```python
def _run_js_in_window_titled(api: AppApi, title: str | None, js: str) -> None:
    """Executes js in the window with the given title."""
    if title is None:
        return
    for w in api.open_windows():
        if w.title == title:
            w.evaluate_js(js)
            return
    # also: silently completes when no window matches

# Callers — no visible decision about None:
title = _key_window_title()
threading.Thread(
    target=_run_js_in_window_titled,
    args=(api, title, "openFindBar()"),
    daemon=True,
).start()
```

After — contract narrowed to non-`None`; each caller chooses; no-match raises:

```python
def _run_js_in_window_titled(api: AppApi, title: str, js: str) -> None:
    """Executes js in the window with the given title."""
    for w in api.open_windows():
        if w.title == title:
            w.evaluate_js(js)
            break
    else:
        raise ValueError(f"No such window: {title}")

# Caller decides: if we don't know which window is active, skip the dispatch.
title = _key_window_title()
if title is not None:
    threading.Thread(
        target=_run_js_in_window_titled,
        args=(api, title, "openFindBar()"),
        daemon=True,
    ).start()
```

The reader of the call site now sees the `if title is not None:` decision in plain view, instead of having to read the helper to discover it.

---

Related:
- [silent_early_return.md](silent_early_return.md) — prefer raise/log over silent return on unexpected conditions
- [`try_X` naming](try_x_naming.md) — when "may no-op" *is* the contract, name the function so callers can see it
- [is_not_none_over_truthy.md](is_not_none_over_truthy.md) — use `is not None` for the caller-side check
