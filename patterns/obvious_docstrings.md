# Obvious/redundant docstrings

## Quick trigger

A docstring that restates what is already clear from the function/class/module name, or that duplicates information available elsewhere in the same file.

Common locations where AIs add these:

- **Test functions:** `def test_user_can_log_in(): """Test that a user can log in."""` -- The test name already says this, especially when using sentence-style given-when-then naming.
- **Module docstrings:** A module docstring that merely repeats the contained class's docstring or restates what the module name implies.
- **Short self-explanatory functions:** `def _expand_tabs(text: str, tabsize: int = 4) -> str: """Expand tabs to spaces."""` -- Borderline; depends on whether the function name is clear enough in context.

## Why revise

Obvious docstrings add visual noise without informing the reader. They train readers to skip docstrings entirely, which means genuinely useful docstrings get less attention.

## When NOT to revise

- When the docstring adds information beyond the name: preconditions, thread-safety guarantees, non-obvious return values, side effects.
- When the function is part of a public API where documentation tools (Sphinx, pdoc) will render the docstring.
- When the function name is abbreviated or domain-specific enough that even a "restating" docstring helps newcomers.

## Fix

Remove the docstring. If there is non-obvious information buried in an otherwise obvious docstring, extract that information into a separate comment or a more focused docstring.

## Example

Before -- test docstrings that restate the test name:

```python
def test_given_missing_prefs_file_when_load_then_returns_defaults():
    """Test that loading returns defaults when prefs file is missing."""
    ...

def test_given_malformed_prefs_file_when_load_then_returns_defaults():
    """Test that loading returns defaults when prefs file is malformed."""
    ...
```

After:

```python
def test_given_missing_prefs_file_when_load_then_returns_defaults():
    ...

def test_given_malformed_prefs_file_when_load_then_returns_defaults():
    ...
```
