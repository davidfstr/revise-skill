# Name implies wrong type

**Trigger:** A variable, constant, or parameter name suggests a different data type than what it actually holds. Most commonly: plural nouns or collective-sounding names used for scalar counts.

**Why:** A name should let you guess its data type even without a type annotation. `LARGE_BYTES` sounds like a collection of bytes; `LARGE_LINES` sounds like a list of line strings. When the name implies the wrong type, readers form incorrect mental models and write bugs.

**Fix:** Add a suffix that clarifies the actual type. Common fixes: `xs` -> `x_count` for scalars that count items.

Before:
```python
LARGE_BYTES = 1_048_576
LARGE_LINES = 10_000
```

After:
```python
_LARGE_DIFF_BYTE_COUNT = 1_048_576
_LARGE_DIFF_LINE_COUNT = 10_000
```
