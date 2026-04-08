# Temporary variable that adds no explanatory value

**Trigger:** A variable is assigned once, used once on the very next line (or nearby), and its name doesn't clarify intent beyond what the expression already communicates.

**Why:** The variable introduces a name that the reader must track mentally without adding information. Inlining removes the indirection.

**When NOT to revise:** When the variable name *explains* an otherwise obscure expression. An "explaining variable" earns its existence by giving a readable name to something that would be hard to understand inline. The test: does the variable name tell you something the expression doesn't?

**Fix:** Inline the expression at its single use site.

Before:
```python
prefs = Prefs.load()
create_window(html_doc, title, prefs, api)
```

After:
```python
create_window(html_doc, title, Prefs.load(), api)
```
