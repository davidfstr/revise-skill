---
name: revise
description: Perform common code quality revisions on AI-drafted code. Use when reviewing recently drafted code for readability and maintainability improvements.
disable-model-invocation: true
argument-hint: [diff-spec]
---

# Revise

Perform common code quality improvement revisions on code that was recently drafted by AI.

## Why this skill exists

AI-written code is usually drafted in a way that is easy for an AI to *write* at the time but not necessarily easy to *read* later (by human or AI). For example:

- AIs frequently use local imports in Python code -- contrary to standard Python convention, but easy to write in isolation without considering the rest of the file.
- AIs tend to write low-level functions before high-level functions -- easy to write a high-level function when you already know the names of the low-level functions it will call, but less efficient for readers who always want to start reading at high-level entrypoints.

The goal of revising code is to optimize its organization for *readers* of the code.

## Input

`$ARGUMENTS` specifies what to review. Supported forms:

- **`uncommitted`** -- Review uncommitted changes, as reported by `git diff` and `git diff --staged`.
- **`last-commit`** -- Review the most recent git commit (`git diff HEAD~1`).
- **A list of files** -- Review the specified files in their entirety (for recently created or heavily modified files).
- **A list of functions or classes** -- Review the specified symbols in context.

If no argument is given, default to reviewing uncommitted changes.
If there are no uncommitted changes, default to reviewing the last commit.

## Procedure

1. **Identify files to review.** Collect all files from the diff or argument list that need review.

2. **Order files for review.** Review higher-level files first, before lower-level implementation details:
   1. **Documentation files** (README, RELEASE_NOTES, doc/**) -- Often explain new features/systems at the highest level, for users or developers.
   2. **Test files** (test/**) -- Define acceptance criteria for a feature and exercise new API surfaces.
   3. **Product code files** -- Among these, try to identify which are higher-level entrypoints and review those first.

3. **Review each file** in the determined order, looking for any of the code smells described below.

4. **Apply revisions** for each code smell found, following the guidance in that smell's entry.

## Code smells

### Symmetric branches written as guard clause

**Trigger:** An `if` with an early `return` (or `continue`/`break`) followed by code at the same indentation level that handles the alternative case.

**When to revise:** When both branches represent peer alternatives of equal importance -- neither is an edge case or precondition. The structure should reflect that they are symmetric choices.

**When NOT to revise:** When the `if` truly guards against a precondition or error (e.g., input validation, null check). Guard clauses are appropriate for filtering out edge cases before the "real" logic begins.

**Fix:** Restructure as an explicit `if/else` so both branches have equal visual weight.

Before:
```python
if try_send(sock_path, tmp_path):
    # Existing GUI server accepted the request
    return

# No server running. Launch one.
subprocess.Popen(...)
```

After:
```python
if try_send(sock_path, tmp_path):
    # Existing GUI server accepted the request
    pass
else:
    # No GUI server running. Launch one.
    subprocess.Popen(...)
```

---

### Short CLI flags in subprocess calls

**Trigger:** Short flags like `-M`, `-w`, `-r` in `subprocess.run()` calls or command construction.

**Why:** Short flags are write-optimized. A reader encountering `-M` has to look up what it means. Long flags like `--find-renames` are self-documenting.

**When NOT to revise:** In user-facing CLI parsing (argparse definitions), where short flags are conventional for frequently-typed options. This pattern targets flags passed to *other programs* in subprocess calls.

**Fix:** Replace with the equivalent long flag. Also propagate the rename through any documentation that references the short flag.

Before:
```python
cmd = ["git", "diff", "-M"] + args
```

After:
```python
cmd = ["git", "diff", "--find-renames"] + args
```

---

### Em/en dashes in comments and text

**Trigger:** Em dashes (`--`) or en dashes (`-`) used in code comments, docstrings, or documentation to attach a trailing fragment to a sentence.

**Why:** Em/en dashes are a signature AI writing style. They feel literary, are hard to type on a physical keyboard, and signal AI-generated text. They also create run-on constructions that are harder to scan.

**Fix:** Use separate sentences (with a period) or a semicolon for closely related clauses. Avoid any kind of dash to separate trailing sentence fragments.

Before:
```python
# Stale socket file — remove it so the next launch starts fresh
```

After:
```python
# Stale socket file. Remove it so the next launch starts fresh.
```

---

### Blank line between adjacent strongly related blocks

**Trigger:** A blank line separating two code blocks that handle closely related concerns in the same logical flow (e.g., two sequential error checks on the result of the same operation).

**When to revise:** When the blocks are tightly coupled steps -- e.g., checking `FileNotFoundError` then checking `returncode`, both handling failures of the same `subprocess.run()` call.

**When NOT to revise:** When the blocks handle genuinely different phases of the function, or when a blank line aids readability by separating a long block from unrelated logic below.

**Fix:** Remove the blank line to visually group the related blocks.

---

### Functions ordered bottom-up (callees before callers)

**Trigger:** In a file, helper/callee functions are defined *above* the higher-level functions that call them. The reader encounters implementation details before understanding the big picture.

**Why:** AIs naturally write callees first (they need to know the helper's name to write the caller). But readers want the opposite: start at the high-level entry point, then drill down into details as needed. Files should read top-down like a newspaper article: headline first, details below.

**Fix:** Reorder so that high-level functions appear first, with callees defined below their callers. Calls should go "downward" in the file.

Before:
```python
def _build_title(args: list[str]) -> str:
    ...

def main() -> None:
    ...
    title = _build_title(args)
    ...

if __name__ == "__main__":
    main()
```

After:
```python
def main() -> None:
    ...
    title = _build_title(args)
    ...

def _build_title(args: list[str]) -> str:
    ...

if __name__ == "__main__":
    main()
```

**Exceptions:** Some definitions must stay at the top or bottom of a file due to technical constraints. For example, `if __name__ == "__main__":` must always remain at the bottom. Decorators must be defined before any function they decorate. Do not reorder past these constraints.

---

### Vague or generic names

**Trigger:** A function, variable, class, or file uses a name that describes *how* it works (mechanism) rather than *what* it represents (domain concept). Common AI patterns: `tmp_file`, `data`, `result`, `process_items`, `handle_request`.

**Why:** Vague names force readers to read the implementation to understand purpose. A precise name communicates intent immediately. This is often the simplest, lowest-risk revision available.

**Fix:** Rename to reflect the domain concept. Propagate the rename through all callers and documentation.

Before:
```python
def write_tmp_file(raw: bytes, title: str) -> Path: ...
def read_tmp_file(path: Path) -> tuple[bytes, str]: ...
```

After (minimal rename, without further restructuring):
```python
def write_gui_request_file(raw: bytes, title: str) -> Path: ...
def read_gui_request_file(path: Path) -> tuple[bytes, str]: ...
```

**Variant -- abbreviations in API surfaces:** Avoid abbreviations in function parameters, class fields, and other API-like surfaces (e.g., `ld_info` → `large_diff_info`). These names appear in call sites, documentation, and IDE tooltips where readers lack surrounding context to decode them. Abbreviations are more acceptable in local variables where scope is small and context is immediate.

**Variant -- name implies wrong type:** A name should let you guess its data type even without a type annotation. Watch for plural nouns or collective-sounding names used for scalar counts. `LARGE_BYTES` sounds like a collection of bytes; `LARGE_LINES` sounds like a list of line strings. Add a suffix like `_count` to make the type obvious.

Before:
```python
LARGE_BYTES = 1_048_576
LARGE_LINES = 10_000
```

After:
```python
_LARGE_DIFF_BYTE_COUNT = 1_048_576
_LARGE_DIFF_LINE_COUNT = 10_000
```

---

### Data clump passed through free functions

**Trigger:** Two or more standalone functions that accept or return the same bundle of parameters (e.g., `raw: bytes, title: str`).

**How to recognize:** A repeated parameter group appears across multiple function signatures. Supporting signals include tightly-coupled function pairs (read/write, serialize/deserialize), but the parameter bundle is the primary trigger.

**When NOT to revise:** When the functions are loosely related and share only one parameter. A single shared parameter is not a data clump.

**Fix:** Introduce a `@dataclass` that holds the shared parameters and convert the free functions into methods on that class.

Before:
```python
def write_gui_request_file(raw: bytes, title: str) -> Path:
    meta = json.dumps({"title": title})
    ...

def read_gui_request_file(path: Path) -> tuple[bytes, str]:
    ...
    return raw, meta.get("title", "gvc")
```

After:
```python
@dataclass(frozen=True)
class GuiRequest:
    title: str
    diff_bytes: bytes

    def write_to(self, filepath: Path) -> None: ...
    def write_to_temp_file(self) -> Path: ...

    @staticmethod
    def read_from(filepath: Path) -> GuiRequest: ...
```

---

### Conditionally-meaningful fields or parameters

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

**Variant -- union instead of optional:** When the conditionally-meaningful data represents a completely different input mode (not just "present or absent"), use a union type rather than an optional parameter. This forces callers to pass exactly one variant and forces the receiving function to handle each variant explicitly.

Before (part 1 refactoring left the optional parameter):
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

---

### Underscore-prefixed module names in applications

**Trigger:** Module files named `_foo.py` in a codebase that is an application (not a library).

**Why:** The underscore-prefix convention indicates "private module, not part of the public API." This is meaningful in libraries where some modules are internal implementation details. In applications, there is no public API to distinguish from, so the convention adds noise without communicating useful information.

**When NOT to revise:** In libraries or packages that expose a public API to external consumers.

**Fix:** Rename `_foo.py` to `foo.py`. Update all imports.

---

### Manual resource cleanup

**Trigger:** A resource (file, socket, connection, etc.) is opened/created in one place and cleaned up with an explicit `close()`, `unlink()`, or similar call later -- often at the end of a function or after a `try/except`.

**Why:** Manual cleanup steps are easy to skip on exception paths, leading to resource leaks. AI-drafted code frequently uses this pattern because it's straightforward to write linearly.

**Fix (in priority order):**
1. **Use the resource as a context manager** if it supports the `__enter__`/`__exit__` protocol (e.g., `with open(...) as f:`).
2. **Use `contextlib.closing()`** if the resource has a `close()` method but doesn't support the context manager protocol (e.g., `with closing(socket.socket(...)) as sock:`).
3. **Use `try/finally`** for cleanup that doesn't fit a context manager (e.g., `sock_path.unlink()`).

**When NOT to revise:** When the resource's creation and cleanup happen in different functions or different contexts (e.g., a resource created in one method and cleaned up in a callback). Structured cleanup only works when creation and cleanup are in the same scope.

Before:
```python
server_sock = socket.socket(socket.AF_UNIX, socket.SOCK_STREAM)
server_sock.bind(str(sock_path))
server_sock.listen(backlog=5)
...
webview.start()
server_sock.close()
sock_path.unlink(missing_ok=True)
```

After:
```python
with closing(socket.socket(socket.AF_UNIX, socket.SOCK_STREAM)) as server_sock:
    server_sock.bind(str(sock_path))
    try:
        server_sock.listen(backlog=5)
        ...
        webview.start()
    finally:
        sock_path.unlink(missing_ok=True)
```

---

### Parameter order mismatches visual/logical order

**Trigger:** Function parameters list data elements in a different order than they appear visually in the UI, or in a different order than the conceptual data model.

**Why:** When code parameters mirror visual or logical order, readers can map between code and UI/data intuitively. Mismatches create unnecessary cognitive friction.

**Fix:** Reorder parameters to match the visual top-to-bottom or logical order. For UI-related code, match the order elements appear on screen.

Before:
```python
def _open_window(raw: bytes, title: str, api, prefs_loader) -> None:
    # title appears above the diff in the UI, but comes second in params
```

After:
```python
def _open_window(title: str, diff_bytes: bytes, api: AppApi, prefs_loader: _PrefsLoader) -> None:
    # title first (top of window), then diff_bytes (window contents)
```

**Note:** This is not yet the most illustrative example. Look for better examples in future updates to this skill.

---

### Missing type annotations

**Trigger:** Function parameters or return types without type annotations, especially for non-obvious types like callback parameters, API objects, or protocol types.

**Why:** AI-drafted code frequently omits type annotations on parameters whose types aren't obvious from the name (e.g., `api`, `prefs_loader`, `callback`). Missing annotations force readers to trace through call sites to understand what a function expects.

**Fix:** Add type annotations. Use `TYPE_CHECKING` blocks for imports that are only needed by the type checker. Consider introducing type aliases for complex or repeated types (e.g., `type _PrefsLoader = Callable[[], Prefs]`).

Before:
```python
def _open_window(title: str, diff_bytes: bytes, api: AppApi, prefs_loader):
```

After:
```python
from typing import TYPE_CHECKING
if TYPE_CHECKING:
    from gvc.app_api import AppApi
    from gvc.prefs import Prefs
type _PrefsLoader = Callable[[], Prefs]

def _open_window(title: str, diff_bytes: bytes, api: AppApi, prefs_loader: _PrefsLoader) -> None:
```

---

### Missing `assert_never` on union dispatch

**Trigger:** An `isinstance` chain or `match` statement that dispatches on a union type and exhibits either of these antipatterns:

1. **Last variant handled by unchecked `else`:** The final `else` assumes the remaining type without an explicit `isinstance`/`case` check. New variants silently fall through.
2. **No `else` branch at all:** The chain handles all current variants but has no fallback. New variants silently do nothing.

Both are unsafe: adding a new variant to the union later won't produce a type error at dispatch sites.

**Fix:** Make every variant an explicit `isinstance` check (or `case`), then add a final `else: assert_never(x)` branch. Import `assert_never` from `typing`.

Before (antipattern 1 -- unchecked else):
```python
type Shape = Triangle | Square | Circle

shape: Shape = ...
if isinstance(shape, Triangle):
    ...
elif isinstance(shape, Square):
    ...
else:  # Circle -- unsafe: silently swallows new variants
    ...
```

Before (antipattern 2 -- no else):
```python
if isinstance(shape, Triangle):
    ...
elif isinstance(shape, Square):
    ...
elif isinstance(shape, Circle):
    ...
# no else -- unsafe: new variants silently do nothing
```

After:
```python
if isinstance(shape, Triangle):
    ...
elif isinstance(shape, Square):
    ...
elif isinstance(shape, Circle):
    ...
else:
    assert_never(shape)
```

---

### `# type: ignore` and `cast(...)` in typed code

**Trigger:** Any use of `# type: ignore` or `cast(...)` in Python code.

**Why:** These are type-safety escape hatches that suppress real errors. AI-drafted code reaches for them quickly rather than fixing the underlying type issue. They can mask bugs that the type checker would otherwise catch.

**Fix:**
1. Temporarily remove the `# type: ignore` or `cast(...)` to see the original typechecker error it was suppressing.
2. Use `/fix-typecheck-errors` for strategies to fix the underlying type issue (e.g., `isinstance()` narrowing, assertions, protocol types, restructuring).

---

### Magic numbers

**Trigger:** Numeric literals used directly in logic (e.g., `max(8, min(32, size))`), where the meaning of the number isn't obvious from context.

**Why:** Magic numbers force readers to guess what the value represents. A named constant communicates intent immediately.

**Fix:** Extract to a named constant. Scope the constant to its smallest relevant context: class-level constant if only one class uses it, module-level if used across functions.

Before:
```python
def set_font_size(self, size: int) -> None:
    size = max(8, min(32, int(size)))
```

After:
```python
class AppApi:
    _MIN_FONT_SIZE = 8
    _MAX_FONT_SIZE = 32
    ...
    def set_font_size(self, size: int) -> None:
        size = max(self._MIN_FONT_SIZE, min(self._MAX_FONT_SIZE, int(size)))
```

---

### if/else assigning same variable → conditional expression

**Trigger:** An `if/else` block where both branches assign to the same variable and the expressions are short enough to read inline.

**Why:** The if/else form obscures that the block's sole purpose is to produce a single value. A conditional expression makes the intent explicit: "compute this value."

**When NOT to revise:** When the branch expressions are complex, multi-line, or have side effects. If the conditional expression would exceed a comfortable line length or require nested ternaries, keep the if/else.

**Fix:** Convert to a conditional expression.

Before:
```python
if large_diff_info is not None:
    html_doc = render(large_diff_info)
else:
    html_doc = render(parse(diff_bytes))
```

After:
```python
html_doc = (
    render(large_diff_info)
    if large_diff_info is not None
    else render(parse(diff_bytes))
)
```

---

### Temporary variable that adds no explanatory value

**Trigger:** A variable is assigned once, used once on the very next line (or nearby), and its name doesn't clarify intent beyond what the expression already communicates.

**Why:** The variable introduces a name that the reader must track mentally without adding information. Inlining removes the indirection.

**When NOT to revise:** When the variable name *explains* an otherwise obscure expression. An "explaining variable" earns its existence by giving a readable name to something that would be hard to understand inline. The test: does the variable name tell you something the expression doesn't?

**Fix:** Inline the expression at its single use site.

Before:
```python
prefs = Prefs.load()
create_window(html_doc, title, prefs, api)
```

After:
```python
create_window(html_doc, title, Prefs.load(), api)
```

---

### Loose Notes - not yet elaborated

- **Local imports in main() entry points:** In files that serve as the first module loaded when Python starts (containing a `main()` function), it is *conventional* to keep project-specific (non-stdlib) imports local to `main()` or similar functions. This minimizes startup time for commands like `--help` that don't need the full module graph. Do not move these imports to the top of the file.
- **Local imports in non-entry-point modules** should be moved to the top of the file, per standard Python convention.
- **British vs. American English:** Prefer American English spelling in code comments and documentation (e.g., `behavior` not `behaviour`).
- **Speculative generality / unnecessary indirection:** AI-drafted code may sometimes add abstraction layers "for flexibility" or "for testability" that have only one concrete usage and no tests. Examples: callback parameters always passed the same callable. Inline the value and remove the indirection.
- **`try_X` naming for failable operations:** When an operation returns `None` on failure rather than raising an exception, prefer the `try_` prefix (e.g., `try_parse`, `try_send`). This signals to readers that a `None` return is expected and normal, not an error.
