# Sections grouped by kind instead of by feature/concern

## Quick trigger

Section headings that describe what items ARE rather than what they DO or what feature they serve. Common triggers:

- **"Public API"** or **"Public"** -- Separating public from private is rarely a useful organizing principle. Almost all code mixes public and private definitions within the same concern.
- **"Constants"** -- Borderline acceptable (constants often float to the top), but a formal "Constants" section suggests items serving different features are lumped together. Constants typically live in the unnamed section at the top of a file, not in a labeled section.
- **"Data Model"** or **"Data Types"** -- Vague/unvisualizable section name. What *kind* of data? What feature does it model?

## Why revise

Grouping by kind forces the reader to jump between sections to understand a single feature. A "Constants" section with 10 items serving 3 different features gives no hint which constants relate to which feature. Grouping by concern keeps everything about a feature together: its types, constants, and functions.

## When NOT to revise

- When the file genuinely has one concern, and the kind-based sections are small. A file with 3 constants in a "Constants" section and 2 functions is fine.
- When constants are truly shared across all features in the file. A handful of shared constants at the top of the file (without a section heading) is conventional.

## Modules and packages by kind CAN be acceptable

This pattern applies to *sections within a file*. At the next level up -- modules (files) and packages (directories) -- grouping by kind is sometimes appropriate:

- **Technical constraints** may force separation: HTML (structure) vs. CSS (presentation), backend vs. frontend code.
- **Layered architectures** sometimes split "models" and "views/controllers" into separate files. But note that these layers are usually themselves grouped by feature at a higher level:

```
apps/
    login/
        model.py       # Custom user model
        views.py       # Login-related pages
    planner/
        model.py       # Calendar-related models
        views.py       # Day/week/month views
    assessment/
        model.py       # Assessment-related models
        views.py       # Assessment question & answer views
```

Each feature package groups by concern (`login/`, `planner/`), while within each package the files are split by kind (`model.py`, `views.py`) due to framework conventions.

## Fix

1. Identify the features/concerns in the file.
2. Create section headings named for the concern (e.g., "Large Diffs", "Parse Diff" rather than "Constants", "Public API").
3. Move each constant, type, and function to the section for the concern it serves.
4. Constants that are truly shared across concerns can remain in an unnamed block at the top of the file (below imports).

## Example

Before -- grouped by kind:

```python
# --- Data Model ---
LineKind = Literal["context", "added", "removed"]

@dataclass
class LargeDiffInfo:
    byte_count: int
    line_count: int

# --- Constants ---
_DIFF_HEADER = re.compile(r"^diff --git ...")
_LARGE_DIFF_BYTE_COUNT = 1_048_576
_LARGE_DIFF_LINE_COUNT = 10_000

# --- Public API ---
def parse(diff_bytes: bytes) -> list[FileDiff]: ...
```

After -- grouped by concern:

```python
# --- Data Model ---
LineKind = Literal["context", "added", "removed"]

# --- Large Diffs ---
_LARGE_DIFF_BYTE_COUNT = 1_048_576
_LARGE_DIFF_LINE_COUNT = 10_000

@dataclass(frozen=True)
class LargeDiffInfo:
    byte_count: int
    line_count: int

# --- Parse Diff ---
_DIFF_HEADER = re.compile(r"^diff --git ...")

def parse(diff_bytes: bytes) -> list[FileDiff]: ...
```
