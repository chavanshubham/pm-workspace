---
name: project-note
description: Create or open a project living document. Triggers on "new project", "project note", "open project", or a project name.
allowed-tools: Read, Write, Glob, Bash
argument-hint: "<project-name>"
---

Create or open a project living document in the PM workspace.

WORKSPACE: /Users/shubhamchavan/Desktop/programs/work
TODAY: !`date +%Y-%m-%d`
NOW: !`date +%Y-%m-%dT%H:%M`

ARGUMENTS: $ARGUMENTS

## Steps

1. If ARGUMENTS is empty, ask the user for the project name before continuing.

2. Derive the slug from the project name: lowercase, spaces → hyphens, remove special chars.
   Example: "Alpha Redesign" → `alpha-redesign`
   FILE = `$WORKSPACE/projects/<slug>.md`

3. Check if FILE exists using Glob.
   - If it EXISTS: Read and display it. Tell the user the project note is open. Stop here.
   - If it DOES NOT EXIST: continue.

4. Read the template at `$WORKSPACE/_templates/project.md`.

5. Replace placeholders:
   - `{{TITLE}}` → original project name (preserve capitalisation)
   - `{{SLUG}}` → derived slug
   - `{{DATE}}` → TODAY
   - `{{DATETIME}}` → NOW

6. Write the filled note to FILE.

7. Display the created note.

8. Remind the user:
   - Add team members/stakeholders to `people:` for cross-indexing.
   - Set `owner:` and `target_date:` fields.
   - Run `/sync` after editing to update indexes.
