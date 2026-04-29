---
name: update-readme
description: Refresh the Revise Skill's README.md and its downstream marketing page (at dafoster.net) so they reflect pattern, category, and structure changes since the last major content update to those artifacts. Use when the user says the README is stale, asks to bring the README up to date, asks to sync the README with SKILL.md, or asks to refresh the marketing page after a stretch of pattern work.

---

# Update README

Bring the Revise Skill's user-facing artifacts back in sync with its authoritative content after a stretch of accumulated changes.

The two artifacts this skill maintains:

1. **README.md** at repo root — `./README.md` (`https://github.com/davidfstr/revise-skill/README.md`)
2. **Marketing page** — `https://github.com/davidfstr/dafoster.net/projects/revise-skill/index.md`
    - To locate the absolute path of the local clone on this computer, consult local `/locate-github-repo-url-clone` skill, user memory (`~/.claude/CLAUDE.md`), or ask the user.

The authoritative content lives in `SKILL.md` (categories, pattern catalog, procedure) and the top-level directory layout (`patterns/`, `reference/`, etc.). When patterns and categories shift over a stretch of commits, both artifacts drift and need to be refreshed.

The README is upstream of the marketing page. Update the README first, then mirror the changes downstream.

## Procedure

### 1. Find the floor — last major content update to the artifact

Look at the artifact's commit history to establish what counts as "since":

```
git log --oneline -- README.md
```

A *major* content update rewrites or substantially restructures content (e.g., the original `Add README`, a category overhaul). A *minor* one fixes a logo URL, extends a sentence, or moves a comment. Pick the most recent major update as the floor.

For the marketing page, the user often knows the floor offhand — e.g., "it was based on the README before the last 2 commits." Otherwise, compare the marketing page's category table against `git log -- README.md` to find when those categories last appeared in the README.

### 2. Survey what's changed since the floor

```
git log --oneline <floor-sha>..HEAD
```

Read the commit messages. Group by theme — new patterns, renames, category splits, new top-level files/directories, new sections in SKILL.md. For commits whose subjects don't make the change clear, run `git show <sha>` to inspect.

### 3. Compare the artifact against current authoritative content

Read the artifact (README or index.md) alongside SKILL.md. The recurring forms of staleness:

- **Categories table** out of sync with SKILL.md: missing categories, renamed categories, split categories, or pattern lists that haven't been refreshed since new patterns landed.
- **Directory tree** in the Structure section missing top-level directories that now exist.
- **Features** documented in SKILL.md but not surfaced in the artifact (or vice versa — features the artifact still mentions that have been removed).
- **Anchor links** that point to sections that no longer exist.

For the marketing page specifically, also confirm that link styles still resolve — the marketing page can't relative-link into the GitHub repo, so it uses absolute URLs.

### 4. Triage: apply-directly vs. ask-the-user

Sort each candidate change into one of two buckets.

**Apply directly** — factual fixes where the artifact is incorrect, not just incomplete:

- Renamed categories (e.g., `Clarity` → `Clarity / Anti-Obscurity`)
- Split categories (e.g., `Organization` → `Organization (file-level)` + `Organization (within-function)`)
- New categories that have never been listed
- New top-level directories absent from the Structure section
- Broken anchor links

**Ask the user** — judgment calls about voice, scope, or curation:

- Which example patterns to list under each category, and how many. If the table format has historically been "all patterns", err toward listing all. If it's been "curated examples", curation is a deliberate editorial choice the user should weigh in on.
- Whether to add entirely new sections describing newer features (e.g., MCP companion, pattern-learning workflow).
- Wording of new sections.
- Whether a change is worth surfacing in the artifact at all (versus living only in SKILL.md).

Apply the first bucket directly. Summarize the second bucket and ask before drafting.

### 5. Apply approved judgment-call changes

Use the user's answers to draft and apply the remaining edits. When adding a new section, place it where the surrounding flow suggests — an installation-related companion section near Installation, a "how patterns are added" section near the sentence that motivates linking to it, etc.

### 6. Mirror to the marketing page

After the README is settled, propagate the same changes to the marketing page. Adapt to its idiom rather than blindly copying:

- **Line wrapping**: the marketing page wraps prose at ~70 chars. Wrap new prose accordingly. Markdown table rows stay on one line either way.
- **Link style**: convert relative repo links (e.g., `SKILL.md#anchor`) to absolute GitHub URLs (e.g., `https://github.com/davidfstr/revise-skill/blob/main/SKILL.md#anchor`). Internal anchor links (e.g., `#how-patterns-are-added`) stay as-is.
- **Section coverage**: the marketing page omits some sections the README has — most notably `Structure` and `Model codebases`. Don't add those when mirroring; respect what the marketing page chose to keep.
- **Frontmatter**: the marketing page has Jekyll-style frontmatter at the top. Leave it alone.

Before mirroring, re-diff the README against its pre-edit state:

```
git diff <pre-edit-sha>..HEAD -- README.md
```

Between turns the user may make small changes to the README directly. Re-diffing catches those, so the marketing page doesn't reintroduce older wording.

## Notes

- If the user asks to update only one artifact, do just that. Otherwise default to README first, then offer to mirror.
- The triage step is the most important one. Reporting "here's what I'd apply directly, here's what I'd like your input on" lets the user catch wrong calls before they propagate to the marketing page.
- This skill is scoped to the README ↔ SKILL.md ↔ marketing page triangle. It's not a general doc-sync task.
