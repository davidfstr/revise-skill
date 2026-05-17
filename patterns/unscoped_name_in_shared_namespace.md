# Short/generic/unscoped name in a shared namespace

**Trigger:** A symbol is defined in a context where *all* top-level names share a single flat namespace, but its name is short, generic, or doesn't indicate which subsystem/feature it belongs to. Risk of collision and ambiguity at the call site.

This pattern is **language- and tooling-specific** — it only applies when the language or build system does NOT supply a namespace boundary per file/module.

## Where it applies

- **JavaScript/TypeScript without module wrapping.** Multiple `<script>` tags, IIFE-bundled assets, or a minimal build system that concatenates files all share one global scope. A function `_onDragStart` in `diff-reorder.js` collides with `_onDragStart` in any other concatenated file.
- **CSS IDs and classes.** All selectors in all loaded stylesheets share one global namespace. Generic names like `.outline-row`, `#header` collide across apps, components, and embedded contexts. Convention: prefix every selector with an app-specific tag (TechSmart uses `ts-`, Crystal uses `cr-`, etc.).
- **C top-level function/global names.** No namespaces; the linker symbol table is flat. Conventionally prefixed with module/library tag (`png_read_info`, `sqlite3_open`).
- Other places where this applies as evidence accumulates: shell function names in long-lived bash scripts, Make targets across included makefiles, Objective-C class names, …

## Where it does NOT apply

- **Python.** Module path provides the namespace; `outline.on_keydown` and `find.on_keydown` coexist via import.
- **Go, Java, Rust, Swift, Kotlin.** Package/module qualification is built into the language.
- **ES modules with proper bundling.** Each module's top-level is file-local; no collision.

When in doubt, ask: "if a second file in this same project defined the same symbol name, what happens?" If the answer is *silent shadowing or link error*, the namespace is shared and this pattern applies.

**Fix:** Prefix the name with the subsystem, element, or feature it belongs to. For event handlers in JS, name after the element the listener is *attached to* (so the call site matches: `outline.addEventListener("keydown", _onOutlineKeydown)`). For CSS selectors, use the project-wide prefix convention.

## Examples

JavaScript (no modules — file concatenation):
```javascript
// Before: any other file could define these
outline.addEventListener("dragstart", _onDragStart);
outline.addEventListener("keydown", _onHandleKeydown);
outline.addEventListener("click", _onHandleClick);

// After: outline-scoped
outline.addEventListener("dragstart", _onOutlineDragStart);
outline.addEventListener("keydown", _onOutlineKeydown);
outline.addEventListener("click", _onOutlineClick);
```

CSS (global selector namespace):
```css
/* Before: collides with any host page or component */
.outline-row { ... }
#header { ... }

/* After: app-prefixed */
.gvc-outline-row { ... }
#gvc-header { ... }
```

C (flat linker namespace):
```c
// Before
int read_header(FILE *f);

// After: library-prefixed
int png_read_header(FILE *f);
```

## Notes for codebases mid-migration

Adopting an app-prefix convention often requires a dedicated cleanup commit — not piecemeal during feature work. If you find unprefixed names in a codebase that *is* converging on a convention, note it for the cleanup batch rather than fixing inconsistently. (For example: gvc's CSS currently lacks a `gvc-` prefix on its IDs and classes; that's a planned dedicated cleanup, not a per-PR fix.)
