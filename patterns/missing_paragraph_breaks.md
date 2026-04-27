# Multiple concerns not separated by blank lines

**Trigger:** Two or more distinct concerns sit adjacent inside a function with no blank line between them. The reader's eye flows continuously across what are really separate phases.

Common shapes:

- A function has distinct phases (setup vs. main operation, data preparation vs. dispatch, validation vs. work) packed without paragraph breaks.
- A region marked with `NOTE: Duplicated in X and Y` (see [duplicate_code.md](duplicate_code.md)) is not bookended by blank lines, so a maintainer cannot see where the "must stay in sync" zone ends.

**Why revise:** Blank lines act as paragraph breaks. Without them, scanning a function feels like reading prose with no paragraph indents — the reader cannot quickly locate phase boundaries. For duplicated regions specifically, missing brackets let the duplicated lines visually blend into surrounding logic, so a future editor may modify only one copy and miss the requirement to keep the other in sync.

**When NOT to revise:** When the adjacent blocks really are one concern. See [split_paragraph.md](split_paragraph.md) for the inverse pattern -- *removing* blank lines between tightly coupled blocks.

**Fix:** Insert a blank line at each phase boundary. Bookend `NOTE: Duplicated in X and Y` regions with blank lines on both sides.

## Example: phase boundary

Before -- validation, work, and result-shaping run together:

```python
def render_diff(diff_bytes: bytes, title: str) -> str:
    if not diff_bytes:
        raise ValueError("empty diff")
    if not title:
        raise ValueError("empty title")
    file_diffs = parse(diff_bytes)
    html = render(file_diffs)
    return wrap_in_document(html, title)
```

After -- one blank line between validation and the work phase:

```python
def render_diff(diff_bytes: bytes, title: str) -> str:
    if not diff_bytes:
        raise ValueError("empty diff")
    if not title:
        raise ValueError("empty title")

    file_diffs = parse(diff_bytes)
    html = render(file_diffs)
    return wrap_in_document(html, title)
```

## Example: bracketing a duplicated region

Before -- the duplicated region's extent is invisible:

```python
def handle_request(req: Request) -> Response:
    log.info("handling %s", req.id)
    # NOTE: Duplicated in handle_batch_request
    user = lookup_user(req.user_id)
    if user is None:
        return Response(status=404)
    if user.is_banned:
        return Response(status=403)
    return Response(status=200, body=user.to_json())
```

After -- blank lines bracket the must-stay-in-sync zone:

```python
def handle_request(req: Request) -> Response:
    log.info("handling %s", req.id)

    # NOTE: Duplicated in handle_batch_request
    user = lookup_user(req.user_id)
    if user is None:
        return Response(status=404)
    if user.is_banned:
        return Response(status=403)

    return Response(status=200, body=user.to_json())
```

---

Related:
- [split_paragraph.md](split_paragraph.md) — inverse: when adjacent blocks are one concern, *remove* the blank line
- [grouped_paragraphs.md](grouped_paragraphs.md) — when sub-sections grow past several paragraphs each, paragraph breaks alone aren't enough; wrap each in an anonymous block
