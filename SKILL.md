---
name: revise
description: Perform common code quality revisions on AI-drafted code. Use when reviewing recently drafted code for readability and maintainability improvements.
disable-model-invocation: true
argument-hint: "[diff-spec]"
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

4. **Apply revisions** for each code smell found, following the guidance in that smell's detail file.

## Code smells

### Organization

- **[Local imports in non-entrypoint modules](patterns/local_imports.md)** -- Project imports placed inside functions instead of at the top of the file.
  ```python
  def _open_window(title, diff_bytes, api):
      from gvc.renderer import render  # should be at top of file
  ```

- **[Functions ordered bottom-up](patterns/functions_ordered_bottom_up.md)** -- Callees defined above callers; reader hits details before the big picture.
  ```python
  def _build_title(args): ...    # helper defined first
  def main(): ...                # entry point buried below
  ```

- **[Headings to group definitions](patterns/headings_to_group.md)** -- A file has many top-level definitions (>7-10) with no visual grouping.

- **[Sections grouped by kind instead of feature/concern](patterns/sections_by_kind.md)** -- Headings like "Constants", "Data Model", "Public API" lump items by what they are, not what feature they serve.
  ```python
  # --- Constants ---       # serves 3 different features
  # --- Public API ---      # public vs. private is rarely useful
  ```

- **[Parameter order mismatches](patterns/parameter_order.md)** -- Parameters (or declarations) listed in a different order than their visual/logical order.
  ```python
  def _open_window(raw: bytes, title: str, ...):  # title appears first in UI
  ```

- **[Blank lines between related blocks](patterns/blank_lines_related_blocks.md)** -- A blank line separates two tightly coupled blocks (e.g., sequential error checks on the same operation).

- **[Symmetric operations split across modules](patterns/symmetric_operations_colocated.md)** -- Send/receive, read/write, encode/decode pairs live in different modules instead of adjacent in the same one.
  ```python
  # _ipc.py has try_send(), but _gui.py has the receive logic inline
  ```

- **[Symmetric branches as guard clause](patterns/symmetric_branches.md)** -- An `if/return` followed by the alternative case at the same level, when both branches are peer alternatives.
  ```python
  if try_send(sock_path, tmp_path):
      return
  subprocess.Popen(...)  # looks like main path, but is actually the alternative
  ```

### Good Names

- **[Vague or generic names](patterns/vague_generic_names.md)** -- Name describes mechanism (`tmp_file`, `data`, `result`) rather than domain concept.
  ```python
  def write_tmp_file(raw: bytes, title: str) -> Path: ...
  ```

- **[Name implies wrong type](patterns/name_implies_wrong_type.md)** -- Name suggests a collection or different type than what it actually holds.
  ```python
  LARGE_BYTES = 1_048_576   # sounds like a list of bytes, actually a count
  ```

- **[Abbreviations in API surfaces](patterns/abbreviations_in_api.md)** -- Abbreviated names in function parameters, class fields, or other API-like surfaces.
  ```python
  def render(fd: list[FileDiff], ld_info: LargeDiffInfo | None = None): ...
  ```

- **[`try_X` naming for failable operations](patterns/try_x_naming.md)** -- An operation that returns `None` on failure lacks the `try_` prefix to signal that.

### Clarity / Anti-Obscurity

- **[Magic numbers](patterns/magic_numbers.md)** -- Numeric literals used directly in logic where the meaning isn't obvious.
  ```python
  size = max(8, min(32, int(size)))
  ```

- **[Short CLI flags in subprocess calls](patterns/short_cli_flags.md)** -- Short flags like `-M`, `-w` in subprocess calls instead of self-documenting long flags.
  ```python
  cmd = ["git", "diff", "-M"] + args
  ```

- **[Docstring with implementation details](patterns/docstring_with_impl_details.md)** -- Docstring contains implementation details (why a workaround, why a specific library). Docstrings are for API guarantees; implementation rationale goes in `# NOTE:` comments.
  ```python
  def _is_dark_mode() -> bool:
      """... Uses NSUserDefaults rather than ..."""  # implementation detail, not API
  ```

- **[Clarifying comments on non-obvious code](patterns/clarifying_comments.md)** -- A code block responds to a situation non-obviously (e.g., silently swallowing errors), or a paragraph is 5-7+ lines with no label.
  ```python
  except (json.JSONDecodeError, TypeError):
      return cls()  # why? intentional? looks like a bug
  ```

### Formatting & Style

- **[Em/en dashes in comments and text](patterns/em_en_dashes.md)** -- Dashes used to attach trailing fragments to sentences. Signature AI writing style.
  ```python
  # Stale socket file — remove it so the next launch starts fresh
  ```

- **[British vs. American English](patterns/british_vs_american.md)** -- British spelling in comments/docs (e.g., `behaviour`, `initialise`).

### Concision

- **[Duplicate code](patterns/duplicate_code.md)** -- Two+ blocks doing substantially the same thing, either >3 lines or far apart. Copies will eventually diverge.
  ```python
  # _render_file: icon default "⬜"    — already diverged from:
  # _render_outline: icon default "📄"
  ```

- **[Temporary variable with no explanatory value](patterns/temporary_variable.md)** -- Variable assigned once, used once, name adds nothing the expression doesn't already say.
  ```python
  prefs = Prefs.load()
  create_window(html_doc, title, prefs, api)
  ```

- **[if/else → conditional expression](patterns/conditional_expression.md)** -- Both branches of an if/else assign to the same variable; expressions are short.
  ```python
  if large_diff_info is not None:
      html_doc = render(large_diff_info)
  else:
      html_doc = render(parse(diff_bytes))
  ```

- **[Dead code](patterns/dead_code.md)** -- Unused constants, unreachable branches, unused literal members, commented-out blocks. Misleads readers into thinking it matters.
  ```python
  _OLD_FILE = re.compile(r"^--- (?:a/)?(.+)$")   # never referenced
  ```

- **[Hand-rolled caching](patterns/hand_rolled_caching.md)** -- Module-level sentinels (`_FOO: str | None = None`) with `if _FOO is None` guard. Replace with `@cache`.
  ```python
  _CSS: str | None = None
  def _assets():
      global _CSS
      if _CSS is None: ...
  ```

### Type Design

- **[Data clump → dataclass](patterns/data_clump.md)** -- Multiple functions share the same parameter bundle.
  ```python
  def write_gui_request_file(raw: bytes, title: str) -> Path: ...
  def read_gui_request_file(path: Path) -> tuple[bytes, str]: ...
  ```

- **[Conditionally-meaningful fields](patterns/conditionally_meaningful_fields.md)** -- Fields/params only meaningful when some condition holds (boolean flag, status literal).
  ```python
  @dataclass
  class FileDiff:
      status: Literal["added", "deleted", "modified", "large"]
      raw_size: int = 0    # only meaningful when status == "large"
      raw_lines: int = 0
  ```

### Type Safety

- **[`assert_never` on union dispatch](patterns/assert_never.md)** -- `isinstance` chain or `match` on a union type without exhaustive fallback.
  ```python
  if isinstance(shape, Triangle): ...
  elif isinstance(shape, Square): ...
  else:  # Circle -- unsafe: silently swallows new variants
  ```

- **[`type: ignore` and `cast(...)` in typed code](patterns/type_ignore_cast.md)** -- Type-safety escape hatches that suppress real errors.

- **[Missing type annotations](patterns/missing_type_annotations.md)** -- Untyped parameters, especially non-obvious types like callbacks or API objects.
  ```python
  def _open_window(title: str, diff_bytes: bytes, api: AppApi, prefs_loader):
  ```

### Uncategorized (elaborated)

- **[Manual resource cleanup](patterns/manual_resource_cleanup.md)** -- Explicit `close()`/`unlink()` instead of context managers or `try/finally`.
  ```python
  server_sock = socket.socket(...)
  ...
  server_sock.close()  # skipped on exception
  ```

### Loose patterns (not yet elaborated)

- **Speculative generality / unnecessary indirection:** Abstraction layers with only one concrete usage and no tests. Examples: callback parameters always passed the same callable. Inline the value and remove the indirection.
- **Underscore-prefixed module names in applications:** `_foo.py` in applications where there is no public API to distinguish from. Rename to `foo.py`.
- **Verbose construction where simpler equivalent exists:** `{f for f in x}` → `set(x)`, `[x for x in items]` → `list(items)`, `{k: v for k, v in d.items()}` → `dict(d)`. Use the built-in constructor when the comprehension adds no filtering or transformation.
- **Redundant safety mechanisms:** When two mechanisms provide the same guarantee, remove the more complex one. Example: `fcntl.flock` + `os.replace` -- the atomic replace already ensures last-writer-wins, making the lock unnecessary.

### Loose categories

Categories that may emerge as more patterns are discovered:

- **Define errors out of existence** -- similar to Type Design
- **Cohesion**
- **Correctness**

---

*See [organization-guidelines.md](organization-guidelines.md) for the principles governing how this skill is structured.*
