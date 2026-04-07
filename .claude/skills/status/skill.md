---
name: status
description: Generate a project status report. Triggers on "project status", "status of", "how is X going", "status update for", "status report".
allowed-tools: Read, Write, Glob, Bash
argument-hint: "<project-name-or-slug>"
---

Generate a concise project status report from your PM notes.

WORKSPACE: /Users/shubhamchavan/Desktop/programs/work
TODAY: !`date +%Y-%m-%d`

ARGUMENTS: $ARGUMENTS

## Steps

1. Identify the project:
   - Derive slug from ARGUMENTS (lowercase, spaces → hyphens).
   - Read `$WORKSPACE/projects/<slug>.md`. If not found, glob `$WORKSPACE/projects/*.md` and pick the closest title match.
   - If still not found, list available projects and ask.

2. Gather supporting data:
   a. Project doc: read fully. Extract `status`, `owner`, `target_date`, goals, risks, open questions.
   b. Recent meetings: glob `$WORKSPACE/meetings/*.md`, read frontmatter of all where `projects:` contains this slug. Take the 5 most recent by date.
   c. Open action items: if `$WORKSPACE/_index/open-actions.md` exists, grep lines containing this project slug. Otherwise scan meeting frontmatter for `action_items` with `status: open`.
   d. Recent daily notes: glob `$WORKSPACE/daily/*.md`, read frontmatter of the 5 most recent where `projects:` contains this slug.

3. Output the status report:

---

## <Project Title> — Status Update
**Date:** TODAY
**Status:** <status from project doc>
**Owner:** <owner>
**Target Date:** <target_date>

---

### Summary
<2–4 sentences: what is this project, where does it stand right now, what is the current momentum.>

---

### Recent Activity
<Bullet points from the last 2–3 meetings and daily notes. Format: "DATE — what happened / was decided.">

---

### Open Action Items

| # | Owner | Task | Due | Status |
|---|-------|------|-----|--------|
| 1 | | | | Open |

<If no open items: "No open action items.">

---

### Risks & Blockers
<From the Risks section of the project doc and blockers in recent daily notes. If none: "No active risks or blockers.">

---

### Next Steps
<2–3 bullets: what needs to happen next, with owners if known.>

---

4. After outputting, offer:
   - "Want me to draft an email to <owner> with this status update?"
   - "Want me to save this as an entry in the project file?"
