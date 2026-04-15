---
name: revise
description: Perform common code quality revisions on AI-drafted code. Use when reviewing recently drafted code for readability and maintainability improvements. Use when asked to learn new revise patterns from a commit or diff.
disable-model-invocation: true
argument-hint: commit SHA or "git diff" command to revise (or learn revision patterns from)
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

3. **Pass 1 -- Immediate fixes.** Review each file in order. For each code smell found:
   - **Apply immediately** if the fix is small and localized (rename, inline, add comment, extract helper, etc.).
   - **Defer** if the fix would cause large diff traffic or is more like a new feature than a small revision. Record deferred fixes per file. Common reasons to defer:
     - **High diff traffic:** Reordering top-level sections/functions, changing indentation style. These touch many lines, easily conflict with smaller changes happening in parallel.
     - **Non-trivial new work:** Changing build systems, configuring new tooling (linters, typecheckers), adding assets. These are features, not revisions.

4. **Review point 1.** Present the immediate fixes for review before proceeding. Also present the list of unapplied deferred fixes for confirmation.

5. **Pass 2 -- Deferred fixes.** Apply the recorded deferred fixes, one file at a time.

6. **Review point 2.** Present the deferred fixes for review.

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

An `mcp__revise__rename_symbol` tool is available for quickly renaming functions, variables, modules, and other symbols. Details: <reference/rename_symbol.md>

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

- **[Obvious/redundant docstrings](patterns/obvious_docstrings.md)** -- Docstring restates what the function/test/module name already says. Especially common in AI-drafted test functions.
  ```python
  def test_user_can_log_in():
      """Test that a user can log in."""  # adds nothing
  ```

- **[Unprefixed comments should describe the next code](patterns/unprefixed_comment_scope.md)** -- Unprefixed comments label *what* the immediately following code does. General commentary belongs under `NOTE:` or in documentation.
  ```python
  # Internal dispatch: the bundled executable is dual-purpose. When another
  # gvc process invokes it with `--gui-server <request_file>` (see below),
  # act as the persistent GUI server instead of the CLI.
  if args and args[0] == "--gui-server":
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

- **[Docstring verb form](patterns/docstring_verb_form.md)** -- Docstring starts with an imperative verb. Prefer third-person indicative.
  ```python
  def main() -> None:
      """Build ./dist/gvc.app via PyInstaller."""  # -> "Builds ..."
  ```

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

- **[Bare `dict` at serialization boundaries](patterns/bare_dict_at_boundary.md)** -- Function returns `dict` where data crosses systems (Python→JS, module→module). Use `TypedDict` to make the contract explicit.
  ```python
  def get_prefs(self) -> dict:    # shape is implicit
      return {"font_size": ...}
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

## How to learn new patterns from a commit

Invoked when the user says something like "Learn any new revise patterns from commit `<SHA>`" or "Review the revisions I made to AI-drafted code in commit `<SHA>`".

The user has manually revised code that an AI previously drafted, and captured those revisions in a single commit. The goal is to extract *generalizable* patterns from that commit and codify them into this skill, so future `/revise` runs catch the same issues automatically.

### Procedure

1. **Examine the commit's diff.**
   - Run `git show -U10 -w <SHA>`.
     - `-U10` gives enough context to see what's going on without being too noisy.
     - `-w` ignores whitespace changes, which makes indentation-only revisions far less noisy.
     - `show` (vs. `diff`) includes the commit *message*, which often summarizes the user's intent.
   - If `<SHA>` is omitted, default to `HEAD`.

2. **Interview the user.** Use the `AskUserQuestion` tool. Ask about:
   - **Why specific revisions were made**, when the rationale isn't obvious from the diff alone. Surface-level changes often hide a deeper principle worth codifying.
   - **Why expected revisions were NOT made.** Scan the pre-revision code for patterns already in this skill -- if a pattern *would* have triggered but the user left the code alone, that's a boundary condition worth capturing. Don't only learn from changes; learn from the user's deliberate non-changes too.
   - **Where a revision fits** among existing patterns -- is it a new pattern, a refinement of an existing one, or an inverse/complementary case?

   Keep questions focused and batched. Prefer a few well-chosen questions over an exhaustive interrogation.

3. **Draft an update to the skill.** For each generalizable pattern learned:
   - If the pattern is genuinely new: add a bullet under the appropriate category in SKILL.md, and create a detail file in `patterns/<name>.md`.
   - If the pattern refines or qualifies an existing one: update the existing detail file (add a "When NOT to revise" entry, an inverse case, or a clarifying example) rather than creating a new one.
   - Follow the structure of existing pattern files: **Quick trigger**, **Why revise**, **When NOT to revise**, **Fix**, **Example (Before/After)**.
   - Capture the user's *rationale* in the pattern file, not just the mechanical rule. The "why" lets future-me judge edge cases.

4. **Reevaluate skill organization.** After drafting updates, check whether the skill still reads well at a high level. Reorganize if needed -- see [organization-guidelines.md](organization-guidelines.md) for the principles (7-10 items per category, progressive disclosure, categories close to recognizable situations).

### What counts as a generalizable pattern

- **Generalizable:** applies across files, languages, or projects -- e.g., "docstrings should state API guarantees, not implementation details."
- **Not generalizable:** one-off cleanups tied to this specific codebase's history, tooling, or content -- e.g., "switched the build system from Hatch to Poetry," "moved the paused PyInstaller packaging files to a side branch," or "added a screenshot to README." These are project decisions, not code-quality patterns. They belong in the commit message, not the skill.

When in doubt, ask: *would this same revision make sense in a different codebase written by a different team?* If yes, it's a pattern.

---

*See [organization-guidelines.md](organization-guidelines.md) for the principles governing how this skill is structured.*
