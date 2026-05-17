# Function name references an implementation technique

**Trigger:** A function name references an uncommon named algorithm or animation/programming technique (FLIP, <TODO: need more examples>) instead of describing what the function *does in the domain*.

**Why:** A function name should answer "what does this do?" in the reader's vocabulary. Technique names answer "how does this work?" — which is implementation detail. Two failure modes:

1. **Reader doesn't know the technique.** They have to look it up before they can understand the call site. The name fails as documentation.
2. **Author isn't sure whether it's a known technique** (e.g. a draft labels something `_flipAnimate` but the author has invented the term, or assumes others will recognize it). Either way, the name is fragile — its meaning depends on the reader recognizing jargon.

The reverse path *is* useful: an implementation comment that says "uses the FLIP technique <link>" lets a reader cross-reference if they want, without forcing them to.

**Fix:** Rename the function to describe what it does in the domain. Move the technique reference (with a link if it's a well-known public technique) into an implementation comment near the algorithm itself.

Before:
```javascript
// Flip: capture rects, mutate, then animate from old position to new.
function _flipAnimate(elements, mutate) { ... }

function _flipReorderOutline(mutate) { ... }
```

After:
```javascript
// Animate elements from their pre-mutation positions to their post-mutation
// positions. Uses the FLIP technique:
// https://css-tricks.com/animating-layouts-with-the-flip-technique/
function _reorderOutlineRowsWithAnimation(elements, mutate) { ... }

function _reorderAllOutlineRows(mutate) { ... }
```

Section headers follow the same rule: `// FLIP Animation` → `// Reorder Animation`.

## When NOT to revise

- **The technique name *is* the domain vocabulary** for the project's audience. A linter's `parseExpression` doesn't need to be renamed even though "parsing" is a technique — because it's also what the function does in the domain.
- **Library exports whose public contract is the technique.** E.g. lodash's `_.debounce` — renaming it would break callers. Inside such a library, internal callsites can keep the technique name because it matches the published API.
- **Universally-recognized utility verbs** (`sort`, `filter`, `hash`) — these have crossed from technique to vocabulary.
- **Commonly-recognized technique names** (`debounce`, `memoize`) — ditto

The rule applies when the technique is *non-obvious to the file's likely readers* and is doing the work of a domain verb. When in doubt, ask: "would a reader who's never heard of this technique understand the call site?"
