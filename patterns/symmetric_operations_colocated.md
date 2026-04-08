# Symmetric operations co-located

## Quick trigger

You see a send/receive, read/write, encode/decode, serialize/deserialize, or similar paired operation where one half lives in a different module or file from the other.

## Why revise

Symmetric operations are tightly coupled -- when one changes, the other almost always must change too. Separating them across modules makes it easy to update one side and forget the other. Co-locating them (in the same module, adjacent to each other) makes the pairing obvious and discoverable.

This is a cohesion improvement: coupled logic that must change together should live in nearby locations.

## When NOT to revise

- When the two operations are in different layers with genuinely different responsibilities (e.g., a high-level API serializer vs. a low-level transport deserializer that intentionally decouple).
- When moving one operation would create a circular import or break a clean dependency direction.

## Fix

1. Identify which module is the natural home for the paired operations (usually the module that already defines the data format or protocol).
2. Extract the displaced operation into a function in that module.
3. Place the two operations adjacent to each other in the file so the pairing is visually obvious.
4. Update all call sites to import from the new location.

## Example

Before -- receive logic inline in `_gui.py`, send logic in `_ipc.py`:

```python
# _ipc.py
def try_send(sock_path: Path, request_filepath: Path) -> bool:
    sock = socket.socket(socket.AF_UNIX, socket.SOCK_STREAM)
    sock.sendall(str(request_filepath).encode("utf-8"))
    sock.close()
    ...

# _gui.py  (receive logic is here, far from send)
def _socket_listener(server_sock, api):
    ...
    chunks: list[bytes] = []
    while chunk := conn.recv(4096):
        chunks.append(chunk)
    conn.close()
    tmp_path = Path(b"".join(chunks).decode("utf-8").strip())
```

After -- both operations in `_ipc.py`, adjacent:

```python
# _ipc.py
def try_send(sock_path: Path, request_filepath: Path) -> bool:
    sock = socket.socket(socket.AF_UNIX, socket.SOCK_STREAM)
    sock.sendall(str(request_filepath).encode("utf-8"))
    sock.close()
    ...

def receive(conn: socket.socket) -> Path:
    chunks: list[bytes] = []
    while chunk := conn.recv(4096):
        chunks.append(chunk)
    return Path(b"".join(chunks).decode("utf-8").strip())
```

## Common pairs to watch for

- `send` / `receive`
- `write` / `read` (file formats, serialization)
- `encode` / `decode`
- `serialize` / `deserialize`
- `pack` / `unpack`
- `request` / `response` (building vs. parsing)
- `open` / `close`
