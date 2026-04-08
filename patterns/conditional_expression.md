# if/else assigning same variable -> conditional expression

**Trigger:** An `if/else` block where both branches assign to the same variable and the expressions are short enough to read inline.

**Why:** The if/else form obscures that the block's sole purpose is to produce a single value. A conditional expression makes the intent explicit: "compute this value."

**When NOT to revise:** When the branch expressions are complex, multi-line, or have side effects. If the conditional expression would exceed a comfortable line length or require nested ternaries, keep the if/else.

**Fix:** Convert to a conditional expression.

Before:
```python
if large_diff_info is not None:
    html_doc = render(large_diff_info)
else:
    html_doc = render(parse(diff_bytes))
```

After:
```python
html_doc = (
    render(large_diff_info)
    if large_diff_info is not None
    else render(parse(diff_bytes))
)
```
