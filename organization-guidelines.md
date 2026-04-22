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

## Inverse patterns: surface them, don't nest them

When learning a pattern that's the *inverse* or *complementary opposite* of an existing one -- e.g., "when to REMOVE a blank line" paired with "when to ADD a blank line" -- treat the inverse as its own pattern, not as a sub-note buried inside the source pattern.

**Default placement when first captured:** a top-level section at the end of the source pattern's detail file (its own `#` heading, separate from the source pattern's examples and "When NOT to revise"). This keeps the two cases discoverable together while signaling that the inverse is a distinct pattern.

**Why not nest inside "When NOT to revise":** that section is for boundary conditions of the same pattern. A genuinely inverse case has its own triggers, rationale, and fix -- not just "don't apply the rule here." Burying it misleads a reader looking up the inverse directly, and the trigger-scan pass in /revise is unlikely to descend into another pattern's "When NOT" to find it.

**Promote to a standalone pattern when ready:** once an inverse section has real examples and judgment calls, promote it to:

1. Its own `patterns/<name>.md` file.
2. Its own bullet in SKILL.md (possibly in a different category if the inverse belongs elsewhere).

Leave a "See also" cross-reference on the source pattern's file.

**Review periodically:** inverse sections that have been sitting in the bottom of their source file across several /revise sessions without growing are probably stable enough to promote -- extracting them makes them directly discoverable from the SKILL.md index.

## When to reorganize

Review the skill's organization after incorporating patterns from a related sequence of commits (e.g., a multi-part refactoring). Signs that reorganization is needed:

- A category has grown past 7-10 items.
- A pattern feels like it belongs in two categories (may need a better category boundary).
- The "Uncategorized" or "Loose patterns" sections are growing faster than patterns graduate out of them.
- An embedded "inverse case" section in a pattern file has accumulated enough content to justify promotion to a standalone pattern.
