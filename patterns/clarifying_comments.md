# Missing clarifying comments on non-obvious code

## Quick trigger

Two situations within a function body:

1. **Non-obvious response to a situation.** Code handles a scenario in a way that a reader might question -- especially when it deliberately violates a common principle (e.g., silently swallowing an error when "errors must not pass silently" is the default expectation). Add a comment explaining *what* the response is and *why*.

2. **Long or complex paragraph.** A block of code within a function is 5-7+ lines or contains dense/complex expressions. Add a leading label comment saying what the paragraph does, so readers can scan the function's structure without parsing every line.

## Why revise

Without these comments, a reader must mentally execute the code to understand what scenario is being handled and whether the response is intentional. This is especially costly for error-handling branches where a bare `return` or silent fallback looks like a bug.

## When NOT to revise

- When the code is self-documenting (the variable names and structure make the intent clear).
- When the function is short enough (under 5 lines) that every line is immediately visible in context.
- Do not add comments that merely restate what the code does. The comment should add information the code does not convey: the scenario, the rationale, or the high-level purpose.

## Fix

For non-obvious responses, state the situation and the deliberate response:
```python
# Situation. Deliberate response.
```

For paragraph labels, state what the paragraph accomplishes:
```python
# Brief description of what follows
```

## Example

Before -- silent fallbacks with no explanation:

```python
@classmethod
def load(cls) -> Prefs:
    path = cls._path()
    if not path.exists():
        return cls()
    try:
        with path.open("r") as fh:
            data = json.load(fh)
        known_field_names = {f for f in cls.__dataclass_fields__}
        return cls(**{k: v for k, v in data.items() if k in known_field_names})
    except (json.JSONDecodeError, TypeError):
        return cls()
```

After -- each scenario labeled:

```python
@classmethod
def load(cls) -> Prefs:
    path = cls._path()

    if not path.exists():
        # Missing prefs file. Use default prefs.
        return cls()

    # Read prefs file
    try:
        with path.open("r") as fh:
            data = json.load(fh)
        known_field_names = {f for f in cls.__dataclass_fields__}
        return cls(**{k: v for k, v in data.items() if k in known_field_names})
    except (json.JSONDecodeError, TypeError):
        # Malformed prefs file. Revert to default prefs silently.
        return cls()
```
