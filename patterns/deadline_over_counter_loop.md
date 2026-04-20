# Deadline-based wait loops (with `time.monotonic()`)

## Quick trigger

A time-based wait uses a **counter loop** instead of a **deadline loop**, and/or uses `time.time()` instead of `time.monotonic()` to measure duration.

```python
# Counter loop — claims to wait "3 seconds" but actually waits at least 3s
# and possibly far longer under system load.
for _ in range(30):
    time.sleep(0.1)
    if condition():
        return
```

```python
# time.time() for duration — broken by clock changes (NTP, DST, manual adjust)
start = time.time()
while time.time() - start < 5.0:
    ...
```

## Why revise

**Counter loops drift under load.** `time.sleep(0.1)` is a *minimum* — the OS can delay return by seconds under CPU pressure, swap, or a stop-the-world GC. `for _ in range(30): time.sleep(0.1)` reads as "3 seconds" but can easily take 10+. If the loop is a timeout, the timeout is a lie.

A deadline loop states the actual intent — "keep trying until this wall-clock moment" — and self-corrects: if one iteration takes 2s, the next check compares against a fixed deadline, not a cumulative counter.

**`time.time()` is not monotonic.** It reflects the system clock, which can jump backward (NTP correction, DST transitions, manual user changes). A `time.time()`-based duration can compute a negative elapsed time, time out instantly, or hang indefinitely. `time.monotonic()` is guaranteed non-decreasing and unaffected by clock adjustments — it exists specifically to measure durations.

## When NOT to revise

- **Fixed-count retries** where "try N times" is the spec, not "try for T seconds." E.g., retry a network call 3 times with backoff.
- **Single-shot sleeps** (`time.sleep(5)`) where you want to pause for a known duration without polling.
- **Code that intentionally follows wall-clock time** (scheduling a job "at 9:00 AM"). Use `time.time()` or `datetime` there — monotonic clocks have no relation to wall-clock.

## Fix

Replace counter loops with:

```python
deadline = time.monotonic() + timeout  # capture
while time.monotonic() < deadline:
    if condition():
        return
    time.sleep(poll_interval)
raise TimeoutError(...)
```

Note the `# capture` marker — see [assignment_marker_comments.md](assignment_marker_comments.md).

Replace `time.time()`-based duration measurement with `time.monotonic()` across the board. The only argument for `time.time()` in a duration is "I want the elapsed wall-clock time even if the clock jumps," which is almost never what you want.

## Example

Before:
```python
def teardown(self) -> None:
    ...
    os.kill(pid, signal.SIGTERM)
    ...
    # Wait up to 3 s for a clean exit, then force-kill
    for _ in range(30):
        time.sleep(0.1)
        try:
            os.kill(pid, 0)
        except ProcessLookupError:
            return
    os.kill(pid, signal.SIGKILL)
```

After:
```python
def close(self) -> None:
    ...
    os.kill(pid, signal.SIGTERM)
    ...
    # Wait up to 3s for a clean exit, then force-kill
    deadline = time.monotonic() + 3.0  # capture
    while time.monotonic() < deadline:
        time.sleep(0.1)
        try:
            os.kill(pid, 0)  # process exists? (POSIX-only)
        except ProcessLookupError:
            return
    os.kill(pid, signal.SIGKILL)
```

Similarly for poll loops:

```python
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
```
