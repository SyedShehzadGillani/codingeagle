# CodingEagle Plugin -- Session Notes

## What We Built

11 skills + 1 shared protocol doc packaged as the `codingeagle` plugin at `~/.claude/plugins/codingeagle/`:

| Skill | Purpose |
|-------|---------|
| `codingeagle:product-manager` | Defines requirements, PRDs, user stories. Plan mode with 10 discovery questions. Supports **Auto Mode** that chains all skills automatically. Manual invoke only. |
| `codingeagle:product-architect` | System design, tech stack, data models, API contracts, ADRs. Plan mode with 10 architecture questions. Recommends options with trade-offs. |
| `codingeagle:ui-designer` | Visual design system, color schemes, typography, component specs. Plan mode with 10 design questions. Shows 3 direction options. Accepts screenshots/logos. |
| `codingeagle:tech-lead` | Triages QA/security findings, root-cause analysis, implements fixes, code review. Plan mode with 7 diagnostic questions. |
| `codingeagle:qa-tester` | Functional testing with 4-pillar framework. Plan mode with 7 discovery questions. Full feedback loop (find -> fix -> re-verify, max 3 iterations). |
| `codingeagle:code-reviewer` | Reviews implementation against PRD, architecture, and design system. 5-dimension review (spec compliance, architecture, design, quality, security). Catches spec drift. |
| `codingeagle:performance-tester` | Frontend (Core Web Vitals, bundle size), backend (API latency, N+1 queries), database (indexes, pagination), infrastructure (CDN, caching). |
| `codingeagle:ux-reviewer` | Heuristic evaluation (Nielsen's 10) + WCAG 2.1 accessibility audit. Prioritized improvement tiers. User selects what to apply. |
| `codingeagle:spec-sharder` | Preprocessor: takes scattered ideas or big tasks, organizes into structured, prioritized task lists with AI commentary. Feeds into /propose. |
| `codingeagle:propose` | Full 11-phase SDLC pipeline with git workflow. Manual (11 gates) and Auto mode. Resume protocol for interrupted sessions. |
| `codingeagle:analyze` | Reverse-engineers existing projects (read-only). Generates project identity, component map, data model, reconstructed PRD, health assessment. |

## Pipeline Phases

```
Phase 0 (optional): Spec Sharder
Phase 1: Product Discovery (PM)
Phase 2: PRD (PM)
Phase 3: System Architecture (Architect)
Phase 4: UI Design (UI Designer)
Phase 5: Implementation Plan (Tech Lead)
Phase 5.5: Git Setup (create feature branch)
Phase 6: Build + QA Loop (Developer + QA + Tech Lead)
Phase 7: Code Review (Code Reviewer) -- NEW
Phase 8: Performance Testing (Performance Tester)
Phase 9: UX Review (UX Reviewer)
Phase 10: Final Verification & Delivery (All Roles + Git PR/Merge)
```

## Cross-Cutting Protocols (PROTOCOLS.md)

Four protocols applied to ALL 12 skills:

### 1. Git Workflow
- Feature branch created at Phase 5.5 (`feature/[project-slug]`)
- Each skill has a commit tag and commit trigger
- Commits after each task/fix/optimization
- PR or merge offered at Phase 11
- Commit format: `[tag](scope): description`

### 2. Resume / Checkpoint Protocol
- Each phase writes status to `docs/[project-slug]/README.md`
- On start: check for prior session docs
- If found: offer to resume from last completed phase
- Supports partial phase recovery (continue from where it left off)

### 3. Context Pass-Through
- Skills read existing project docs BEFORE asking questions
- Extract key decisions into an internal context map
- Cross-reference: link to other skills' docs, don't duplicate
- Each skill ensures its output is scannable by other skills

### 4. Smart Skip
- Before each discovery question: check if answer exists in loaded context
- If found: "Based on [source], I know [answer]. Correct? (yes/modify)"
- Batch skip when multiple answers available
- Never silently skip -- user always confirms
- Each skill defines which questions can be skipped and which never skip

## Document Output System

All skills create and maintain markdown files:
- **Small outputs** (< 15 lines) go inline in the master doc
- **Large outputs** get their own `.md` file, referenced from master
- **Update, never recreate** -- modify in place with changelog entries
- **Reference, don't duplicate** -- link to other skills' docs
- Propose pipeline creates full `docs/[project-slug]/` directory

## Key Design Decisions

- **Skills over subagents** for the SDLC pipeline (shared conversation context)
- **Subagents** for parallel independent work
- All skills support **$ARGUMENTS**
- All skills start in **plan mode** with extensive discovery questions
- All skills are **interconnected** with handoff protocols, escalation triggers, and inter-skill communication
- All skills have **Cross-Cutting Protocols** section (git, resume, context, smart skip)
- `product-manager`, `propose`, `spec-sharder`, `analyze` have `disable-model-invocation: true`

## Auto Mode

Product Manager and Propose pipeline both support Auto Mode:
- User answers discovery questions (can't automate understanding)
- All subsequent phases run automatically without approval gates
- Pauses on CRITICAL issues or UX Tier 2+ improvements
- Uses sensible defaults (boring tech, WCAG AA, standard assessments)

## Role Clarifications

- **Product Manager** vs **Product Architect**: PM owns the *problem* (what/why), Architect owns the *solution design* (how)
- **Product Architect** vs **UI Designer**: Architect defines structure and tech stack, Designer defines visual system within those constraints
- **UI Designer** vs **UX Reviewer**: Designer creates the design system proactively, UX Reviewer evaluates what was built
- **Code Reviewer** vs **Tech Lead**: Reviewer catches spec drift and quality issues, Tech Lead actually fixes them
- **QA Tester** vs **Performance Tester**: QA = functional correctness, Performance = speed/efficiency
- **Spec Sharder** vs **Product Manager**: Sharder organizes chaos into structure, PM turns structure into specs
- **Developer** is Claude's default behavior -- skills add the surrounding roles

## Planned Future Skills

**Tier 1:**
- `devops` -- Dockerfiles, CI/CD, deployment, infra-as-code
- `api-designer` -- Deep API design (REST/GraphQL)

**Tier 2:**
- `db-engineer` -- Schema optimization, query performance, indexing
- `technical-writer` -- API docs, READMEs, changelogs

**Tier 3:**
- `refactorer` -- Code smells, design patterns, complexity reduction
- `standup` -- Daily standup from git history
- `changelog` -- Release notes from commits
- `onboard` -- Explain codebase to new developers
