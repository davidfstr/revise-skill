# Errors must not pass silently (raise, or log-and-return)

## Quick trigger

A function `return`s early on a failure condition without raising, logging, or otherwise signalling that something unexpected happened. A reader reaching this code sees the early exit but has no way, in the field, to know it fired.

```python
app_menu_item = main_menu.itemAtIndex_(0)
if app_menu_item is None:
    return   # silent — field debugging is blind
app_menu = app_menu_item.submenu()
if app_menu is None:
    return   # silent — same problem
```

Also triggers on the complementary case: a search loop that silently completes without finding a match.

```python
for i in range(menu.numberOfItems()):
    item = menu.itemAtIndex_(i)
    if item.action() == "orderFrontStandardAboutPanel:":
        item.setTarget_(handler)
        break
# if nothing matched, nothing happens and nobody knows
```

## Why revise

Silent failure is the single most expensive class of bug to diagnose in production. A feature "just stops working": no traceback, no log line, no clue. Every silent `return` on an unexpected-but-survivable condition is a future debugging session that will have no starting point.

The default should be to **raise**: the caller gets a real exception it can log, retry, or surface. Use log-and-return only when the caller genuinely has no better option to offer (e.g., this is a best-effort hook invoked during startup; the app must keep running whether or not it succeeds).

## When NOT to revise

- **The caller will clearly handle it.** Returning `None` or `False` from a function whose contract is "returns `None` on miss" is fine; the caller's branch handles the message. (See also [`try_X` naming](try_x_naming.md) for making that contract visible.)
- **Expected-and-meaningless misses.** `dict.get(key)` returning `None`, `re.match` returning `None` — these aren't failures, they're the normal API shape. No log needed.
- **Already-logged-at-a-lower-layer.** If the call site is a thin re-dispatch and the callee has already logged the real cause, a second log at the return point is just noise.

## Fix

**Prefer `raise`.** Define or reuse a specific exception that names the condition. See [catch_specific_exceptions.md](catch_specific_exceptions.md) for how to pick the right type.

**When log-and-return is the right call**, write a message that gives a field operator something to work with. If no specific convention is already used by the project, use the format `print(..., file=sys.stderr)` with a warning-level message shaped like:

> **`[subsystem] what happened. impact. remediation.`**

- `[subsystem]` — optional (usually omit); a bracketed prefix (e.g. `[gvc testmode]`) used when the source of the error wouldn't otherwise be obvious from context. Grep the codebase for existing examples before inventing a new prefix.
- *what happened* — the observable fact, not the internal variable name. "Main menu not found" — not "itemAtIndex_(0) returned None".
- *impact* — what the user/caller loses. "Cannot define About Panel."
- *remediation* — optional but upgrades a good message to a great one. "Set `GVC_DEBUG=1` for more detail." / "Reopen the window."

Minimum is *what + impact*; anything less is still silent.

For search loops with no match, Python's `for/else` is the idiomatic slot for the no-match log — the `else` runs exactly when the loop completed without `break`.

## Example

Before — two silent returns and a silent no-match loop:
```python
def _define_about_panel() -> None:
    ...
    app = AppKit.NSApplication.sharedApplication()
    main_menu = app.mainMenu()
    if main_menu is None:
        return
    app_menu_item = main_menu.itemAtIndex_(0)
    if app_menu_item is None:
        return
    app_menu = app_menu_item.submenu()
    if app_menu is None:
        return
    for i in range(app_menu.numberOfItems()):
        item = app_menu.itemAtIndex_(i)
        if item.action() == "orderFrontStandardAboutPanel:":
            item.setTarget_(handler)
            item.setAction_("showAboutPanel:")
            break
```

After — each unexpected-miss gets a *what + impact* log; the loop gains a `for/else` for the no-match case:
```python
def _define_about_panel() -> None:
    ...
    app = AppKit.NSApplication.sharedApplication()
    main_menu = app.mainMenu()
    if main_menu is None:
        print("Main menu not found. Cannot define About Panel.", file=sys.stderr)
        return
    app_menu_item = main_menu.itemAtIndex_(0)
    if app_menu_item is None:
        print("App menu item not found. Cannot define About Panel.", file=sys.stderr)
        return
    app_menu = app_menu_item.submenu()
    if app_menu is None:
        print("App menu not found. Cannot define About Panel.", file=sys.stderr)
        return
    for i in range(app_menu.numberOfItems()):
        item = app_menu.itemAtIndex_(i)
        if item.action() == "orderFrontStandardAboutPanel:":
            item.setTarget_(handler)
            item.setAction_("showAboutPanel:")
            break
    else:
        print("About menuitem not found. Cannot define About Panel.", file=sys.stderr)
        return
```

Log-and-return (not raise) is the right call here because the caller is a one-shot `webview.start(func=...)` hook during startup — there is no reasonable recovery path to pass the exception to. A best-effort menu patch that logs its own failures is the best available behavior.

---

Related:
- [catch_specific_exceptions.md](catch_specific_exceptions.md) — when raising, use a named type per failure mode
- [`try_X` naming](try_x_naming.md) — if the function's contract *is* "may miss," put `try_` in the name instead
