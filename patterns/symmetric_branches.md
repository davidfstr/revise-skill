# Guard clause followed by peer alternative

**Trigger:** An `if` with an early `return` (or `continue`/`break`) followed by code at the same indentation level that handles the alternative case.

**When to revise:** When both branches represent peer alternatives of equal importance -- neither is an edge case or precondition. The structure should reflect that they are symmetric choices.

**When NOT to revise:** When the `if` truly guards against a precondition or error (e.g., input validation, null check). Guard clauses are appropriate for filtering out edge cases before the "real" logic begins.

**Fix:** Restructure as an explicit `if/else` so both branches have equal visual weight.

Before:
```python
if try_send(sock_path, tmp_path):
    # Existing GUI server accepted the request
    return

# No server running. Launch one.
subprocess.Popen(...)
```

After:
```python
if try_send(sock_path, tmp_path):
    # Existing GUI server accepted the request
    pass
else:
    # No GUI server running. Launch one.
    subprocess.Popen(...)
```

---

Related:
- [long_then_block.md](long_then_block.md) — inverse: when one branch *is* a precondition filter, do use a guard clause
