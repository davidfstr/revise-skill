---
name: revise
description: Perform common code quality revisions on AI-drafted code. Use when reviewing recently drafted code for readability and maintainability improvements.
disable-model-invocation: true
argument-hint: [diff-spec]
---

# Revise

Perform common code quality improvement revisions on code that was recently drafted by AI.

## Why this skill exists

AI-written code is usually drafted in a way that is easy for an AI to *write* at the time but not necessarily easy to *read* later (by human or AI). For example:

- AIs frequently use local imports in Python code -- contrary to standard Python convention, but easy to write in isolation without considering the rest of the file.
- AIs tend to write low-level functions before high-level functions -- easy to write a high-level function when you already know the names of the low-level functions it will call, but less efficient for readers who always want to start reading at high-level entrypoints.

The goal of revising code is to optimize its organization for *readers* of the code.

## Input

`$ARGUMENTS` specifies what to review. Supported forms:

- **`uncommitted`** -- Review uncommitted changes, as reported by `git diff` and `git diff --staged`.
- **`last-commit`** -- Review the most recent git commit (`git diff HEAD~1`).
- **A list of files** -- Review the specified files in their entirety (for recently created or heavily modified files).
- **A list of functions or classes** -- Review the specified symbols in context.

If no argument is given, default to reviewing uncommitted changes.
If there are no uncommitted changes, default to reviewing the last commit.

## Procedure

1. **Identify files to review.** Collect all files from the diff or argument list that need review.

2. **Order files for review.** Review higher-level files first, before lower-level implementation details:
   1. **Documentation files** (README, RELEASE_NOTES, doc/**) -- Often explain new features/systems at the highest level, for users or developers.
   2. **Test files** (test/**) -- Define acceptance criteria for a feature and exercise new API surfaces.
   3. **Product code files** -- Among these, try to identify which are higher-level entrypoints and review those first.

3. **Review each file** in the determined order, looking for any of the code smells described below.

4. **Apply revisions** for each code smell found, following the guidance in that smell's entry.

## Code smells

### Symmetric branches written as guard clause

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

### Short CLI flags in subprocess calls

**Trigger:** Short flags like `-M`, `-w`, `-r` in `subprocess.run()` calls or command construction.

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

---

### Em/en dashes in comments and text

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

---

### Blank line between adjacent strongly related blocks

**Trigger:** A blank line separating two code blocks that handle closely related concerns in the same logical flow (e.g., two sequential error checks on the result of the same operation).

**When to revise:** When the blocks are tightly coupled steps -- e.g., checking `FileNotFoundError` then checking `returncode`, both handling failures of the same `subprocess.run()` call.

**When NOT to revise:** When the blocks handle genuinely different phases of the function, or when a blank line aids readability by separating a long block from unrelated logic below.

**Fix:** Remove the blank line to visually group the related blocks.

---

### Loose Notes - not yet elaborated

- **Local imports in main() entry points:** In files that serve as the first module loaded when Python starts (containing a `main()` function), it is *conventional* to keep project-specific (non-stdlib) imports local to `main()` or similar functions. This minimizes startup time for commands like `--help` that don't need the full module graph. Do not move these imports to the top of the file.
