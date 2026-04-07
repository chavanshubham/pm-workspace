---
name: daily
description: Create or open today's daily PM note. Triggers on "daily note", "open today", "start my day", "journal".
allowed-tools: Read, Write, Glob, Bash
---

Create or open today's daily note in the PM workspace.

WORKSPACE: /Users/shubhamchavan/Desktop/programs/work
TODAY: !`date +%Y-%m-%d`
NOW: !`date +%Y-%m-%dT%H:%M`

## Steps

1. Set FILE to `$WORKSPACE/daily/$TODAY.md`.

2. Check if FILE already exists using Glob.
   - If it EXISTS: Read and display its contents. Tell the user the note is already open. Stop here.
   - If it DOES NOT EXIST: continue to step 3.

3. Read the template at `$WORKSPACE/_templates/daily.md`.

4. Replace all placeholders:
   - `{{DATE}}` → TODAY value
   - `{{DATETIME}}` → NOW value

5. Write the filled template to FILE.

6. Display the created note.

7. Tell the user: "Add project slugs to `projects:` and person slugs to `people:` as you fill this in. Run `/sync` when done to update all indexes."
