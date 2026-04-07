---
name: week
description: Generate a weekly recap from this week's notes. Triggers on "weekly summary", "week in review", "what happened this week", "weekly recap".
allowed-tools: Read, Glob, Bash
argument-hint: "[YYYY-MM-DD]  (optional: Monday start date, defaults to current week)"
---

Generate a weekly summary from this week's PM notes.

WORKSPACE: /Users/shubhamchavan/Desktop/programs/work
TODAY: !`date +%Y-%m-%d`
WEEK_MONDAY: !`date -v-$(( $(date +%u) - 1 ))d +%Y-%m-%d 2>/dev/null || date -d "last monday" +%Y-%m-%d`

ARGUMENTS: $ARGUMENTS

## Steps

1. Determine the week range:
   - If ARGUMENTS contains a date, use it as the Monday start date.
   - Otherwise use WEEK_MONDAY.
   - Week end = Friday (or TODAY if TODAY is before Friday).
   - List all dates Mon–Fri.

2. Collect all notes for the week:
   - For each day, check for `$WORKSPACE/daily/<date>.md` and `$WORKSPACE/_index/daily-combined/<date>-combined.md`.
   - Glob all `$WORKSPACE/meetings/<date>-*.md` files for dates in range.

3. If no notes found for the week, tell the user and stop.

4. Build the summary:
   a. Projects touched: deduplicate `projects:` across all note frontmatter.
   b. Meetings: list all with date, title, attendees, and first decision if any.
   c. Decisions made: collect all `decisions:` from meeting frontmatter.
   d. Action items created: collect all `action_items:` from meeting frontmatter.
   e. Day highlights: 1–2 bullets per day from the "Yesterday"/"Today" sections of daily notes.

5. Output:

---

## Week in Review — <Monday> to <Friday>

### Projects This Week
- <project>: <brief note on what happened>

### Meetings (<count>)
| Date | Meeting | Key Decision |
|------|---------|--------------|

### Decisions Made
-

### Action Items Created
| Owner | Task | Due | Project |
|-------|------|-----|---------|

### Day by Day
**Mon <date>:**
**Tue <date>:**
**Wed <date>:**
**Thu <date>:**
**Fri <date>:**

### Reflection
<2–3 sentences: what was the week's main focus, what moved forward, what carries into next week.>

---
