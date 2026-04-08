# Conditionally-meaningful fields or parameters

**Trigger:** A dataclass has fields (or a function has parameters) that are only meaningful when some condition holds. Common signals: a boolean flag that gates whether sibling fields/params matter, fields with default values that are only populated for one variant of the object, or a `status` Literal with values that imply different field sets.

**Why:** Conditionally-meaningful fields violate the type system's ability to enforce correctness. Callers can access `raw_size` even when `status != "large"`, getting a meaningless default. The type signature promises data that isn't always there.

**Fix:** Extract the conditional fields into a separate type and use a union or optional reference. The absence of the object (`None`) replaces the boolean flag.

Before:
```python
@dataclass
class FileDiff:
    status: Literal["added", "deleted", "modified", "large"]
    old_path: str
    new_path: str
    # Only meaningful when status == "large":
    raw_size: int = 0
    raw_lines: int = 0

def render(file_diffs: list[FileDiff], *, large: bool = False,
           raw_size: int = 0, raw_lines: int = 0) -> str: ...
```

After:
```python
@dataclass(frozen=True)
class LargeDiffInfo:
    byte_count: int
    line_count: int

    @staticmethod
    def try_parse(diff_bytes: bytes) -> LargeDiffInfo | None: ...

@dataclass
class FileDiff:
    status: Literal["added", "deleted", "modified"]
    old_path: str
    new_path: str

def render(file_diffs: list[FileDiff], *,
           large_diff_info: LargeDiffInfo | None = None) -> str: ...
```

**How to spot:** Look for boolean parameters paired with data parameters that are only used inside the `if` branch. Look for dataclass fields with default values that are only set in one code path. Look for Literal types with values that imply different field sets. Look for call sites that pass meaningless placeholder values (like `[]` or `0`) to satisfy parameters that aren't relevant in that code path.

## Variant: union instead of optional

When the conditionally-meaningful data represents a completely different input mode (not just "present or absent"), use a union type rather than an optional parameter. This forces callers to pass exactly one variant and forces the receiving function to handle each variant explicitly.

Before (optional parameter still allows meaningless placeholders):
```python
def render(file_diffs: list[FileDiff], *,
           large_diff_info: LargeDiffInfo | None = None) -> str: ...

# Caller must pass a meaningless empty list:
render([], large_diff_info=ld_info)
```

After (union parameter -- no meaningless placeholders):
```python
def render(file_diffs: list[FileDiff] | LargeDiffInfo) -> str:
    if isinstance(file_diffs, LargeDiffInfo):
        ...
    elif isinstance(file_diffs, list):
        ...
    else:
        assert_never(file_diffs)

# Caller passes exactly what it has:
render(large_diff_info)
```
