---
name: sync
description: Rebuild all index files in the PM workspace by scanning note frontmatter. Triggers on "sync", "rebuild indexes", "update indexes", "refresh indexes", or after editing any note.
allowed-tools: Read, Write, Glob, Grep, Bash
---

Rebuild aggregated index files in the PM workspace. Runs incrementally by default — only reads/writes what changed since the last sync. Pass `--force` to do a full rebuild.

WORKSPACE: /Users/shubhamchavan/Desktop/programs/work
MANIFEST: /Users/shubhamchavan/Desktop/programs/work/.sync-manifest.json
NOW: !`date +%Y-%m-%dT%H:%M`
TODAY: !`date +%Y-%m-%d`

---

## Phase 0 — Detect what changed

### 0a. Check for manifest

Run: `test -f "$MANIFEST" && echo exists || echo missing`

**If manifest is missing OR `--force` was passed** → set `full_rebuild = true`, skip to Phase 0c.

**If manifest exists and no `--force`** → proceed to 0b.

### 0b. Find changed/new/deleted files (incremental mode)

1. Find files modified after the manifest:
   ```
   find "$WORKSPACE/daily" "$WORKSPACE/meetings" "$WORKSPACE/projects" "$WORKSPACE/people" \
     -name "*.md" -newer "$MANIFEST" 2>/dev/null
   ```
   Store result as `changed_files` (list of absolute paths).

2. Find all current note files:
   ```
   find "$WORKSPACE/daily" "$WORKSPACE/meetings" "$WORKSPACE/projects" "$WORKSPACE/people" \
     -name "*.md" 2>/dev/null
   ```
   Store as `current_files`.

3. Read the manifest JSON. Extract the `files` object — its keys are relative paths like `meetings/2026-04-06-foo.md`. Convert to absolute paths → `manifest_files`.

4. `deleted_files` = paths in `manifest_files` but not in `current_files`.

5. **If `changed_files` is empty AND `deleted_files` is empty** → print:
   ```
   /sync — already up to date (nothing changed since last sync)
   Run `/sync --force` for a full rebuild.
   ```
   Stop here.

### 0c. In full rebuild mode

Set `changed_files` = all files in `daily/`, `meetings/`, `projects/`, `people/`.
Set `deleted_files` = [].
Set `manifest_data` = `{ "last_sync": "", "files": {} }` (empty).
Skip to Phase 1.

---

## Phase 1 — Load manifest cache + parse changed files

### 1a. Load manifest cache

If not in full rebuild mode, read manifest JSON. Load `manifest_data.files` — this is a dictionary keyed by relative path, with cached frontmatter metadata for every previously-synced file.

For **unchanged** files (in `current_files` but not in `changed_files` and not deleted): use their cached entry from `manifest_data.files` directly — **do not read these files from disk**.

### 1b. Parse changed/new files

For each file in `changed_files`: read the file from disk. Parse YAML frontmatter (between first `---` pair). Extract:
- `type`, `title`, `date`, `slug` (if present)
- `projects` (list of slugs, default [])
- `people` (list of slugs, default [])
- `status` (for project notes)
- `action_items` list of `{owner, task, due, status}` (for meeting notes)
- `resolves` list of `{source_meeting, owner, task, outcome, note}` (for meeting notes)
- `decisions` list of strings (for meeting notes)
- `role`, `team` (for person notes)

Store each parsed file's metadata as `parsed[relative_path]`.

---

## Phase 2 — Determine dirty indexes

Using `parsed` (for changed/new files) and manifest cache (for unchanged files), plus `deleted_files` metadata from manifest cache:

Build a unified view of ALL files: for each file, its `{type, date, projects, people, has_action_items}` — from `parsed` if changed/new, from manifest cache otherwise.

**Determine which indexes need rebuilding:**

- `dirty_projects` (set of project slugs): union of `projects` from all changed/new/deleted files
- `dirty_people` (set of person slugs): union of `people` from all changed/new/deleted files
- `dirty_dates` (set of date strings): union of `date` from all changed/new/deleted files
- `actions_dirty` (boolean): true if ANY meeting note is in changed/new/deleted files, OR if any changed file has `has_action_items: true` in manifest

Also: if a project/person note itself changed, add its slug to `dirty_projects`/`dirty_people`.

---

## Phase 3 — Assemble data for dirty indexes (no additional reads)

**Critical rule: do not read any file from disk in this phase.** All data must come from either `parsed` (changed/new files, read in Phase 1b) or the manifest cache (unchanged files). If data for a file is not in either source, that file must have been missed in Phase 1b — go back and read it then.

For each dirty index, determine which files contribute to it using the unified view (manifest cache keys + `parsed` keys), then collect their data:

- **Dirty project index `P`**: collect all files where `projects` contains `P`. Data comes from `parsed[file]` if the file was changed/new, else `manifest_data.files[file]` for unchanged files. Fields needed: `title`, `date`, `type`, `people`, `action_items`, `resolves`, `decisions`.

- **Dirty person index `P`**: collect all files where `people` contains `P`. Same source rule.

- **Dirty daily-combined `DATE`**: collect all files where `date == DATE`. Same source rule.

- **`open-actions.md` / `all-actions.md`**: collect `action_items` and `resolves` from ALL meeting files using the same source rule — `parsed` for changed files, manifest cache for unchanged. No additional reads.

The manifest caches enough fields to fully render any index without touching source files. This is intentional: the only disk I/O for a rebuild is **writing** the affected index files.

---

## Phase 4 — Build resolution map

From all meeting notes' `resolves` entries (from `parsed` + manifest cache):

Build `resolution_map`: `{ "source_meeting_slug|owner|task" → { outcome, note, resolved_in_title, resolved_in_file } }`

Key = composite of source_meeting slug + owner + task text (normalized: lowercase, trimmed).

Apply to action items: if a key exists in resolution_map, that item's effective status = outcome value.

---

## Phase 5 — Write dirty project indexes

For each slug in `dirty_projects`:

a. Get project file data (from `parsed` if it was changed, else manifest cache, else read `projects/<slug>.md`).
b. Gather all entries tagged with this project from unified view.
c. Apply resolution_map for effective action item statuses.
d. Collect unique person slugs; resolve display names.
e. Write `$WORKSPACE/_index/by-project/<slug>.md`:

```markdown
---
generated: NOW
source: /sync
project: SLUG
---

# Project Index: TITLE

> Auto-generated by `/sync` on NOW. Do not edit manually.

## Project Document
[TITLE](../../projects/SLUG.md) — `STATUS` — owner: OWNER — target: TARGET_DATE

## Meetings (COUNT)
| Date | Title | People |
|------|-------|--------|
| DATE | [TITLE](../../meetings/FILENAME.md) | SLUGS |

## Daily Notes (COUNT)
| Date | File |
|------|------|
| DATE | [DATE](../../daily/DATE.md) |

## Open Action Items
| Owner | Task | Due | Source |
|-------|------|-----|--------|
| OWNER | TASK | DUE | [MEETING](../../meetings/FILENAME.md) |

## Resolved Action Items
| Owner | Task | Outcome | Resolved In | Note |
|-------|------|---------|-------------|------|
| OWNER | TASK | done/partial/carried-over | [MEETING](../../meetings/FILENAME.md) | NOTE |

*(Omit this section if no resolved items.)*

## People Involved
- [DISPLAY NAME](../../people/SLUG.md)
```

---

## Phase 6 — Write dirty person indexes

For each slug in `dirty_people`:

a. Get person file data.
b. Gather all entries tagged with this person from unified view.
c. Apply resolution_map. Group action items by project, split open/resolved.
d. Write `$WORKSPACE/_index/by-person/<slug>.md`:

```markdown
---
generated: NOW
source: /sync
person: SLUG
---

# Person Index: DISPLAY NAME

> Auto-generated by `/sync` on NOW. Do not edit manually.

## Profile
[DISPLAY NAME](../../people/SLUG.md) — ROLE, TEAM

## Projects (COUNT)
- [PROJECT TITLE](../../_index/by-project/SLUG.md) — `STATUS`

## Meetings (COUNT)
| Date | Title | Project |
|------|-------|---------|
| DATE | [TITLE](../../meetings/FILENAME.md) | PROJECT SLUG |

## Action Items

### PROJECT TITLE
**Open**
| Task | Due | Source |
|------|-----|--------|
| TASK | DUE | [MEETING](../../meetings/FILENAME.md) |

**Resolved**
| Task | Outcome | Resolved In | Note |
|------|---------|-------------|------|
| TASK | done/partial/carried-over | [MEETING](../../meetings/FILENAME.md) | NOTE |

*(Repeat one ### section per project. Omit Open or Resolved subsection if empty. If no action items at all: "None assigned.")*

## Daily Note Mentions (COUNT)
| Date | File |
|------|------|
| DATE | [DATE](../../daily/DATE.md) |
```

---

## Phase 7 — Write dirty daily-combined indexes

For each date in `dirty_dates`:

a. Get daily note data for that date (if any).
b. Get all meeting notes for that date.
c. Write `$WORKSPACE/_index/daily-combined/<date>-combined.md`:

```markdown
---
generated: NOW
source: /sync
date: DATE
---

# Combined Daily View — HUMAN_DATE

> Auto-generated by `/sync`. Do not edit manually.

---

## Daily Journal
*Source: [daily/DATE.md](../../daily/DATE.md)*

### Yesterday
CONTENT

### Today
CONTENT

### Blockers
CONTENT

---

## Meetings Today (COUNT)

### MEETING TITLE
*Source: [FILENAME](../../meetings/FILENAME.md)*
**Attendees:** NAMES
**Decisions:** LIST
**Action Items:** OWNER: TASK (due DATE)

---

## Projects Active Today
- [TITLE](../../projects/SLUG.md)

## People Mentioned Today
- [NAME](../../people/SLUG.md)
```

---

## Phase 8 — Write open-actions and all-actions indexes

Only if `actions_dirty`:

Collect ALL `action_items` from all meeting notes (using `parsed` + manifest cache `action_items` field). Apply resolution_map.

**open-actions.md** — only items whose effective status is open (not resolved as `done`). Items with `partial` or `carried-over` remain open, labelled accordingly. Sort by `due` ascending; no-due-date items go last. Prefix past-due items (due < TODAY) with ⚠️.

Write `$WORKSPACE/_index/open-actions.md`:

```markdown
---
generated: NOW
source: /sync
---

# Open Action Items

> Auto-generated by `/sync` on NOW. Do not edit manually.
> To close an item: add a `resolves:` entry to the follow-up meeting note, then re-run `/sync`.

| # | Owner | Task | Due | Status | Project | Source |
|---|-------|------|-----|--------|---------|--------|
| 1 | OWNER | TASK | DUE | open/partial/carried-over | PROJECT | [MEETING](../../meetings/FILENAME.md) |
```

**all-actions.md** — ALL action items, grouped by project slug (alphabetical). Within each project, sort by due ascending (open/partial/carried-over first, then done). Prefix past-due open items with ⚠️.

Write `$WORKSPACE/_index/all-actions.md`:

```markdown
---
generated: NOW
source: /sync
---

# All Action Items

> Auto-generated by `/sync` on NOW. Do not edit manually.
> Full lifecycle view — open and resolved — grouped by project.

---

## PROJECT TITLE
*[project file](../projects/SLUG.md) — `STATUS`*

| Owner | Task | Due | Status | Source |
|-------|------|-----|--------|--------|
| OWNER | TASK | DUE | open | [MEETING](../meetings/FILENAME.md) |
| OWNER | TASK | DUE | ⚠️ overdue | [MEETING](../meetings/FILENAME.md) |
| OWNER | TASK | DUE | partial | [MEETING](../meetings/FILENAME.md) — resolved in [MEETING](../meetings/FILENAME.md) |
| OWNER | TASK | DUE | carried-over | [MEETING](../meetings/FILENAME.md) — resolved in [MEETING](../meetings/FILENAME.md) |
| OWNER | TASK | DUE | ~~done~~ | [MEETING](../meetings/FILENAME.md) — resolved in [MEETING](../meetings/FILENAME.md) |

*(Repeat one ## section per project. Done items use strikethrough on the task text.)*
```

---

## Phase 9 — Update manifest

Write `.sync-manifest.json` with updated data. Structure:

```json
{
  "last_sync": "NOW",
  "files": {
    "meetings/2026-04-06-foo.md": {
      "type": "meeting",
      "date": "2026-04-06",
      "title": "Meeting Title",
      "projects": ["slug-a"],
      "people": ["person-a", "person-b"],
      "action_items": [
        { "owner": "person-a", "task": "Do thing", "due": "2026-04-10", "status": "open" }
      ],
      "resolves": [
        { "source_meeting": "2026-03-01-foo", "owner": "person-a", "task": "Old thing", "outcome": "done", "note": "" }
      ],
      "decisions": ["Chose option A"]
    },
    "daily/2026-04-06.md": {
      "type": "daily",
      "date": "2026-04-06",
      "title": "2026-04-06",
      "projects": ["slug-a"],
      "people": ["person-a"],
      "action_items": [],
      "resolves": [],
      "decisions": []
    }
  }
}
```

Rules:
- Remove entries for `deleted_files`.
- Update entries for all files in `changed_files` / new files using data from `parsed`.
- **Never re-read unchanged files** — keep their entries exactly as-is.
- Every field used in Phase 3 index assembly (`title`, `date`, `type`, `projects`, `people`, `action_items`, `resolves`, `decisions`) must be present for every file. Omitted fields must default to `[]` or `""` — never left absent — so Phase 3 can always assemble indexes without fallback reads.

After writing the manifest, touch it so its mtime is after all current note files:
```
touch "$MANIFEST"
```

---

## Phase 10 — Report to user

```
/sync complete — NOW

Mode: incremental  (or "full rebuild" if --force)
Changed/new: N files | Deleted: N files

Indexes rebuilt:
  _index/by-project/:      SLUG, SLUG, ...   (or "none")
  _index/by-person/:       SLUG, SLUG, ...   (or "none")
  _index/daily-combined/:  DATE, DATE, ...   (or "none")
  _index/open-actions.md:  N open items      (or "skipped — no meeting changes")
  _index/all-actions.md:   N total items     (or "skipped — no meeting changes")

Skipped (unchanged): N indexes

⚠ Warnings: [orphaned slugs with the file they appear in]
```
