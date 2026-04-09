# Dead code

## Quick trigger

Code that is not referenced anywhere: unused constants, unreachable branches, unused enum/literal members, commented-out code blocks, unused imports.

## Why revise

Dead code misleads readers into thinking it matters. They may spend time understanding it or trying to keep it "compatible" with other code, not knowing that it is dead. Removing dead code reduces noise and makes the living code clearer.

## When NOT to revise

- When the code is clearly a placeholder for a planned feature (and ideally has a TODO comment saying so).
- When removing it would require changes across a PR/commit boundary you don't control.
- When you're not confident it's truly unused -- some code is invoked dynamically (e.g., via `getattr`, template rendering, or cross-language calls). If uncertain, grep the full project before removing.

## Fix

1. Verify the code is truly unused (grep for references, check for dynamic invocation).
2. Delete it. Do not comment it out, do not leave a `# removed: ...` marker. Git history preserves it if needed.

## Example

Before -- unused regex constants and unused literal member:

```python
_OLD_FILE = re.compile(r"^--- (?:a/)?(.+)$")   # never referenced
_NEW_FILE = re.compile(r"^\+\+\+ (?:b/)?(.+)$") # never referenced

LineKind = Literal["context", "added", "removed", "hunk", "noeol"]
#                                                  ^^^^^ never used
```

After:

```python
LineKind = Literal["context", "added", "removed", "noeol"]
```
