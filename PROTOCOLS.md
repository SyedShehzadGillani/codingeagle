# CodingEagle -- Cross-Cutting Protocols

These protocols apply to ALL skills. Each skill includes a compact reference; this document is the authoritative source.

---

## 1. Git Workflow Protocol

### Branch Strategy

- **Pipeline work** (via `/propose`): Create feature branch at Phase 5 (before implementation)
  - Branch name: `feature/[project-slug]`
  - All implementation, QA fixes, security fixes, UX improvements, and performance optimizations commit to this branch
  - Phase 9 (Final Verification) offers to create a PR or merge

- **Standalone skill work**: Check if on a feature branch
  - If yes: commit to current branch
  - If no and changes are non-trivial: suggest creating a branch first
  - If no and changes are trivial (single fix): commit to current branch with user confirmation

### Commit Convention

Format: `[skill-tag](scope): description`

| Skill | Tag | Example |
|-------|-----|---------|
| product-manager | `prd` | `prd: define requirements for user auth` |
| product-architect | `arch` | `arch: design system with PostgreSQL + Express` |
| ui-designer | `design` | `design: create design system tokens and component specs` |
| tech-lead | `fix` | `fix(auth): resolve session expiry race condition` |
| qa-tester | `qa` | `qa: add test coverage for login edge cases` |
| code-reviewer | `review` | `review: address spec drift in user profile endpoint` |
| vulnerability-tester | `security` | `security: remediate XSS in profile page` |
| performance-tester | `perf` | `perf: add database index for orders query` |
| ux-reviewer | `ux` | `ux: improve error messages and loading states` |
| propose | `feat` | `feat: complete user authentication system` |
| spec-sharder | `docs` | `docs: create spec shard for project roadmap` |
| analyze | -- | Does not commit (read-only) |

### Commit Triggers (when each skill should commit)

| Skill | Commit When |
|-------|-------------|
| product-manager | PRD file created/updated |
| product-architect | Architecture docs created/updated |
| ui-designer | Design system docs created/updated |
| tech-lead | After each fix is implemented and tested |
| qa-tester | After QA report is written (does not commit code fixes -- tech-lead does) |
| code-reviewer | After review report is written (does not commit fixes -- tech-lead does) |
| vulnerability-tester | After security report written AND after each security fix |
| performance-tester | After performance report written AND after each optimization |
| ux-reviewer | After UX report written AND after each UX improvement |
| propose | Delegates to active role's commit trigger |
| spec-sharder | After shard document created/updated |
| analyze | Never commits (read-only) |

### PR Creation

At pipeline completion (Phase 9 or standalone skill completion):

```
GIT STATUS

Branch: feature/[slug]
Commits: [N] commits across [N] phases
Files changed: [N]

Options:
1. Create Pull Request (recommended)
   -> Opens PR with summary from delivery report
2. Merge to main
   -> Direct merge (only if solo developer, no review needed)
3. Keep branch
   -> Leave as-is for manual review
```

---

## 2. Resume / Checkpoint Protocol

### How It Works

Every skill phase writes its status to the project's master README. When any skill starts, it checks for existing docs.

### Checkpoint Format

The project README (`docs/[project-slug]/README.md`) contains a phase status table:

```markdown
| Phase | Status | Doc | Checkpoint |
|-------|--------|-----|------------|
| 1. Discovery | Complete | [inline] | 2026-04-03T14:30:00Z |
| 2. PRD | Complete | [prd.md](./prd.md) | 2026-04-03T14:45:00Z |
| 3. Architecture | Complete | [architecture.md](./architecture.md) | 2026-04-03T15:10:00Z |
| 4. UI Design | In Progress | [design-system.md](./design-system.md) | 2026-04-03T15:30:00Z |
| 5. Implementation | Pending | -- | -- |
```

### Resume Detection

When a skill starts:

1. **Search for project docs**: look for `docs/*/README.md` matching the project/feature name
2. **If found**: read the phase status table
3. **If the current skill's phase is already `Complete`**:
   ```
   CHECKPOINT FOUND

   I found existing output from a previous session:
   - [Phase name]: completed on [date]
   - Document: [file path]

   Options:
   1. Resume from next incomplete phase (recommended)
   2. Re-do this phase (existing doc will be archived as [name].bak.md)
   3. Start completely fresh (all existing docs archived)
   ```
4. **If the current skill's phase is `In Progress`**: load the partial output and continue from where it left off
5. **If not found**: start fresh, create the project directory and README

### Checkpoint Writing

After completing work, every skill MUST:
1. Update the phase status in the project README to `Complete`
2. Add the timestamp
3. Add the document link

### Standalone Skill Resume

When a skill is invoked standalone (not via `/propose`):
1. Check if any project docs exist in `docs/`
2. If related docs found: "I found existing project docs. Want me to use them as context? (yes/no)"
3. If yes: load context and use smart skip for questions already answered

---

## 3. Context Pass-Through Protocol

### How It Works

Skills read existing project documentation before asking questions. This prevents re-asking what other skills have already established.

### Context Loading Sequence

When a skill starts, before asking discovery questions:

```
1. Search: docs/*/README.md, docs/*-prd*.md, docs/*-architecture*.md,
           docs/*-design-system*.md, docs/*-qa*.md, docs/*-security*.md
2. Read found docs
3. Extract key decisions into a context map:

CONTEXT MAP (internal, not shown to user)
  user_persona:     [extracted from PRD or PM discovery]
  problem:          [extracted from PRD]
  tech_stack:       [extracted from architecture]
  framework:        [extracted from architecture]
  database:         [extracted from architecture]
  design_direction: [extracted from design system]
  color_scheme:     [extracted from design system]
  accessibility:    [extracted from design system or UX review]
  platform:         [extracted from PRD]
  auth_model:       [extracted from PRD or architecture]
  known_issues:     [extracted from QA or security reports]
  performance_targets: [extracted from PRD or architecture]
```

### Context Output

After completing work, every skill MUST ensure its output documents contain all key decisions in a scannable format. Use consistent headings so other skills can find information:

- **PRD docs**: must have clear `## User Stories`, `## Scope`, `## Success Metrics` sections
- **Architecture docs**: must have clear `## Tech Stack`, `## Component Map`, `## Data Model` sections
- **Design docs**: must have clear `## Design Tokens`, `## Color Palette`, `## Typography` sections
- **QA/Security/Perf reports**: must have clear `## Findings Summary`, `## Recommendation` sections

### Cross-Referencing

When a skill uses information from another skill's docs:
- Reference the source: "Based on the architecture doc, the tech stack is Next.js + PostgreSQL"
- Don't copy large sections -- link to them
- If the information seems outdated, flag it: "The architecture doc says X, but I see Y in the code. Which is current?"

---

## 4. Smart Skip Protocol

### How It Works

For each discovery question, check if the answer already exists in loaded context before asking.

### Skip Flow

```
For each discovery question:
  1. Check context map for answer
  2. If found:
     "Based on [source doc], I already know:
      [topic]: [extracted answer]
      Is this still correct? (yes / modify)"
  3. If user says "yes": record answer, skip to next question
  4. If user says "modify": ask the full question
  5. If not found in context: ask the question normally
```

### Rules

- **Never silently skip** -- always show the user what you're skipping and let them correct
- **Batch skips when possible** -- if 5 of 10 questions have answers, show all 5 at once for confirmation
- **Fresh invocations skip nothing** -- if no project docs exist, ask every question
- **Stale context warning** -- if docs are > 7 days old, warn: "These docs are from [date]. Answers may be outdated."
- **User override** -- if the user says "ask me everything" or "don't skip", respect it

### Batch Skip Format

When multiple questions can be skipped:

```
CONTEXT LOADED -- SMART SKIP

I found answers to several questions from existing project docs:

From PRD (docs/project/prd.md):
  1. Target User: Senior developers building web apps
  2. Platform: Web app, desktop + mobile responsive
  3. Auth Model: OAuth (Google, GitHub) + email/password

From Architecture (docs/project/architecture.md):
  4. Tech Stack: Next.js 14 + PostgreSQL + Tailwind CSS
  5. Hosting: Vercel

All correct? (yes / let me modify some / ask me everything fresh)
```
