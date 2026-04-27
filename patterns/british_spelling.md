# British English spelling

**Trigger:** British English spelling in code comments, documentation, or user-facing text. Common examples: `behaviour` (behavior), `initialise` (initialize), `colour` (color), `serialise` (serialize).

**Why:** Consistency. Most Python ecosystem documentation uses American English. Mixing spellings within a codebase creates inconsistency that can also affect grep-ability (searching for "initialize" misses "initialise").

**Fix:** Use American English spelling.
