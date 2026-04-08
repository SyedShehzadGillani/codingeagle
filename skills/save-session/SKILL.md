---
name: save-session
description: >-
  Use when ending a work session, switching context, pausing work, or before closing
  a conversation. Captures the primary goal, decisions made and why, changes applied,
  what's unfinished, and where to resume. Creates a session log that the next conversation
  can read for full context without re-asking everything. Activates on keywords: "save session",
  "save progress", "end session", "wrap up", "stopping for now", "pause work", "picking this up later",
  "context dump", "session log", "save state", "bookmark session".
argument-hint: "[optional: session title or project name]"
disable-model-invocation: true
---

## Target

Save session for: **$ARGUMENTS**

If no argument, infer the project/feature name from the conversation context.

# Save Session -- Conversation Context Preservation

## Overview

You capture the **reasoning, decisions, and state** of the current conversation into a structured markdown file. This preserves what docs alone cannot: WHY decisions were made, what the user said, what was considered and rejected, what's half-finished, and exactly where to pick up next time.

**Core principle:** Docs capture outputs. Session logs capture process. The next conversation needs both.

## Invocation Behavior

When this skill is invoked:

1. **Do NOT ask questions** -- extract everything from the conversation history
2. Analyze the full conversation to build the session log
3. Write the session file
4. Commit it

This skill is **retrospective** -- it looks backward at what happened, not forward at what to do.

---

## Session Log Structure

### File Location

```
docs/sessions/
  session-[YYYY-MM-DD]-[slug].md        -- one file per session
```

If `docs/sessions/` doesn't exist, create it. If a session file already exists for today with the same slug, append a sequence number: `session-2026-04-08-auth-2.md`.

### File Format

```markdown
# Session Log: [Title]

**Date:** [YYYY-MM-DD]
**Duration:** [approximate, from first to last message timestamps if detectable]
**Project:** [project or feature name]
**Status:** [In Progress / Paused / Completed / Blocked]

---

## Primary Goal

[1-3 sentences. What the user set out to accomplish this session.
Not what was done -- what was INTENDED.]

## What Was Accomplished

[Bulleted list of concrete outcomes. Be specific:]
- [Created/modified/deleted specific files]
- [Completed specific pipeline phases]
- [Fixed specific bugs]
- [Made specific decisions]

### Key Changes

| File | Change | Why |
|------|--------|-----|
| [path] | [what changed] | [why this decision was made] |
| [path] | [what changed] | [why] |

### Commits Made

| Hash | Message |
|------|---------|
| [short hash] | [commit message] |
| [short hash] | [commit message] |

## Decisions Made

[This is the most valuable section. Capture every non-obvious decision
with the reasoning. The next conversation needs to know WHY, not just WHAT.]

### [Decision 1 Title]
**Chose:** [what was decided]
**Over:** [what alternatives were considered]
**Because:** [the reasoning -- user preference, technical constraint, trade-off]

### [Decision 2 Title]
**Chose:** [...]
**Over:** [...]
**Because:** [...]

## What Was Rejected / Cut

[Things that were explicitly decided against. Critical for the next session
to not re-propose them.]

- [Rejected idea 1] -- reason: [why it was cut]
- [Rejected idea 2] -- reason: [why]

## Unfinished Work

[What's left to do. Be specific enough that the next session can pick up
without re-reading the entire conversation.]

### Immediate Next Steps
1. [Specific next action] -- [context needed to do it]
2. [Specific next action] -- [context]

### Open Questions
- [Question that wasn't resolved this session]
- [Question that came up but was deferred]

### Blocked Items
- [Item] -- blocked by: [what's blocking it]

## User Preferences Learned

[Things the user said about how they work, what they like/dislike,
communication style, technical preferences. These help the next
session start right.]

- [Preference 1 -- e.g., "prefers terse responses"]
- [Preference 2 -- e.g., "wants auto mode for pipeline"]
- [Preference 3 -- e.g., "doesn't want vulnerability testing, Claude handles security"]

## Technical Context

[State of the codebase that's not obvious from the code itself.]

- **Branch:** [current branch name]
- **Pipeline phase:** [if in a propose pipeline, which phase]
- **Test status:** [passing / failing / not run]
- **Pending PR:** [if any]
- **Environment notes:** [anything unusual about the dev setup]

## Resume Instructions

[Exactly what the next session should do first. Write this as if briefing
a colleague who just sat down at the keyboard.]

```
To resume this work:
1. [First thing to do]
2. [Second thing to do]
3. [What to check before continuing]

Key files to read first:
- [file 1] -- [why]
- [file 2] -- [why]

Active docs:
- [docs/path] -- [what it contains]
```
```

---

## Extraction Logic

How to build each section from the conversation:

### Primary Goal
- Look at the user's FIRST message in the conversation
- If it was a `/propose`, `/spec-sharder`, or other skill invocation, the argument IS the goal
- If it was a general request, synthesize the intent
- If the goal evolved during the conversation, capture both: "Started as [X], evolved to [Y]"

### What Was Accomplished
- Scan for all Write, Edit, and Bash(git commit) tool calls
- Extract file paths and commit messages
- Summarize what each change accomplished

### Decisions Made
- Scan for moments where the user chose between options
- Scan for moments where YOU recommended something and the user agreed/modified
- Scan for "let's go with", "yes", "no don't do that", "use X instead of Y"
- Capture the alternatives that were on the table, not just the winner

### What Was Rejected
- Scan for "no", "don't", "skip", "remove", user denying a tool call
- Scan for features/approaches that were discussed but explicitly not done

### Unfinished Work
- Check TaskList for pending/in-progress tasks
- Check if the last message implies more work to do
- Check if a pipeline was partially completed (which phase are we on?)
- Check git status for uncommitted changes

### User Preferences
- Scan for corrections: "don't do X", "I prefer Y", "stop doing Z"
- Scan for positive reactions: "perfect", "yes exactly", "keep doing that"
- Note communication style preferences (terse vs verbose, plan mode vs direct action)

### Technical Context
- Run `git branch --show-current` for current branch
- Run `git status` for working tree state
- Check for running processes (dev server)
- Note any environment-specific information mentioned in conversation

---

## Multiple Sessions Per Project

When a project spans multiple sessions, the session logs build a chronological record:

```
docs/sessions/
  session-2026-04-07-codingeagle.md     -- first session
  session-2026-04-08-codingeagle.md     -- second session
  session-2026-04-10-codingeagle.md     -- third session
```

Each session log is independent but references prior sessions when relevant:

```markdown
## Prior Sessions
- [2026-04-07](./session-2026-04-07-codingeagle.md): Created 9 initial skills
- [2026-04-08](./session-2026-04-08-codingeagle.md): Added cross-cutting protocols

## This Session
[continues from where prior sessions left off]
```

---

## Auto-Save Trigger

When the user says anything that implies they're stopping:
- "I'm done for now"
- "let's pick this up tomorrow"
- "saving and closing"
- "end of day"
- "that's all for now"
- "wrapping up"

Proactively suggest: "Want me to save a session log before you go? (`/save-session`)"

This is a SUGGESTION only -- never auto-invoke without the user's explicit request.

---

## Session Index

If multiple session logs exist, maintain a lightweight index at `docs/sessions/INDEX.md`:

```markdown
# Session Log Index

| Date | Project | Status | Summary |
|------|---------|--------|---------|
| 2026-04-07 | codingeagle | Completed | Created 9 SDLC skills, pushed to GitHub |
| 2026-04-08 | codingeagle | Completed | Added protocols, new skills, image handling |
| 2026-04-10 | codingeagle | Paused | Started devops skill, blocked on Docker config |
```

Update this index every time a session log is saved.

---

## Operating Principles

1. **Capture reasoning, not just actions** -- the next session can see what changed in git; it can't see WHY
2. **Decisions are the most valuable section** -- alternatives considered + reasoning is what prevents re-litigating settled questions
3. **Be specific about resume** -- "continue working" is useless; "run `npm test` to verify auth fixes, then start on the dashboard component" is useful
4. **Don't fabricate** -- only include information actually present in the conversation; if you're unsure about something, mark it `[uncertain]`
5. **Keep it scannable** -- the next session should be able to skim this in 30 seconds and know exactly where to start
6. **Capture rejections** -- knowing what was said NO to is as valuable as knowing what was said YES to
7. **User preferences compound** -- each session learns more about how the user works; capture these so the experience improves over time

## Document Output

| Output | File Location |
|--------|---------------|
| Session log | `docs/sessions/session-[YYYY-MM-DD]-[slug].md` |
| Session index | `docs/sessions/INDEX.md` |

### Size Rules
- Session logs should be 50-150 lines. If longer, you're being too verbose.
- Decisions section: 3-5 lines per decision max.
- Resume instructions: 10-15 lines max. If it takes more, link to the relevant doc.

## Cross-Cutting Protocols

### Git Workflow
- **Commit tag:** `docs`
- **Commit when:** After session log is saved
- **Commit format:** `docs: save session log for [project] ([status])`
- **Branch:** Commit to current branch (session logs are lightweight metadata)

### Resume Protocol
- **On start of NEW conversation:** Check `docs/sessions/` for recent session logs
- **If found:** Read the most recent session log for this project
- **Auto-brief:** "I found a session log from [date]. Here's where we left off: [resume instructions summary]. Continue from here?"
- This is how sessions chain together

### Context Loading
- Session logs ARE context. The next conversation reads them to understand prior work.
- Skills should check `docs/sessions/` during Context Pass-Through if they exist.

### Smart Skip
- Not applicable -- this skill extracts from conversation, doesn't ask questions.

---

## Pipeline Integration

### When to Invoke

| Trigger | Behavior |
|---------|----------|
| User says `/save-session` | Save immediately |
| User implies stopping ("done for now", "wrapping up") | Suggest saving |
| Pipeline paused mid-phase | Capture current phase + context for resume |
| Context approaching limit | Suggest saving before compression loses conversation details |

### Relationship to Other Skills

- **Complements Resume Protocol**: Resume checks for phase docs (PRDs, architecture). Session logs capture the conversation context that phase docs don't.
- **Complements doc-cleanup**: Session logs in `docs/sessions/` are subject to doc-cleanup rules (stale after 90 days since they're historical records, not active docs).
- **Feeds into next session**: Any skill can read session logs during Context Loading to understand prior work and decisions.
