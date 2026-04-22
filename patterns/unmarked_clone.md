# Unmarked intentional copy → `# clone`

## Quick trigger

A shallow or deep copy of a value is made for correctness — typically to safely iterate while the underlying collection is mutated, or to hand a defensive copy to untrusted code — but the call site has no marker indicating that the copy is load-bearing.

```python
# Why the list(...) wrapping? Looks like noise.
for listener in list(self.listeners):
    listener.fire()   # may cause self.listeners to be mutated
```

## Why revise

Defensive copies look like waste. A reader "cleaning up" the call site might remove `list(...)` to save an allocation — and introduce a `RuntimeError: dictionary changed size during iteration` (or a silent missed iteration) the next time a callback mutates the collection during the loop.

A `# clone` marker states that the copy is intentional and correctness-critical. Removing it without understanding becomes an obvious regression, not a plausible cleanup.

## When NOT to revise

- **Builders and transformations** where the copy is the whole point (`sorted(xs)`, `list(map(f, xs))`). No marker needed — the operation already makes the new collection obvious.

## Fix

Add `# clone` on the line that creates the copy:

```python
for listener in list(self.listeners):  # clone
    listener.fire()
```

```python
def snapshot(self) -> list[Event]:
    return list(self._events)  # clone  (prevent external mutation)
```

## Example

Before — the `list(...)` wrapping is correctness-critical but could read as noise:
```python
for listener in list(self.listeners):
    listener.fire()   # callback may call self.remove_listener(...)
```

After:
```python
for listener in list(self.listeners):  # clone
    listener.fire()
```

---

Related markers (same family, different triggers):
- [`# reinterpret`](unmarked_reassignment.md) — intentional rebinding of an existing variable
- [`# rename`](unmarked_rename.md) — rebinding that intentionally changes the *name* (same value), to match a narrower meaning
- [`# capture`](unmarked_capture.md) — snapshot an ordering-sensitive value at this point
