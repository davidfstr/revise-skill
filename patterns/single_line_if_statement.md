# Single-line if-statement

**Trigger:** An `if` statement places its condition and its body on the *same physical line*, regardless of braces — e.g. `if (e.dataTransfer) e.dataTransfer.dropEffect = "move";` or `if (y < mid) { insertBefore = r; break; }`. The body is something more substantial than a one-token guard.

**Why:**

- **Easy to miss when scanning.** Readers scan the *start* of each line for control flow. Logic packed to the right of the condition reads like commentary; the body slips past the eye.
- **Long-line pressure.** A condition + non-trivial body on one line typically blows past the column limit and needs to be wrapped anyway. Wrapping it idiomatically lands you at the multi-line form.

**Fix:** Move the body to its own line(s).

Before (JavaScript):
```javascript
if (e.dataTransfer) e.dataTransfer.dropEffect = "move";

if (y < mid) { insertBefore = r; break; }
```

After:
```javascript
if (e.dataTransfer) {
    e.dataTransfer.dropEffect = "move";
}

if (y < mid) {
    insertBefore = r;
    break;
}
```

Before (Python):
```python
if e.data_transfer: e.data_transfer.drop_effect = "none"
```

After:
```python
if e.data_transfer:
    e.data_transfer.drop_effect = "none"
```

## When NOT to revise — short guard clauses

A one-token guard whose body exits the surrounding scope (`return`, `break`, `continue`, `throw`) is *idiomatic* on a single line — readers recognize it as a guard at a glance, and the short body doesn't fight line-length:

```javascript
if (rows.length === 0) break;          // OK, idiomatic
if (!_dragRow) return;                 // OK, idiomatic
if (idx === 0) return;                 // OK, idiomatic
```

Wrapping a one-token guard in braces is not idiomatic but also not wrong:

```javascript
if (rows.length === 0) { break; }      // OK, not idiomatic
```

What you should **not** do is omit braces while putting the guard on its own line — that's the unbraced multi-line form covered by [optional_brace_omitted_on_multiline.md](optional_brace_omitted_on_multiline.md):

```javascript
if (rows.length === 0)
    break;                             // NO — see optional-brace pattern
```

## Scope

Applies to any language whose `if` syntax permits a same-line body — JS/TS, Python, C/C++/Java, Go (rarely), Ruby (`x if y` trailing form is a separate idiom). Loops in this single-line form (`for (...) doStuff();`) are rare in practice and the same reasoning applies if they appear.
