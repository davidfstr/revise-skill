# Renaming symbols with `mcp__revise__rename_symbol`

A tool for renaming functions, variables, classes, modules, and other symbols across a workspace. Delegates to the language server's real rename refactoring (via a VSCode extension), so it follows symbol references — not text matches.

**Use it whenever** you're amending a vague or unclear name. Symbol renames are one of the most common revisions, and this tool makes them near-free.

## How to use

Anchor the rename with any line of code that references the symbol. Pass the line in `oldString`/`newString` form, with only the symbol text differing:

```
filePath:   "/path/to/somefile.py"
oldString:  "def compute_total("
newString:  "def compute_order_total("
```

The tool will rename `compute_total` → `compute_order_total` at its definition and every reference across the workspace.

**Module renames:** Target an import statement that mentions the module. The tool will rename the module identifier, update every import, AND rename the `.py` file on disk.

```
filePath:   "src/gvc/_gui.py"
oldString:  "from gvc._ipc import gui_socket_path"
newString:  "from gvc.ipc import gui_socket_path"
```

## Limitations

**Does not touch comments, string literals, or documentation.** The language server only sees symbol references, so any occurrence inside:

- `"""docstrings"""` and `# comments`
- String literals like `subprocess.Popen([..., "gvc._gui", ...])`
- Markdown files, `.txt` docs, ASCII-art diagrams

...will be left unchanged, with no warning. The tool does not currently report potential matches it skipped.

**Mitigation:** For *public* symbols likely to have widespread references (public functions, public class fields, public module names), follow up with a global grep for both the old and new names to find any references the language server missed. Private/underscore-prefixed symbols are usually safe without this step.

**Fails cleanly on non-symbols.** If the anchor line is itself a string literal (no real symbol at that position), the tool returns `"The element can't be renamed."` — fall back to a manual rename (file `mv` + `Edit` passes).

**Requires the VSCode MCP extension to be running** and reachable at `localhost:4000/mcp`. If the tool returns a 404 or connection error, check that VSCode is open on the workspace and the extension is active. Another process squatting on port 4000 (e.g. a local dev server) will also cause this.

## Verify after renaming

After a rename that touches multiple files, run the project typechecker (e.g. `mypy`) to catch any references the LSP rename missed or broke. A clean typecheck is the simplest proof that the rename was complete and consistent.

If the project has no typechecker, a global grep for the old name across both source and documentation is the next-best check.
