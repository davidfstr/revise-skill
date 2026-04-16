# Exhaustive dispatch on variant types

**Trigger:** A switch-like dispatch on a set of known variants that exhibits any of these antipatterns:

1. **Last variant handled by unchecked `else`:** The final `else` (or a bare ternary) assumes the remaining type without an explicit check. New variants silently fall through.
2. **No `else` branch at all:** The chain handles all current variants but has no fallback. New variants silently do nothing.
3. **Ternary assumes two variants:** `x === 'a' ? ... : ...` treats the `else` arm as "must be b" -- unsafe when a third variant `c` is added later.

All are unsafe: adding a new variant later won't produce an error at dispatch sites.

**Fix (general principle):** Make every variant an explicit check, then add a final `else` that throws. The exact mechanism depends on the language:

- **Python:** `else: assert_never(x)` (from `typing`). Gives both a runtime error and a static type error.
- **TypeScript:** `else { throw new Error(...) }` for runtime safety; for compile-time exhaustiveness, use a `Record<UnionType, Value>` lookup or a function with a `never`-typed default.
- **Ternaries:** Replace with an explicit if/else chain, a lookup object, or a function that checks each variant and throws on unknown values.

## Python

Before (unchecked else):
```python
type Shape = Triangle | Square | Circle

shape: Shape = ...
if isinstance(shape, Triangle):
    ...
elif isinstance(shape, Square):
    ...
else:  # Circle -- unsafe: silently swallows new variants
    ...
```

Before (no else):
```python
if isinstance(shape, Triangle):
    ...
elif isinstance(shape, Square):
    ...
elif isinstance(shape, Circle):
    ...
# no else -- unsafe: new variants silently do nothing
```

After:
```python
if isinstance(shape, Triangle):
    ...
elif isinstance(shape, Square):
    ...
elif isinstance(shape, Circle):
    ...
else:
    assert_never(shape)
```

## TypeScript / JavaScript

Ternaries that dispatch on string literal unions are a common source of this pattern. They look small and innocent but silently swallow new variants.

Before (ternary assumes two variants):
```typescript
type TokenType = 'verb' | 'keyword';

// Unsafe: if a third variant is added, it silently becomes 'keyword-class'
const cls = tokenType === 'verb' ? 'verb-class' : 'keyword-class';
const wrapped = tokenType === 'keyword' ? `[${text}]` : text;
```

After (lookup object -- compile-time exhaustive at definition site):
```typescript
// Record<TokenType, string> requires an entry for every variant at compile time
const CLASS_BY_TOKEN_TYPE: Record<TokenType, string> = {
    verb: 'verb-class',
    keyword: 'keyword-class',
};
const cls = CLASS_BY_TOKEN_TYPE[tokenType];
// NOTE: This is NOT runtime-safe if `tokenType` originates from untyped/external data, producing `undefined` at runtime silently.
// Add a throw guard if needed:
if (!cls) throw new Error(`Unexpected token type: ${tokenType}`);
```

After (function with explicit throw -- runtime exhaustive):
```typescript
function wrapTokenText(tokenType: TokenType, text: string): string {
    if (tokenType === 'verb') return text;
    if (tokenType === 'keyword') return `[${text}]`;
    throw new Error(`Unexpected token type: ${tokenType}`);
}
```

**When NOT to revise:**
- True boolean dispatch (`if (x) ... else ...`) where there are exactly two cases by nature.
- The value is inherently open-ended (e.g., an HTTP status code) and the `else` provides a genuine default, not an implicit assumption about the remaining variant.
