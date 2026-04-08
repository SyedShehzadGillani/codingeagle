---
name: doc-cleanup
description: >-
  Use when docs/ directories are bloated, outdated, or wasting context tokens.
  Scans for stale, oversized, resolved, orphaned, and duplicate docs, then archives
  or compresses them directly. Enforces size limits (500 lines per doc, 200KB total).
  Activates on keywords: "cleanup docs", "stale docs", "doc bloat", "docs too big",
  "context bloat", "archive docs", "shrink docs", "doc maintenance", "old reports",
  "clean up docs", "doc-cleanup", "prune docs".
argument-hint: "[project-slug | all] [--dry-run] [--hard-delete] [--undo last|date] [--max-age N] [--max-lines N] [--max-size NKB]"
---

## Target

Clean docs for: **$ARGUMENTS**

If no target is specified, scan the current project's `docs/` directory. If `all` is specified, scan every `docs/*/` subdirectory.

# Doc Cleanup -- Documentation Hygiene Engine

## Overview

You are a **documentation hygiene engine**. You scan project docs for stale, oversized, resolved, orphaned, and duplicate files -- then archive, compress, or delete them directly. Your purpose is keeping `docs/` lean so that Context Pass-Through doesn't waste tokens on information nobody needs.

**Core principle:** Every byte in `docs/` competes for context tokens. Resolved findings, completed shards, superseded decisions, and verbose changelogs are dead weight. Remove them.

## Invocation Behavior

When this skill is invoked:

1. **Do NOT enter plan mode** -- this skill takes direct action
2. Parse arguments for flags (`--dry-run`, `--hard-delete`, `--undo`, `--max-age`, `--max-lines`, `--max-size`)
3. Run the 4-phase pipeline: Scan -> Classify -> Act -> Report

### Argument Flags

| Flag | Default | Effect |
|------|---------|--------|
| `--dry-run` | off | Run Scan + Classify only, show what WOULD be done, take no action |
| `--hard-delete` | off | Permanently delete instead of archiving (warns before proceeding) |
| `--undo last` | -- | Restore all files from the most recent cleanup run |
| `--undo [YYYY-MM-DD]` | -- | Restore all files archived on a specific date |
| `--max-age N` | 30 | Days without update before a doc is classified STALE |
| `--max-lines N` | 500 | Per-file line limit before compression triggers |
| `--max-size NKB` | 200 | Total `docs/` directory size limit in KB |

### Undo Mode

When `--undo` is specified, skip the normal pipeline and restore:

```
UNDO MODE

Restoring from: docs/.archive/[date]-*
Files to restore: [N]

Restoring:
  docs/.archive/2026-04-07-qa-report-auth.md → docs/[slug]/qa-report-auth.md
  docs/.archive/2026-04-07-triage-log-auth.md → docs/[slug]/triage-log-auth.md
  [...]

Restored [N] files. Archive directory unchanged (originals still in .archive/).
```

---

## Phase 1: Scan

Collect a complete inventory of every markdown file in the target `docs/` directory.

```
DOC CLEANUP ENGINE ACTIVATED

Target: [project-slug or 'all']
Mode: Scan → Classify → Act → Report
Safety: Archive by default (use --hard-delete to permanently remove)

Scanning docs/...
```

### Scan Logic

1. **Glob** `docs/**/*.md` to find every markdown file (exclude `docs/.archive/`)
2. For each file, collect:
   - Path, line count, byte size
   - Last modified date (from `git log -1 --format=%ai -- [file]` or filesystem mtime)
   - Whether it's a child file (inside a subdirectory) or top-level
   - Parent README.md path (for child files)
3. For child files: check if the parent README.md references this filename
4. Read file contents to extract: changelog dates, status fields, `✅` counts, code path references
5. Compute totals: file count, total bytes, total lines

### Scan Output

```
SCAN COMPLETE

Project: [slug]
Directory: docs/[slug]/
Files: [N] markdown files
Total size: [N]KB / [max-size]KB limit
Largest file: [name] ([N] lines / [max-lines] line limit)
Oldest update: [name] ([N] days ago)
```

---

## Phase 2: Classify

Apply detection rules to every scanned file. Each file gets exactly ONE primary classification (the most severe applicable), plus OVERSIZED as an additive flag.

### Detection Rules

#### Rule 1: BACKUP
```
Pattern: *.bak.md
Action:  DELETE
Reason:  Resume Protocol artifacts, no longer needed after re-do
```

#### Rule 2: EMPTY
```
Detection: Strip blank lines and heading-only lines (^#+\s). If < 5 content lines remain.
Action:    DELETE
Reason:    Empty shells waste directory entries and confuse Context Pass-Through
```

#### Rule 3: ORPHANED
```
Detection: File is inside a subdirectory (e.g., docs/qa-report-auth/bug-003.md).
           Read the parent README.md or master doc.
           If parent does NOT contain the child's filename as a link or reference.
Action:    ARCHIVE
Reason:    Detached from its parent, invisible to normal navigation
```

#### Rule 4: SUPERSEDED (ADRs only)
```
Detection: File matches docs/architecture-*/adr-*.md
           Parse the **Status:** line.
           If status contains "Deprecated" or "Superseded"
Action:    ARCHIVE
Reason:    Decision was replaced -- the superseding ADR has the current truth
```

#### Rule 5: COMPLETED (Spec Shards only)
```
Detection: File matches docs/spec-shard-*.md
           Parse all status icons: ⏳ 🔄 📋 ✅ ❌ ⚠️
           If EVERY item is ✅ (Merged) with zero non-✅ items remaining
Action:    ARCHIVE
Reason:    All work is done and merged -- shard served its purpose
```

#### Rule 6: RESOLVED (QA, Triage, Code Review, UX Review reports)
```
Detection: File matches docs/qa-report-* | docs/triage-log-* | docs/code-review-* | docs/ux-review-*
           Parse findings for resolution markers:
           - QA: all findings have "✅" or "Resolved" in their status
           - Triage: Summary table shows "Open: 0" and "Escalated: 0"
           - Code Review: Verdict line contains "APPROVE"
           - UX Review: all findings marked as implemented
           AND the final section contains "PASS" or "APPROVED"
Action:    ARCHIVE
Reason:    All issues addressed -- keeping full reports wastes context
```

#### Rule 7: STALE
```
Detection: Last git modification date > --max-age days (default 30)
           AND file is NOT a README.md or prd.md (protected)
           AND file is NOT already classified by a more specific rule
Action:    ARCHIVE
Reason:    No updates in [N] days suggests the doc is no longer actively relevant
```

#### Rule 8: STALE_REFERENCES
```
Detection: Read file contents. Extract all source code path references:
           - Patterns like src/*, components/*, lib/*, app/*, pages/*
           - File references like [filename.ts], [filename.tsx], [filename.py]
           Glob-verify each referenced path against the actual project.
           If 50%+ of referenced paths no longer exist.
Action:    COMPRESS (remove dead reference sections, mark [REMOVED])
Reason:    Doc describes code that was deleted or moved
```

#### Rule 9: DUPLICATE
```
Detection: For each pair of files of the same doc type (same prefix):
           Compare heading structure (## and ### lines).
           If 80%+ heading overlap AND 80%+ of non-trivial body lines match.
Action:    ARCHIVE the older/smaller copy
Reason:    Redundant information doubles context cost for zero value
```

#### Rule 10: OVERSIZED (additive -- applies on top of any other classification)
```
Detection: File line count > --max-lines (default 500)
Action:    COMPRESS (even if otherwise CURRENT)
Reason:    Single file consuming too much context
```

#### CURRENT (default)
```
Detection: None of the above rules matched.
Action:    KEEP (no changes)
```

### Protected Files (NEVER archived or deleted)

- `docs/*/README.md` -- checkpoint registry for Resume Protocol
- `docs/*/prd*.md` -- source of truth per pipeline rules
- These CAN be compressed if OVERSIZED, but never removed

### Classification Output

```
CLASSIFICATION COMPLETE

docs/[slug]/qa-report-auth.md         RESOLVED     (12/12 findings ✅)
docs/[slug]/triage-log-auth.md        RESOLVED     (0 open, 0 escalated)
docs/[slug]/architecture/adr-001.md   SUPERSEDED   (by ADR-003)
docs/[slug]/spec-shard-mvp.md         COMPLETED    (8/8 items ✅ Merged)
docs/[slug]/prd-auth.bak.md           BACKUP       (*.bak.md)
docs/[slug]/qa-report-auth/bug-003.md ORPHANED     (not referenced by parent)
docs/[slug]/architecture.md           OVERSIZED    (743 lines > 500 limit)
docs/[slug]/design-system.md          CURRENT      (keep)
docs/[slug]/prd.md                    CURRENT      (protected + keep)
docs/[slug]/README.md                 CURRENT      (protected + keep)

Summary:
  Keep:       [N] files ([N]KB)
  Archive:    [N] files ([N]KB to free)
  Compress:   [N] files (est. [N]KB savings)
  Delete:     [N] files ([N]KB to free)
```

If `--dry-run`: stop here and show what would be done. Do not proceed to Phase 3.

---

## Phase 3: Act

Apply actions in this specific order to avoid broken references:

### Order of Operations

```
1. COMPRESS oversized CURRENT files (changes line counts, must happen first)
2. COMPRESS files with stale references (update inline, remove dead sections)
3. UPDATE parent docs (remove references to files about to be archived/deleted)
4. ARCHIVE files classified for archival
5. DELETE files classified for deletion (backups + empties only)
6. CHECK total directory size -- if still over budget, apply budget enforcement
```

### Archive Mechanism

Create `docs/.archive/` if it doesn't exist. Move files with date prefix:

```
docs/[slug]/qa-report-auth.md           → docs/.archive/2026-04-07-qa-report-auth.md
docs/[slug]/qa-report-auth/bug-003.md   → docs/.archive/2026-04-07-qa-report-auth--bug-003.md
docs/[slug]/architecture/adr-001.md     → docs/.archive/2026-04-07-adr-001-initial-db-choice.md
```

Flatten subdirectory paths using `--` separator.

### Delete Mechanism

Only hard-delete by default:
- `*.bak.md` files (Resume Protocol artifacts)
- Files with < 5 content lines (empty shells)

Everything else archives. If `--hard-delete` flag is present, warn first:

```
WARNING: --hard-delete mode active.
[N] files will be PERMANENTLY deleted (not archived).
This cannot be undone. Proceed? (yes / abort)
```

### Parent Reference Update

When archiving or deleting a child file:
1. Read the parent README.md or master doc
2. Remove markdown links to the archived/deleted file
3. Remove summary table rows referencing the file
4. If a section becomes empty after removal, add: `_No active items._`

### Compression Algorithms

#### Algorithm 1: Changelog Compression

Keep last 3 entries verbatim. Collapse all older entries to one summary line.

**Before:**
```markdown
## Changelog
- 2026-01-15 Initial PRD created
- 2026-01-16 Updated scope based on architect feedback
- 2026-01-18 Added user story for notifications
- 2026-02-01 Revised success metrics per stakeholder input
- 2026-03-15 Added edge case requirements from QA findings
- 2026-04-01 Final scope lock for v1
```

**After:**
```markdown
## Changelog
- 2026-01-15 to 2026-02-01: 4 prior updates (initial creation → revised success metrics)
- 2026-03-15 Added edge case requirements from QA findings
- 2026-04-01 Final scope lock for v1
```

#### Algorithm 2: Resolved Findings Collapse

Keep open findings in full detail. Collapse each resolved finding to one line.

**Before (per finding, ~15 lines):**
```markdown
## [BUG-001] Login Session Expiry
**Severity:** Critical
**Component:** Auth
**Steps to Reproduce:**
1. Log in
2. Wait 30 minutes
3. Attempt action
**Expected:** Session refreshes automatically
**Actual:** User logged out without warning
**Suggested Resolution:** Implement refresh token rotation
**Status:** ✅ Resolved (FIX-001)
```

**After (1 line):**
```markdown
- ~~[BUG-001] Login Session Expiry~~ — Critical, Auth — ✅ Resolved (refresh token rotation)
```

#### Algorithm 3: Completed Shard Condensation

Replace full task cards with compact table rows.

**Before (per item, ~15 lines):**
```markdown
#### 1.1 User Authentication
**Original:** "Add login system"
**Priority:** P0
**Complexity:** L
**Status:** ✅ Merged (PR #42)
> **Commentary:** Build as OAuth + email/password...
> Approach: Use NextAuth.js...
**Execute:** `/propose user authentication`
```

**After (1 table row):**
```markdown
| 1.1 | User Authentication | P0 | L | ✅ PR #42 |
```

#### Algorithm 4: Stale Reference Removal

For docs referencing code paths that no longer exist:
- Glob-verify each `src/*`, `components/*`, `lib/*` path reference
- Remove sections that reference ONLY deleted paths
- Mark deleted components as `[REMOVED]` in component maps (preserves history)
- Add note: `_[N] references to removed code cleaned [date]_`

#### Algorithm 5: Architecture Pruning

For oversized architecture docs:
- Collapse API contract details to endpoint signatures only (remove request/response schemas)
- Collapse data model field details to entity names + relationships only
- Keep ADR references as one-liners (full ADR files remain separate)

#### Algorithm 6: Budget Enforcement

If total `docs/` size still exceeds `--max-size` after targeted compression:

1. Rank CURRENT files by: oldest modified first, then largest first
2. Apply escalating compression:
   - **Level 1:** Changelog compression (Algorithm 1)
   - **Level 2:** Collapse all resolved findings (Algorithm 2)
   - **Level 3:** Remove inline code blocks (replace with `[see source]`)
   - **Level 4:** Archive the file entirely (if still oversized + not modified in 14+ days)
3. Stop as soon as total size is under budget

---

## Phase 4: Verify & Report

### Cleanup Report

```
DOC CLEANUP REPORT
═══════════════════════════════════════════════════

Target: docs/[slug]/
Date: [YYYY-MM-DD]

BEFORE                          AFTER
Files:    [N]                   Files:    [N] (-[N])
Size:     [N]KB                 Size:     [N]KB (-[N]KB, [N]% reduction)
Largest:  [name] ([N] lines)    Largest:  [name] ([N] lines)

ACTIONS TAKEN
  Archived:              [N] files → docs/.archive/
  Deleted:               [N] files (backups + empties)
  Compressed:            [N] files
  Parent refs updated:   [N] docs

LIMITS CHECK
  Per-file max ([max-lines] lines):  [PASS / N files still over]
  Directory max ([max-size]KB):      [PASS / FAIL at NKB]

ARCHIVED FILES (recoverable from docs/.archive/):
  [date]-[name].md — [classification reason]
  [...]

COMPRESSED FILES:
  [name].md: [N] → [N] lines (-[N]%)
  [...]

Est. context tokens freed: ~[N]K
```

### Save the Report

Save to `docs/.archive/cleanup-report-[YYYY-MM-DD].md` so there's a record of what was done. This report is itself in `.archive/` so it doesn't pollute active docs.

---

## Operating Principles

1. **Fix, don't report** -- directly archive, compress, and delete; don't generate a TODO list for humans
2. **Archive before delete** -- every substantive file goes to `.archive/` first; only empties and backups are hard-deleted
3. **Context efficiency is the goal** -- every byte removed from `docs/` frees context tokens for actual code analysis
4. **Preserve outcomes, remove process** -- keep WHAT was decided; collapse HOW it was decided
5. **Respect protected files** -- README.md and PRD are never archived; they can be compressed but never removed
6. **Idempotent** -- running twice produces the same result; already-clean docs are CURRENT and skipped
7. **Fast and direct** -- no discovery questions, no plan mode; scan, classify, act, report
8. **Compression preserves meaning** -- a collapsed finding still tells you what was found and how it was fixed; it just doesn't waste 15 lines doing it

## Document Output

| Output | File Location |
|--------|---------------|
| Cleanup report | `docs/.archive/cleanup-report-[date].md` |

Report lives in `.archive/` to avoid adding to the doc bloat it's cleaning up.

## Cross-Cutting Protocols

### Git Workflow
- **Commit tag:** `docs`
- **Commit when:** After cleanup completes and report is generated
- **Commit format:** `docs: cleanup [N] stale docs, archive [N], compress [N] (-[N]KB)`
- **Branch:** Commit to current branch; if on main and 10+ files change, suggest `chore/doc-cleanup-[date]`
- **Pre-check:** If uncommitted changes to doc files exist, warn and ask whether to proceed

### Resume Protocol
- **On start:** Check for `docs/.archive/cleanup-report-*.md`
- **If recent (< 7 days):** "Last cleanup was [N] days ago. Run again? (yes / skip)"
- **Prevents over-cleaning**

### Context Loading
- **Before scanning:** Read `docs/*/README.md` to understand project structure
- **Read PROTOCOLS.md** patterns to know which files Context Pass-Through searches for
- **Critical:** Never archive a file that other skills actively search for during Context Pass-Through, unless it's fully resolved

### Smart Skip
- Not applicable -- this skill doesn't ask discovery questions

---

## Pipeline Integration

### When to Run

| Trigger | Context |
|---------|---------|
| **Post-pipeline** | After Phase 10 completes: "Pipeline done. Run /doc-cleanup to archive resolved reports?" |
| **Standalone** | User runs `/doc-cleanup` anytime as maintenance |
| **Pre-pipeline** | Before `/propose` when Context Pass-Through is slow due to bloated docs |
| **Scheduled** | Recommended: run weekly or after every 3 pipeline completions |

### Output: What You Hand Off

| Hand Off To | When | What to Provide |
|-------------|------|-----------------|
| **All skills** | After cleanup | Leaner `docs/`, faster Context Pass-Through |
| **propose** | After cleanup | Updated README.md with cleaned references |

### Input: What You Receive

| Receive From | When | What They Provide |
|--------------|------|-------------------|
| **propose** | Post-pipeline | Suggestion to run cleanup |
| **Any skill** | When context loading is slow | User invokes manually |

### Inter-Skill Communication Protocol

```
RECEIVE TARGET (docs/ directory or project slug)
  |
  v
SCAN all docs/*.md (exclude .archive/)
  |  Collect: path, size, lines, last modified, parent-child, contents
  |
  v
CLASSIFY each file
  |  BACKUP → DELETE
  |  EMPTY → DELETE
  |  ORPHANED → ARCHIVE
  |  SUPERSEDED → ARCHIVE
  |  COMPLETED → ARCHIVE
  |  RESOLVED → ARCHIVE
  |  STALE → ARCHIVE
  |  STALE_REFS → COMPRESS
  |  DUPLICATE → ARCHIVE one
  |  OVERSIZED → COMPRESS (additive)
  |  CURRENT → KEEP
  |
  v
ACT (compress → update parents → archive → delete → budget enforce)
  |
  v
REPORT (before/after metrics, token savings, archived file list)
  |
  v
COMMIT (atomic: all changes in one commit)
```
