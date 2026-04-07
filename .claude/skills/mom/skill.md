---
name: mom
description: Generate a shareable Minutes of Meeting document from a meeting note. Triggers on "write MoM", "meeting minutes", "summarize meeting", "MoM for".
allowed-tools: Read, Write, Glob, Bash
argument-hint: "<meeting-title-or-date-slug>"
---

Generate a formatted Minutes of Meeting (MoM) document from a meeting note.

WORKSPACE: /Users/shubhamchavan/Desktop/programs/work
TODAY: !`date +%Y-%m-%d`
NOW: !`date +%Y-%m-%dT%H:%M`

ARGUMENTS: $ARGUMENTS

## Steps

1. Find the meeting note:
   - If ARGUMENTS looks like a date or slug, glob `$WORKSPACE/meetings/$ARGUMENTS*.md` and pick the best match.
   - If ARGUMENTS is a plain title, glob `$WORKSPACE/meetings/*.md` and find the file whose title most closely matches.
   - If no match found, list available meeting files and ask the user to clarify.

2. Read the meeting note. Extract from frontmatter:
   - `title`, `date`, `people`, `projects`, `meeting_duration`, `location`
   - `action_items` (YAML list of `{owner, task, due, status}`)
   - `decisions` (YAML list of strings)
   Also read the full body for any action items mentioned in prose but not yet in frontmatter.

3. If there are action items in the prose body NOT yet in the `action_items:` frontmatter, add them to the frontmatter list and re-write the file. Set `status: open` for each.

4. For each slug in `people:`, check if `$WORKSPACE/people/<slug>.md` exists and read the `title:` field for a display name. If not found, use the slug as-is.

5. Output the MoM:

---

## Minutes of Meeting — <title>

**Date:** <date>
**Attendees:** <comma-separated display names>
**Duration:** <meeting_duration> minutes
**Location:** <location>
**Projects:** <comma-separated project slugs>

---

### Agenda / Discussion

<Summarise the prose notes from the meeting body into clear bullet points. Group by topic if possible.>

---

### Decisions

<Numbered list from the `decisions:` frontmatter. If empty: "No formal decisions recorded.">

---

### Action Items

| # | Owner | Task | Due Date | Status |
|---|-------|------|----------|--------|
| 1 | <display name> | <task> | <due> | Open |

---

### Next Steps

<2–3 sentences about what happens next based on action items and decisions.>

---

*MoM generated on <TODAY>*

---

6. After outputting, tell the user:
   - "Ready to copy into an email or doc."
   - "Use `/email <person>` to draft a follow-up email with this MoM."
   - "Run `/sync` to update project and person indexes with these action items."
