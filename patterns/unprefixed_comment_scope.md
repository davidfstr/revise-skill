# Unprefixed comments should describe the next code

## Quick trigger

An unprefixed comment (one that does not start with `NOTE:`, `TODO:`, `FIXME:`, etc.) contains general commentary, background, or rationale rather than a description of the code that immediately follows it.

Common shapes:

- A multi-paragraph comment where only the first paragraph describes the next statement; the rest is context or history.
- A comment explaining *why* something is structured a certain way, with no indication of *what* the next line does.
- A comment that reads like prose documentation ("this module does X because Y...") attached to a line of code.

## Why revise

Unprefixed comments earn their keep by labeling the code that follows, so a reader scanning down the file can skip or descend at each label. When an unprefixed comment instead carries general commentary, the reader cannot tell whether it's describing the next line, the next paragraph, or the whole function -- and has to read it all to find out.

Splitting the roles makes the file scannable:

- **Unprefixed comment** = "the next code does X." Short, scannable, always local.
- **`NOTE:` comment** = general commentary, constraint, caveat, or cross-reference that a maintainer needs but which isn't a label for the next line.
- **Documentation** = broader rationale that belongs in `doc/`, module docstrings, or commit messages.

## When NOT to revise

- When the unprefixed comment is already a concise description of the next statement or block.
- When the commentary is short (one line) and genuinely inseparable from the "what" -- e.g., `# Skip header row` doesn't need a `NOTE:` prefix.
- When adding a `NOTE:` prefix would make a simple label feel bureaucratic.

## Fix

Triage each non-local sentence in the comment:

1. **Describes the next code?** Keep it, unprefixed. Tighten if possible.
2. **General commentary a maintainer needs?** Prefix with `NOTE:` (and keep it near the relevant code).
3. **Broader context or rationale?** Move to module docstring, `doc/`, or the commit message. Delete from the code.

When the label describes an *action* the next code performs, prefer an imperative verb opener ("Add enough lines so that...", "Ensure renames are detected...", "Give the candidate enough content...") over a bare noun-phrase fragment ("Enough lines that...", "Content for the candidate..."). The verb nails down the relationship between label and code; a noun phrase leaves it ambiguous whether the comment is describing the *state* of something or the *action* being taken.

> This is the inverse of the docstring convention (third-person indicative). Docstrings describe what a function *is*; unprefixed action-comments describe what the next lines *do*, and read most clearly as imperatives.

## Example

Before -- unprefixed comment mixes a label with context a reader doesn't need at this line:

```python
# Internal dispatch: the bundled executable is dual-purpose. When another
# gvc process invokes it with `--gui-server <request_file>` (see below),
# act as the persistent GUI server instead of the CLI.
if args and args[0] == "--gui-server":
    ...
```

After -- the comment now labels only what the next code does:

```python
# If invoked as `--gui-server <request_file>` by another gvc process,
# act as the persistent GUI server instead of as the CLI.
if args and args[0] == "--gui-server":
    ...
```

Before -- a long rationale that buries the label ("Launch one") under implementation history:

```python
# No GUI server running. Launch one.
#
# If this cli.py itself lives inside a distributed .app bundle, launch
# the bundled executable so LaunchServices picks up its Info.plist
# (proper Dock tooltip, menu bar name, etc.).
#
# Otherwise (dev iteration via `poetry run gvc`) fall back to an
# unbundled `python -m gvc.gui`. The Dock tooltip will say "Python"
# but everything else is patched at runtime -- see gui.py.
bundle_exe = _enclosing_app_executable()
...
```

After -- label describes the branches; the "why" moved to docs:

```python
# No GUI server running. Launch one:
# - If this cli.py itself lives inside a distributed .app bundle, launch it.
# - Otherwise fall back to an unbundled `python -m gvc.gui`.
bundle_exe = _enclosing_app_executable()
...
```
