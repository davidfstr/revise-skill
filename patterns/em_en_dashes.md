# Em/en dashes in comments and text

**Trigger:** Em dashes (`--`) or en dashes (`-`) used in code comments, docstrings, or documentation to attach a trailing fragment to a sentence.

**Why:** Em/en dashes are a signature AI writing style. They feel literary, are hard to type on a physical keyboard, and signal AI-generated text. They also create run-on constructions that are harder to scan.

**Fix:** Use separate sentences (with a period) or a semicolon for closely related clauses. Avoid any kind of dash to separate trailing sentence fragments.

> **Do not silently replace the dash with a comma.** Em/en dashes nearly always attach an independent clause or a standalone fragment, so a comma in the same slot produces a comma splice or run-on. When a dash won't convert cleanly to `;` or `.`, rewrite the sentence rather than downgrading it to a comma.

Before:
```python
# Stale socket file — remove it so the next launch starts fresh
```

After:
```python
# Stale socket file. Remove it so the next launch starts fresh.
```

Counter-example -- **don't** just swap in a comma:
```python
# Stale socket file, remove it so the next launch starts fresh   # run-on
```
