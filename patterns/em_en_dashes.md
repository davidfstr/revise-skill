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

## When NOT to revise

**Docstring `term -- description` listings.** Python docstrings (rST/Sphinx tradition) conventionally use `--` as the separator between an exception/parameter/return name and its description. This is a long-established technical-writing convention, **not** the AI run-on style — leave it alone.

```python
def select_menuitem(self, shortcut: str) -> None:
    """
    Raises:
    * RuntimeError -- if no matching menu item is found
    * NoSuchWindow -- if a matching window does not exist
    """
```

The trigger only fires for em/en dashes used to attach trailing *sentence fragments* in prose comments or docstring body text. List-item separators in `Raises:` / `Returns:` / `Arguments:` blocks (and similar bulleted name-then-description listings) are exempt.

If a project uses a different convention for these listings (e.g., `: ` or `\n    description`), match the project's existing style — but don't invent the colon when `--` is already in use throughout the file.
