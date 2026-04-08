# Short CLI flags in subprocess calls

**Trigger:** Short flags like `-M`, `-w`, `-r` in `subprocess.run()` calls, command construction, or bash scripts.

**Why:** Short flags are write-optimized. A reader encountering `-M` has to look up what it means. Long flags like `--find-renames` are self-documenting.

**When NOT to revise:** In user-facing CLI parsing (argparse definitions), where short flags are conventional for frequently-typed options. This pattern targets flags passed to *other programs* in subprocess calls.

**Fix:** Replace with the equivalent long flag. Also propagate the rename through any documentation that references the short flag.

Before:
```python
cmd = ["git", "diff", "-M"] + args
```

After:
```python
cmd = ["git", "diff", "--find-renames"] + args
```
