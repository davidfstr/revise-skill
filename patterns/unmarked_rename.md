# Unmarked value-preserving rename → `# rename`

## Quick trigger

Inside a block, a variable is rebound to the *same value* under a more specific name, because a narrower fact has just become true about it. The rebinding looks like a wasteful assignment or a plain alias if it carries no marker.

```python
for i in range(app_menu.numberOfItems()):
    item = app_menu.itemAtIndex_(i)
    if item.action() == "orderFrontStandardAboutPanel:":
        about_item = item   # looks wasteful; why not just use `item`?
        about_item.setTarget_(handler)
        about_item.setAction_("showAboutPanel:")
        break
```

## Why revise

This is a sibling of `# reinterpret`: both are rebindings, but `# reinterpret` changes the *value* (and keeps the name), whereas `# rename` changes the *name* (and keeps the value). A reader scanning a block with neither marker can't tell what changed.

The value of a `# rename` is readability at the point of use: once narrowing logic (a type check, a predicate, a branch) has proved that `item` is specifically the About menu item, using it as `about_item` below makes the rest of the block self-documenting. In refactoring terms, the rebound name is an **explaining variable** — a name introduced to capture a meaning that the code has just established but that the original variable doesn't convey on its own. The marker says "this isn't waste — it's a deliberate name swap to match the narrower meaning that's now known."

Removing the rename to "clean up" the duplicate assignment throws away that readability gain and re-forces the reader to remember the narrowing at every call site below.

## When NOT to revise

- **Narrowing that the language expresses directly.** In TypeScript/Kotlin, type narrowing inside an `if` branch already changes the variable's apparent type — no rename needed. In Python with `isinstance` guards, mypy narrows in place. Only rename when the narrowing is *semantic* (this `item` is now specifically the About item) rather than type-level.
- **Aliases that last only one line.** `about = items[0]; about.fire()` is fine without a marker — the scope is too small for confusion.
- **Names that already improve** via a destructuring pattern or tuple unpack — the rebinding is inherent to the syntax.

## Fix

Add `# rename` on the line that rebinds to a more specific name.

```python
if item.action() == "orderFrontStandardAboutPanel:":
    about_item = item  # rename
    about_item.setTarget_(handler)
    about_item.setAction_("showAboutPanel:")
```

Do not add further commentary adjacent to `# rename`. The marker flags the intent; explanation (if needed) goes in a separate `# NOTE:` comment.

## Example

Before — once inside the matching branch, `item` continues to be used generically, even though we now know it's specifically the About menu item:
```python
for i in range(app_menu.numberOfItems()):
    item = app_menu.itemAtIndex_(i)
    if item.action() == "orderFrontStandardAboutPanel:":
        item.setTarget_(handler)
        item.setAction_("showAboutPanel:")
        break
```

After — the rebinding gives the narrowed value a name that matches its narrower meaning, and the marker tells readers the duplication is deliberate:
```python
for i in range(app_menu.numberOfItems()):
    item = app_menu.itemAtIndex_(i)
    if item.action() == "orderFrontStandardAboutPanel:":
        about_item = item   # rename
        about_item.setTarget_(handler)
        about_item.setAction_("showAboutPanel:")
        break
```

---

Related markers (same family, different triggers):
- [`# reinterpret`](unmarked_reassignment.md) — intentional rebinding that changes the *value* (same name)
- [`# capture`](unmarked_capture.md) — snapshot an ordering-sensitive value at this point
- [`# clone`](unmarked_clone.md) — defensive copy for correctness
