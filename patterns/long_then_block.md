# Long then-block with missing/minimal else-block

## Quick trigger

A long block of code is indented under `if <match>:` when the surrounding scope (function, loop, branch) has nothing meaningful after it. Three common shapes:

**1. Long body, then end of function/loop:**

```python
def setup_handler():
    if main_menu is not None:
        ... 15 indented lines ...
```

**2. Long body, with a short else that exits:**

```python
def setup_handler():
    if main_menu is not None:
        ... 15 indented lines ...
    else:
        raise RuntimeError("main menu missing")
```

**3. Long body inside a loop, with `break`/`continue`:**

```python
for i in range(menu.numberOfItems()):
    item = menu.itemAtIndex_(i)
    if str(item.title()) == "Edit":
        ... 15 indented lines ...
        break
else:
    print("Edit menu not found.", file=sys.stderr)
```

## Why revise

A long block indented under `if` pushes the eye to the right and obscures the structure of the body. The non-matching path is the small, exceptional case — invert the condition so the matching block sits at the surrounding scope's primary indentation. The reader then scans the real work without mentally subtracting an indent level.

## When NOT to revise

- **Short bodies (1–3 lines).** Indentation isn't a burden; the inversion adds noise.
- **Both branches do meaningful work.** When the alternative branch isn't a short exit but has its own real logic, the two cases are peers — see [symmetric_branches.md](symmetric_branches.md).
- **The match condition is itself complex.** Even after inverting, `if not (a and b and not c): continue` is hard to read. Factor the original condition into a named local (`is_target = a and b and not c`) before applying the pattern.

## Fix

Invert the condition; the long body un-indents. Use whichever exit fits the surrounding scope:

| Surrounding scope | Exit |
|-------------------|------|
| Function | `return` (or `raise` for the no-match-is-bug case) |
| Loop | `continue` |
| Inside an `else` whose else-branch already returns | `return` from the outer function |

In Python, a search loop's `for/else` survives the inversion — the `else` runs exactly when the loop completed without `break`.

A blank line below the guard (separating it from the body) makes the structure even more scannable.

### Negation style: prefer direct over De Morgan's

When inverting a compound condition, prefer the direct form `not (<original>)` over a De Morgan-rewrite that flips operators:

```python
# Original:  if a is not None and str(a.title()) == "Edit":

# Preferred — direct negation:
if not (a is not None and str(a.title()) == "Edit"):
    continue

# Avoid by default — De Morgan rewrite:
if a is None or str(a.title()) != "Edit":
    continue
```

The direct form preserves the original predicate verbatim, so the reader can compare it against the original at a glance. The rewritten form forces the reader to verify a small algebra step (every operand inverted, `and`↔`or` swapped) and is easy to get subtly wrong during edits.

Use the rewritten form only when it produces a noticeably more readable predicate — e.g., when one of the negations simplifies (`not (x is None)` → `x is not None`) or when one operand is itself trivially negative.

## Example (function, end-of-body)

Before:

```python
def install_handler() -> None:
    main_menu = AppKit.NSApplication.sharedApplication().mainMenu()
    if main_menu is not None:
        edit_menu_item = _find_edit_menu_item(main_menu)
        edit_menu_item.submenu().addItem_(AppKit.NSMenuItem.separatorItem())

        find_item = edit_menu_item.submenu().addItemWithTitle_action_keyEquivalent_(
            "Find…", "openFind:", "f"
        )
        find_item.setTarget_(handler)
        ...
```

After:

```python
def install_handler() -> None:
    main_menu = AppKit.NSApplication.sharedApplication().mainMenu()
    if main_menu is None:
        return

    edit_menu_item = _find_edit_menu_item(main_menu)
    edit_menu_item.submenu().addItem_(AppKit.NSMenuItem.separatorItem())

    find_item = edit_menu_item.submenu().addItemWithTitle_action_keyEquivalent_(
        "Find…", "openFind:", "f"
    )
    find_item.setTarget_(handler)
    ...
```

## Example (loop with no-match log)

Before — body indented under `if`:

```python
for i in range(main_menu.numberOfItems()):
    item = main_menu.itemAtIndex_(i)
    submenu = item.submenu()
    if submenu is not None and str(submenu.title()) == "Edit":
        submenu.addItem_(AppKit.NSMenuItem.separatorItem())

        find_item = submenu.addItemWithTitle_action_keyEquivalent_(
            "Find…", "openFind:", "f"
        )
        find_item.setTarget_(handler)

        find_next = submenu.addItemWithTitle_action_keyEquivalent_(
            "Find Next", "findNext:", "g"
        )
        find_next.setTarget_(handler)
        ...
        break
else:
    print("Edit menu not found.", file=sys.stderr)
```

After — body at the loop's primary indentation; `for/else` still fires on no-match:

```python
for i in range(main_menu.numberOfItems()):
    item = main_menu.itemAtIndex_(i)
    submenu = item.submenu()
    if not (submenu is not None and str(submenu.title()) == "Edit"):
        continue

    submenu.addItem_(AppKit.NSMenuItem.separatorItem())

    find_item = submenu.addItemWithTitle_action_keyEquivalent_(
        "Find…", "openFind:", "f"
    )
    find_item.setTarget_(handler)

    find_next = submenu.addItemWithTitle_action_keyEquivalent_(
        "Find Next", "findNext:", "g"
    )
    find_next.setTarget_(handler)
    ...

    break
else:
    print("Edit menu not found.", file=sys.stderr)
```

---

Related:
- [symmetric_branches.md](symmetric_branches.md) — inverse: don't use a guard clause when both branches are peer alternatives
- [silent_early_return.md](silent_early_return.md) — prefer raise/log over silently exiting on no-match
