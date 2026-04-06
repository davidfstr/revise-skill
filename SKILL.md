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

<!-- Patterns will be added here incrementally, as they are discovered through revision walkthroughs. -->

*No patterns catalogued yet.*
