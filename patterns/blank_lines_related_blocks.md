# Blank line between adjacent strongly related blocks

**Trigger:** A blank line separating two code blocks that handle closely related concerns in the same logical flow.

Common shapes:

- Two sequential error checks on the result of the same operation -- e.g., checking `FileNotFoundError` then checking `returncode`, both handling failures of the same `subprocess.run()` call.
- Variable declarations whose sole purpose is to be closed over / mutated by a nested function defined immediately below. The declarations and the `def` form one setup-then-dispatch block.

**Why revise:** A blank line tells a scanning reader "a new concern starts here." When the blocks are really *one* concern, the blank line misdirects attention and lets the reader's eye skip over coupling they need to see (e.g., that the `nonlocal` inside the inner function refers to variables declared three lines up).

For the nested-function case specifically: PEP 8's blank-line rule applies only to top-level functions/classes and class methods. It has nothing to say about nested `def`s inside a function body. The habit of "always put one blank line before a `def`" is an over-generalization of PEP 8, not a convention.

**When NOT to revise:** When the blocks handle genuinely different phases of the function, or when a blank line aids readability by separating a long block from unrelated logic below. Also keep the blank line when a nested function is *not* tightly coupled to the lines immediately above it (e.g., a helper defined mid-function that just happens to follow unrelated setup).

**Fix:** Remove the blank line to visually group the related blocks.

## Example: nested function closing over preceding declarations

Before -- a blank line implies `done`/`error` and `_block` are separate concerns, but `_block` exists *to produce* them:

```python
done = threading.Event()
error: BaseException | None = None

def _block() -> None:
    nonlocal error
    try:
        browser.webview.setAppearance_(ns_appearance)
    except BaseException as e:
        error = e
    finally:
        done.set()
NSOperationQueue.mainQueue().addOperationWithBlock_(_block)
done.wait(timeout=5.0)
```

After -- declarations + `def` + dispatch form one visible unit:

```python
done = threading.Event()
error: BaseException | None = None
def _block() -> None:
    nonlocal error
    try:
        browser.webview.setAppearance_(ns_appearance)
    except BaseException as e:
        error = e
    finally:
        done.set()
NSOperationQueue.mainQueue().addOperationWithBlock_(_block)
done.wait(timeout=5.0)
```
