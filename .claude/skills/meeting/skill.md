---
name: meeting
description: Create a new structured meeting note. Triggers on "new meeting", "meeting notes", "log a meeting", or when the user describes a meeting.
allowed-tools: Read, Write, Glob, Bash
argument-hint: "<meeting-title> [attendee-slug,attendee-slug] [project-slug]"
---

Create a meeting note in the PM workspace.

WORKSPACE: /Users/shubhamchavan/Desktop/programs/work
TODAY: !`date +%Y-%m-%d`
NOW: !`date +%Y-%m-%dT%H:%M`

ARGUMENTS: $ARGUMENTS

## Steps

1. Parse ARGUMENTS:
   - Meeting title: required. If not provided, ask the user before continuing.
   - Attendees: optional comma-separated person slugs (e.g. `sarah-kim,james-liu`)
   - Project slug(s): optional (e.g. `alpha-redesign`)

2. Derive filename slug from the title: lowercase, spaces → hyphens, remove special chars.
   Example: "Alpha Kickoff" → `alpha-kickoff`
   FILE = `$WORKSPACE/meetings/$TODAY-<title-slug>.md`

3. Check if FILE already exists using Glob.
   - If it EXISTS: Read and display it. Tell the user the note is already open. Stop here.
   - If it DOES NOT EXIST: continue.

4. Read the template at `$WORKSPACE/_templates/meeting.md`.

5. Replace placeholders:
   - `{{TITLE}}` → full meeting title
   - `{{DATE}}` → TODAY
   - `{{DATETIME}}` → NOW

6. In the frontmatter YAML:
   - Set `people:` to a YAML list of any attendee slugs provided.
   - Set `projects:` to a YAML list of any project slugs provided.

7. Write the filled note to FILE.

8. Display the created note.

9. Remind the user:
   - Fill in agenda, notes, decisions, and action items in the body.
   - Add new action items to the `action_items:` YAML list in frontmatter for indexing (not just prose).
   - If this meeting followed up on action items from a previous meeting, fill in the `resolves:` block — one entry per item that was closed, partially completed, or carried over. `/sync` uses this to update the action item lifecycle across all indexes without editing the original meeting note.
   - Run `/sync` after editing to update all indexes.
   - Run `/mom` on this note when the meeting is done to generate a shareable MoM.
