# Docstring scope: API guarantees vs. implementation details

## Quick trigger

A docstring that contains implementation details -- what specific library, implementation approach, or workaround was used. These details are for maintainers of the function, not callers.

## Why revise

Docstrings are the caller's interface documentation. They should describe stable API guarantees: what the function does, what it returns, and any constraints callers should know. Implementation details change more frequently and clutter the caller's view. Moving them to inline implementation `# NOTE:` comments keeps the docstring clean and puts the rationale next to the code it explains.

## When NOT to revise

- When the implementation detail IS an API guarantee (e.g., "Uses atomic os.replace for crash safety" if callers depend on that property).
- When the function is so short that the docstring and implementation are essentially the same thing.

## Fix

1. Identify which parts of the docstring are API guarantees callers care about.
2. Keep those in the docstring.
3. Move implementation rationale to `# NOTE:` comments near the relevant code.

## Example

Before -- implementation detail in docstring:

```python
def _is_dark_mode() -> bool:
    """Return True if the OS is currently in Dark Mode.
    
    Uses NSUserDefaults rather than NSApp.effectiveAppearance() because this
    is called before webview.start() initialises the Cocoa application, at
    which point NSApp is still None.
    """
    from Foundation import NSUserDefaults
    ...
```

After -- API guarantee in docstring, implementation detail in comment:

```python
def _is_dark_mode() -> bool:
    """
    Return True if the OS is currently in Dark Mode.
    
    Safe to call before webview.start() initializes the Cocoa application.
    """
    from Foundation import NSUserDefaults
    # NOTE: Uses NSUserDefaults rather than NSApp.effectiveAppearance() because this
    #       is called before webview.start() initializes the Cocoa application, at
    #       which point NSApp is still None.
    ...
```

Notice the split: "Safe to call before webview.start()" is the API guarantee callers care about. The NSUserDefaults detail is the implementation that provides that guarantee.
