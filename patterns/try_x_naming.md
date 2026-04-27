# Failable operation not named `try_X`

## Quick trigger

A function whose bare name implies it raises on failure (or always succeeds) actually returns failure as a value — typically `bool` for effect-only methods, or `T | None` for parse/decode/strict-finder variants — without a `try_` prefix to flag this.

Strongest case: a `bool`-returning effect/mutation method. Bare names like `send(...)`, `insert(...)`, `close(...)`, or `normalize(...)` read as fire-and-forget actions; nothing in the name nor the type system forces the caller to check the return.

## Why revise

The `try_` prefix tells the reader: failure here is a value, not an exception. It earns its keep when at least one of these is true:

1. **Effect-only / mutation method that returns `bool`.** Without the prefix, callers may treat it as fire-and-forget and silently miss failure. The `bool` return alone is not enough — a bool you ignore is still legal. Examples: `try_send`, `try_insert_script`, `try_normalize_url`, `try_close`.
2. **A non-`try_` peer exists that raises.** The prefix is the disambiguator: e.g., `get_block(...) -> Block` (asserting) alongside `try_get_block(...) -> Block | None` (non-asserting). Idiomatic in test page-object utilities and similar settings where assertion is the default.
3. **Bare name implies success.** `parse`, `decode`, `normalize`, `pick` read as "succeeds or raises" idioms. `try_parse` marks the non-raising variant.

## When NOT to revise

The prefix is not load-bearing — and adding it is just noise — when any of these hold:

- **The bare name already implies optionality.** `find_X`, `lookup_X`, `peek_X` already signal "may not be there." A `find_child(...) -> TreeItem | None` is no clearer as `try_find_child(...) -> TreeItem | None`.
- **`T | None` return with no raising peer**, where every caller is forced by the typechecker to handle `None`. The prefix duplicates information the type already carries.
- **All callers necessarily inspect the return value** for domain reasons — there is no realistic risk of the function being misread as fire-and-forget.

A separate decision precedes this one: in languages with exceptions, designing a function to return an error code/object instead of raising is usually the wrong choice. The `try_X` naming pattern only applies once you've decided a non-raising variant is genuinely warranted.

## Fix

Add the `try_` prefix to the function name. If the codebase needs both raising and non-raising variants, keep the bare name for the raising version and use `try_X` for the non-raising one.

## Example

Effect-only method returning `bool` — the strongest case:

Before:
```python
def send(sock_path: Path, request_filepath: Path) -> bool:
    try:
        with closing(socket.socket(...)) as sock:
            sock.connect(str(sock_path))
            sock.sendall(...)
        return True
    except (FileNotFoundError, ConnectionRefusedError):
        return False

# call site — looks like fire-and-forget; failure goes silent
send(sock_path, request_filepath)
```

After:
```python
def try_send(sock_path: Path, request_filepath: Path) -> bool: ...

# call site — prefix nudges the reader to check the result
if not try_send(sock_path, request_filepath):
    subprocess.Popen(...)  # fall back to launching the server
```

Paired raising + non-raising variants in a test page-object:

```python
def get_block(self, block_id: BlockId) -> Block:                  # raises if absent
    ...

def try_get_block(self, block_id: BlockId) -> Block | None:       # returns None if absent
    ...
```
