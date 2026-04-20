# Unmarked ordering-sensitive assignment → `# capture`

## Quick trigger

An assignment whose correctness depends on *when* the right-hand side is evaluated — typically a snapshot of a changing value that must be held for later comparison — has no marker indicating that ordering is load-bearing.

```python
# Why is this variable introduced? Could it be inlined into the condition?
deadline = time.monotonic() + timeout
while time.monotonic() < deadline:
    ...
```

## Why revise

Without a marker, an ordering-sensitive assignment looks like an ordinary helper variable a reader might "simplify" by inlining or moving. Inlining `time.monotonic() + timeout` into the `while` condition turns a timeout into an infinite loop. Moving the assignment past an intervening state change silently corrupts the snapshot.

A `# capture` marker says "this assignment takes a snapshot *at this specific moment* — do not reorder, inline, or recompute." Maintainers reordering code around it now know to preserve the timing.

## When NOT to revise

- **Pure expressions** (no time-dependent or state-dependent operand) — `x = a + b` can be reordered freely; no marker needed.
- **Idiomatic patterns** where the snapshot is structurally obvious (e.g., `for item in list(items):` — the `list(...)` call is already visually distinct; see [`# clone`](unmarked_clone.md) for that sibling case).
- **Temporal dependency in variable name** already — `old_value`, `original_value`
- **Assignments that are just named-for-clarity helpers** with no ordering sensitivity. Those either are fine as-is or should be inlined per [temporary_variable.md](temporary_variable.md).

## Fix

Add `# capture` on the line that snapshots the ordering-sensitive value.

```python
deadline = time.monotonic() + timeout  # capture
```

Do not add further commentary adjacent to "capture". The marker's purpose is to flag, not to explain. If further explanation is needed, add a separate `# NOTE: ...` comment instead.

## Example

Before — deadline is clearly meant to be captured once, but nothing signals that to the reader or to a future edit:
```python
deadline = time.monotonic() + timeout
while time.monotonic() < deadline:
    ...
```

After:
```python
deadline = time.monotonic() + timeout  # capture
while time.monotonic() < deadline:
    ...
```

Also common: capturing per-request state at a handler's entry, before the request object mutates:
```python
user_id = request.user.id   # capture (request.user may be replaced below)
request.user = _anonymize(request.user)
audit.record(actor=user_id, ...)
```

---

Related markers (same family, different triggers):
- [`# reinterpret`](unmarked_reassignment.md) — intentional rebinding of an existing variable
- [`# clone`](unmarked_clone.md) — defensive copy for correctness
