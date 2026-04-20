# Specific exceptions for meaningful failure modes

## Quick trigger

Error handling that either:

1. **Catches `Exception` or bare `except:` to handle one specific recoverable condition.** The broad catch swallows unrelated errors, masking bugs. Bare `except:` is the same smell — arguably worse, since it also catches `KeyboardInterrupt` and `SystemExit`.
   ```python
   try:
       last_windows = self._client.list_windows()
       if len(last_windows) == count: return last_windows
   except Exception:   # server not ready yet? or a real bug?
       pass

   try:
       ...
   except:             # same smell; also swallows KeyboardInterrupt/SystemExit
       pass
   ```

2. **Combines unrelated exception types under one handler.** Each branch means something different, but they're lumped together.
   ```python
   try:
       pid = int(pid_path.read_text().strip())
   except (ValueError, OSError):   # "file missing" and "garbage pid" are different bugs
       return
   ```

3. **Returns `None` / sentinel for a recoverable condition that deserves its own named type.**

**Why AI-drafted code does this:** LLMs frequently reach for `except Exception` in exactly the spots where *something* is expected to fail but they don't know the exact type. Narrowing the catch requires either deep library knowledge or running the code; typing `except Exception` immediately gets tests green. The broad catch then ships, and real bugs get silently swallowed for months.

Exception: **`except BaseException:` is usually intentional.** It's used when *every* exception — including `KeyboardInterrupt` and `SystemExit` — must be captured and forwarded elsewhere. Most common: RPC servers serializing exceptions for the caller, `Future`/`Promise` implementations capturing rejections, thread/task supervisors logging before rethrowing. If you see `except BaseException:`, don't treat it as the same smell; check whether it's forwarding the exception rather than handling it.

## Why revise

Broad `except Exception` and lumped tuple-excepts are debugging hazards:

- They **hide bugs.** A NameError, AttributeError, or fresh dependency failure is silently treated as "expected." The symptom: tests pass mysteriously, or a feature "just stops working" with no traceback.
- They **lose information.** Different failure modes often require different handling (retry vs. fail hard vs. ignore). When they share a handler, the code can't discriminate.
- They **obscure intent.** Reading `except Exception: pass` forces the reader to guess: what did the author *expect* to fail here? A named exception states the expectation directly.

Defining a specific exception type for an expected-recoverable condition turns a guess into a contract:

```python
try:
    self._client.list_windows()
except GvcGuiNotDoneStarting:  # explicitly expected; other exceptions propagate
    pass
```

Now a fresh bug in `list_windows` surfaces as a real traceback; only the anticipated "server not ready yet" branch is swallowed.

## When NOT to revise

- **Truly open-ended error contexts** where any exception should be logged and the loop should continue (e.g., top-level request handler that must not die). Keep broad `except Exception` *with* `traceback.print_exc()` — the logging preserves the information that a broad catch would otherwise lose.
- **One-liners where the tuple-except is genuinely the same handling** (e.g., both branches should return the same default, both represent "missing config"). Only split when the branches warrant different comments, different logging, or future divergence.

## Fix

1. Identify the specific failure modes the code actually expects.
2. For each expected mode, define a named exception (or locate the stdlib one that already applies: `FileNotFoundError`, `ConnectionRefusedError`, `TimeoutError`).
3. Raise the specific type at the point of failure; catch the specific type at the recovery point.
4. Let unexpected exceptions propagate.

For combined tuple-excepts: split into separate `except` branches, each with its own handling and comment. This is not just cosmetic — differentiating `FileNotFoundError` ("not started yet") from `ValueError` ("started but wrote garbage") often reveals race conditions the combined form hides.

## Example

Before — bare `Exception` in a poll loop, and a tuple-except that lumps unrelated failures:

```python
def wait_for_windows(self, count: int, timeout: float = 5.0) -> list[WindowInfo]:
    deadline = time.monotonic() + timeout
    last_windows: list[WindowInfo] = []
    while time.monotonic() < deadline:
        try:
            last_windows = self._client.list_windows()
            if len(last_windows) == count:
                return last_windows
        except Exception:
            pass
        time.sleep(0.1)
    raise TimeoutError(...)


def teardown(self) -> None:
    pid_path = self.sandbox.runtime_dir / "gvc.pid"
    if not pid_path.exists():
        return
    try:
        pid = int(pid_path.read_text().strip())
    except (ValueError, OSError):
        return
    ...
```

After — a named exception for "server not ready yet"; `FileNotFoundError` and `ValueError` are split, each with its own explanation:

```python
# in client.py
class GvcGuiNotDoneStarting(Exception):
    pass


class TestClient:
    def _call(self, method: str) -> object:
        """
        Raises:
        * GvcGuiNotDoneStarting
        """
        with closing(socket.socket(...)) as s:
            try:
                s.connect(str(self._sock_path))
            except FileNotFoundError:
                raise GvcGuiNotDoneStarting()
            ...


# in app.py
def wait_for_windows(self, count: int, timeout: float = 5.0) -> list[WindowInfo]:
    deadline = time.monotonic() + timeout  # capture
    last_windows: list[WindowInfo] = []
    while time.monotonic() < deadline:
        try:
            last_windows = self._client.list_windows()
        except GvcGuiNotDoneStarting:
            pass
        else:
            if len(last_windows) == count:
                return last_windows
        time.sleep(0.1)
    raise TimeoutError(...)


def close(self) -> None:
    pid_path = self.sandbox.runtime_dir / "gvc.pid"
    try:
        pid = int(pid_path.read_text().strip())
    except FileNotFoundError:
        # Process never started or finished starting
        return
    except ValueError:
        # Pid file exists but process never finished writing pid to it
        # TODO: Wait a bit longer for pid to be written, then kill the process
        return
    ...
```

Note that splitting the `except` also created a place to put the `TODO` for a real race condition — which the combined form would have buried.
