---
name: standup
description: Generate a standup summary from yesterday's notes. Triggers on "standup", "generate standup", "what did I do yesterday", "daily standup".
allowed-tools: Read, Glob, Bash
---

Generate a standup summary from yesterday's PM notes.

WORKSPACE: /Users/shubhamchavan/Desktop/programs/work
TODAY: !`date +%Y-%m-%d`
YESTERDAY: !`date -v-1d +%Y-%m-%d 2>/dev/null || date -d yesterday +%Y-%m-%d`

## Steps

1. Determine YESTERDAY from the injected value above.

2. Collect sources (read whichever exist):
   - Combined view: `$WORKSPACE/_index/daily-combined/$YESTERDAY-combined.md` — prefer this if it exists.
   - Daily note: `$WORKSPACE/daily/$YESTERDAY.md`
   - Meeting notes: glob `$WORKSPACE/meetings/$YESTERDAY-*.md`

3. If none of these files exist, tell the user "No notes found for YESTERDAY. Did you log anything that day?" and stop.

4. Extract:
   - **Done**: "Yesterday" section of daily note + meetings held (title + key outcome in one sentence each)
   - **Doing today**: "Today" section of daily note + follow-up action items due today or overdue
   - **Blockers**: "Blockers" section of daily note. If empty, write "None."

5. Output:

---

## Standup — TODAY

**Done (yesterday):**
-

**Doing today:**
-

**Blockers:**
- None

**Active projects:** <comma-separated project names from yesterday's notes>

---

Keep every bullet to one sentence maximum.
