# `let`/`var` used instead of `const` (JavaScript/TypeScript)

## Quick trigger

A JavaScript/TypeScript variable is declared with `let` (or `var`) when the value is never reassigned after initialization.

```typescript
let result = computeThing();   // never reassigned below — should be const
return format(result);
```

## Why revise

In languages that offer concise final-by-default variables (`const` in JS/TS, `let` in Rust/Swift, `val` in Kotlin), the declaration itself is a signal:

- **`const`** = "this binding never changes" — the reader can ignore it as a possible source of mutation bugs.
- **`let`** (or `var`, `mut`) = "this binding is mutable, expect reassignments below" — a local anomaly the reader should track.

If mutability is the exception, `let` is automatically the marker. No `# reinterpret`-style comment is needed (see [unmarked_reassignment.md](unmarked_reassignment.md) for the Python equivalent) because the language provides the signal.

Defaulting to `let` everywhere destroys that signal. Every declaration now *claims* to be mutable, even when 95% of them never are, and the reader can't tell which variables actually change without reading the whole function.

## When NOT to revise

- **Variables that genuinely are reassigned** — keep `let`. That's the whole point of the convention.
- **Loop-local counters and accumulators** where reassignment is structural (`for (let i = 0; ...)`) — idiomatic `let`, don't convert.
- **Existing files that consistently use `var` (or `let`) throughout**, especially old JavaScript code that predates `const`. Match the file's existing convention rather than introducing a split style. Convert the whole file in a separate, standalone commit if the upgrade is worth doing.

## Fix

1. Scan for `let x = ...` / `var x = ...` declarations.
2. For each, check whether `x` is ever reassigned (not just mutated — `const obj = {}; obj.k = 1` is fine).
3. If never reassigned, change `let`/`var` → `const`.

TypeScript/ESLint rule `prefer-const` automates this — enable it if the project allows linter config changes.

## Example

Before:
```typescript
function render(items: Item[]): string {
    let header = buildHeader(items);
    let body = items.map(formatItem).join("\n");
    let footer = buildFooter(items.length);
    return `${header}\n${body}\n${footer}`;
}
```

After:
```typescript
function render(items: Item[]): string {
    const header = buildHeader(items);
    const body = items.map(formatItem).join("\n");
    const footer = buildFooter(items.length);
    return `${header}\n${body}\n${footer}`;
}
```

Now any future `let` in this function visually pops out as "this one actually mutates."
