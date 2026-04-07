---
name: email
description: Draft a professional email to a person, optionally about a project or meeting. Triggers on "draft email", "email to", "send update to", "write email".
allowed-tools: Read, Write, Glob, Bash
argument-hint: "<person-slug-or-name> [subject or context]"
---

Draft a professional email to a person, grounded in your PM notes.

WORKSPACE: /Users/shubhamchavan/Desktop/programs/work
TODAY: !`date +%Y-%m-%d`

ARGUMENTS: $ARGUMENTS

## Steps

1. Parse ARGUMENTS:
   - Person slug or name (required). Convert name to slug if needed (lowercase, spaces → hyphens).
   - Subject or context (optional) — the purpose of the email.
   - If person is not provided, ask before continuing.

2. Read `$WORKSPACE/people/<slug>.md` if it exists. Extract display name, role, team, and any communication preferences from the Notes section.

3. Determine email context from ARGUMENTS clues:
   - If context mentions a meeting: glob `$WORKSPACE/meetings/*.md`, find the most relevant recent meeting involving this person.
   - If context mentions a project: read `$WORKSPACE/projects/<project-slug>.md`.
   - If context mentions action items or follow-up: check `$WORKSPACE/_index/by-person/<slug>.md` if it exists.
   - If no specific context: read the last 3 daily notes in `$WORKSPACE/daily/` that mention this person's slug.

4. Draft the email. Tone: professional, concise, direct.

   Format:
   ```
   Subject: <subject line>

   Hi <first name>,

   <Opening — 1 sentence: why you're writing>

   <Body — context, key points, decisions, or action items. Use bullet points if more than 2 items.>

   <Clear ask or next step — what you need from them, or what they should expect next>

   Best,
   <sign-off>
   ```

5. Output the draft directly (do not save to a file) — formatted for copy-paste.

6. After the draft, offer:
   - "Want me to adjust the tone (more formal / more casual)?"
   - "Want me to include the MoM from [meeting name] inline?"
