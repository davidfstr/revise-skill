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

---

**Inverse case -- named expression (walrus) as explaining name:** Sometimes a value within an expression benefits from a name even though the name is never referenced elsewhere. A walrus operator (`:=`) can name the value inline for clarity. Prefix the name with underscore to signal it is intentionally unused:

```python
# The names _bg_dark and _bg_light clarify which literal is which
return (
    (_bg_dark := "#0d1117") if _is_dark_mode()
    else (_bg_light := "#ffffff")
)
```