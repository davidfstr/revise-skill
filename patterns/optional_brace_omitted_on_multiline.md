# Multi-line if/loop body without braces (optional-brace languages)

**Trigger:** A language permits optional braces around single-statement `if`/`else`/`for`/`while` bodies (JS/TS, C, C++, Java, Perl), and a function uses the unbraced multi-line form:

```javascript
if (rows.length === 0)
    break;
```

**Why:** The classic "goto-fail" edit hazard. A later author adds a second statement under the `if`, indents it correctly, and the language silently puts it *outside* the conditional:

```javascript
if (rows.length === 0)
    cleanup();
    break;          // looks indented under the if; actually unconditional
```

Always-braced bodies make the boundary unambiguous and produce cleaner diffs when statements are added.

**Fix:** Add braces.

Before:
```javascript
if (rows.length === 0)
    break;

if (delta === 0)
    continue;
```

After:
```javascript
if (rows.length === 0) {
    break;
}

if (delta === 0) {
    continue;
}
```

## When NOT to revise — same-line guards

A guard kept entirely on one line (`if (x) return;`) is the *single-line* form, idiomatic for short guards, and is covered by [single_line_if_statement.md](single_line_if_statement.md). Don't expand those — only the multi-line unbraced form triggers this pattern.

## Scope

- **Applies:** JS, TypeScript, C, C++, Java, Perl — anywhere braces are optional and indentation does not bind to scope.
- **Does not apply:** Python (no braces; indentation *is* the scope). Go and Rust (braces mandatory by the language).
