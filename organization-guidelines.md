# Organization Guidelines for the /revise Skill

Principles governing how this skill document is structured, to keep it effective as it grows.

## Progressive disclosure

The main SKILL.md is an **index**, not a reference manual. Each pattern entry has:

1. A **trigger line** -- 1-2 sentences describing how to quickly recognize the smell.
2. A **short "before" snippet** -- just enough code to confirm recognition. No "after" needed in the index.
3. A **link to a detail file** -- containing the full Why / When NOT / Fix / Before-After guidance.

This lets the reader scan quickly, then drill into details only for patterns that match.

## 7-10 items per hierarchy level

At any level of grouping (categories, patterns within a category), aim for at most 7-10 items. Beyond that, readers struggle to keep the full set in focus and start skipping entries.

If a category grows past 10 patterns, split it into subcategories. If the top-level category count exceeds 10, consolidate or merge.

## Concrete, scannable category names

Category names should be concrete enough that a reader can guess what patterns they contain without reading the entries. Prefer names tied to observable code properties ("Good Names", "Type Safety") over abstract design principles ("Anti-Obscurity", "Cohesion").

## Pattern maturity lifecycle

Patterns progress through stages:

1. **Loose note** -- A brief observation in the "Loose patterns" section of the index. No detail file yet. Waiting for more examples or confidence.
2. **Elaborated but uncategorized** -- Has a full detail file with Why/When NOT/Fix/Before-After, but doesn't yet fit cleanly into a category. Listed under "Uncategorized (elaborated)."
3. **Categorized** -- Placed in the appropriate category in the index, with a detail file.

Similarly, **loose categories** are listed at the bottom of the index as potential future groupings, waiting for enough patterns to justify them.

## When to reorganize

Review the skill's organization after incorporating patterns from a related sequence of commits (e.g., a multi-part refactoring). Signs that reorganization is needed:

- A category has grown past 7-10 items.
- A pattern feels like it belongs in two categories (may need a better category boundary).
- The "Uncategorized" or "Loose patterns" sections are growing faster than patterns graduate out of them.
