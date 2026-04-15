# Docstring verb form

## Quick trigger

A docstring summary begins with an imperative verb: "Build X", "Return Y", "Generate Z".

## Why revise

Although PEP 257 recommends the imperative mood, third-person indicative ("Builds X", "Returns Y", "Generates Z") reads more naturally as a *description of what the function does* rather than a command to the reader. Consistency across a codebase matters more than matching PEP 257, and this project has settled on the indicative form.

## When NOT to revise

- When the project's existing docstrings are consistently imperative -- follow the project's convention, don't mix forms.
- When the "docstring" is actually a module docstring describing a script's effect (e.g., `"""Run with: poetry run python build_app.py"""`) -- those can read as instructions to the user.

## Fix

Convert the leading imperative verb to third-person indicative by adding `s` (or `es`).

## Example

Before:
```python
def main() -> None:
    """Build ./dist/gvc.app via PyInstaller."""

def _build_icns() -> None:
    """Generate build/icon.icns from the source PNG using sips + iconutil."""
```

After:
```python
def main() -> None:
    """Builds ./dist/gvc.app via PyInstaller."""

def _build_icns() -> None:
    """Generates build/icon.icns from the source PNG using sips + iconutil."""
```
