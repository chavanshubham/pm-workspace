---
name: person
description: Create or open a person profile note. Triggers on "new person", "add contact", "person note", or a person's name.
allowed-tools: Read, Write, Glob, Bash
argument-hint: "<person-name>"
---

Create or open a person profile note in the PM workspace.

WORKSPACE: /Users/shubhamchavan/Desktop/programs/work
TODAY: !`date +%Y-%m-%d`
NOW: !`date +%Y-%m-%dT%H:%M`

ARGUMENTS: $ARGUMENTS

## Steps

1. If ARGUMENTS is empty, ask the user for the person's name before continuing.

2. Derive the slug: lowercase, spaces → hyphens, remove special chars.
   Example: "Sarah Kim" → `sarah-kim`
   FILE = `$WORKSPACE/people/<slug>.md`

3. Check if FILE exists using Glob.
   - If it EXISTS: Read and display it. Tell the user the profile is open. Stop here.
   - If it DOES NOT EXIST: continue.

4. Write a new person profile to FILE:

```
---
type: person
title: "<Full Name>"
slug: <slug>
date: <TODAY>
created: <NOW>
updated: <NOW>
projects: []
people:
  - <slug>
tags: []
role: ""
team: ""
---

# <Full Name>

## Profile
**Role:**
**Team:**
**Contact:**

## Notes

## Interaction Log
```

   Replace all placeholders with actual values.

5. Display the created note.

6. Remind the user:
   - Fill in role, team, contact details.
   - Add projects this person is involved in to `projects:`.
   - Run `/sync` after editing to update the person index.
