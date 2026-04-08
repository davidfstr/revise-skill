# Em/en dashes in comments and text

**Trigger:** Em dashes (`--`) or en dashes (`-`) used in code comments, docstrings, or documentation to attach a trailing fragment to a sentence.

**Why:** Em/en dashes are a signature AI writing style. They feel literary, are hard to type on a physical keyboard, and signal AI-generated text. They also create run-on constructions that are harder to scan.

**Fix:** Use separate sentences (with a period) or a semicolon for closely related clauses. Avoid any kind of dash to separate trailing sentence fragments.

Before:
```python
# Stale socket file — remove it so the next launch starts fresh
```

After:
```python
# Stale socket file. Remove it so the next launch starts fresh.
```
