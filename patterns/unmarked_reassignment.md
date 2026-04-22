# Unmarked reassignment of a variable → `# reinterpret`

## Quick trigger

In a language where variables are non-final by default (Python, most dynamic languages), an existing variable is rebound to a new value without a marker explaining that the reassignment is intentional.

```python
def process(raw: bytes) -> None:
    text = raw.decode("utf-8")
    text = _strip_frontmatter(text)   # intentional? or accidental name reuse?
    text = text.strip()
    ...
```

## Why revise

In languages without `final`/`const` defaults, every reassignment looks like it *might* be an accident — a name collision, a forgotten rename, a copy-paste slip. The reader has to inspect each one to decide. Normal reassignments should be treated as anomalous: treat variables as conceptually final unless something says otherwise.

A `# reinterpret` marker says "yes, this rebinding is intentional; the variable now conceptually means something related but different." It also flags the spot to future maintainers: when modifying or reordering code around a `# reinterpret`, take extra care — the variable's meaning shifts here.

## When NOT to revise

- **Languages with concise final-by-default variables** (JavaScript/TypeScript `const`, Rust `let`, Swift `let`, Kotlin `val`). In these, the *absence* of `const` (i.e., `let`/`var`/`mut`) already signals anomaly. See [prefer_const_over_let.md](prefer_const_over_let.md) for the complementary rule: default to `const`, and let non-`const` itself be the marker.
- Mechanical conversions that don't change meaning (`x = int(x)`) — these are rare edge cases and a marker adds noise.
- Loop iteration variables and accumulators where rebinding is the whole point (`for x in xs:`, `total = total + ...`).

## Fix

Add `# reinterpret` on the line that reassigns.

If a reassignment is *not* intentional or deliberate (you could just use a new name), prefer introducing a fresh variable.

## Example

Before — three reassignments, no markers; the reader must verify each is intentional:
```python
data = raw.read_bytes()
data = _decrypt(data)
data = json.loads(data)
```

After — markers state that each rebinding is deliberate; reordering now requires care:
```python
data = raw.read_bytes()
data = _decrypt(data)  # reinterpret
data = json.loads(data)  # reinterpret
```

---

Related markers (same family, different triggers):
- [`# rename`](unmarked_rename.md) — rebinding that intentionally changes the *name* (same value), to match a narrower meaning
- [`# capture`](unmarked_capture.md) — snapshot an ordering-sensitive value at this point
- [`# clone`](unmarked_clone.md) — defensive copy for correctness
