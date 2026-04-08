# `# type: ignore` and `cast(...)` in typed code

**Trigger:** Any use of `# type: ignore` or `cast(...)` in Python code.

**Why:** These are type-safety escape hatches that suppress real errors. AI-drafted code reaches for them quickly rather than fixing the underlying type issue. They can mask bugs that the type checker would otherwise catch.

**Fix:**
1. Temporarily remove the `# type: ignore` or `cast(...)` to see the original typechecker error it was suppressing.
2. Use `/fix-typecheck-errors` for strategies to fix the underlying type issue (e.g., `isinstance()` narrowing, assertions, protocol types, restructuring).
