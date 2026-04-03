# CodingEagle Plugin -- Session Notes

## What We Built

10 skills packaged as the `codingeagle` plugin at `~/.claude/plugins/codingeagle/`:

| Skill | Purpose |
|-------|---------|
| `codingeagle:product-manager` | Defines requirements, PRDs, user stories. Plan mode with 10 discovery questions asked one at a time. Supports **Auto Mode** that chains all skills automatically. Manual invoke only. |
| `codingeagle:product-architect` | System design, tech stack, data models, API contracts, ADRs. Plan mode with 10 architecture questions. Recommends options with trade-offs, user chooses. |
| `codingeagle:ui-designer` | Visual design system, color schemes, typography, component specs. Plan mode with 10 design questions. Shows 3 direction options. Accepts screenshots/logos/visual references. Comes after product-architect. |
| `codingeagle:tech-lead` | Triages QA findings, root-cause analysis, implements fixes, code review. Plan mode with 7 diagnostic questions. |
| `codingeagle:qa-tester` | Functional testing with 4-pillar framework. Plan mode with 7 discovery questions. Includes full feedback loop (find -> fix -> re-verify, max 3 iterations). |
| `codingeagle:vulnerability-tester` | Professional ethical hacker. 10-category OWASP-aligned security assessment. CVSS scoring. Plan mode with 7 scoping questions. Feeds findings to tech-lead for remediation. |
| `codingeagle:ux-reviewer` | Heuristic evaluation (Nielsen's 10) + WCAG 2.1 accessibility audit. Plan mode with 8 discovery questions. Prioritized improvement tiers. User selects what to apply. |
| `codingeagle:spec-sharder` | Preprocessor that takes scattered ideas, brain dumps, or large tasks and organizes them into structured, prioritized task lists with AI commentary. Feeds into /propose. Two modes: organize (scattered points) and shard (big task decomposition). Manual invoke only. |
| `codingeagle:propose` | Full spec-driven pipeline: Spec Shard (optional) -> Discovery -> PRD -> Architecture -> UI Design -> Implementation Plan -> Build+QA Loop -> Vulnerability Testing -> UX Review -> Final Verification. Supports Manual (9 gates) and Auto mode. |
| `codingeagle:analyze` | Reverse-engineers existing projects (read-only). Generates project identity, component map, data model, reconstructed PRD, health assessment. References all 9 other skills. |

## Key Design Decisions

- **Skills over subagents** for the SDLC pipeline because roles need shared conversation context for iterative feedback loops
- **Subagents** are better for parallel independent work (e.g., fixing 3 bugs at once, searching multiple directories)
- All skills support **$ARGUMENTS** (e.g., `/codingeagle:qa-tester src/auth/login.tsx`)
- All skills are **interconnected** with handoff protocols, escalation triggers, and inter-skill communication protocols
- All skills start in **plan mode** and ask **extensive discovery questions** one at a time before taking action
- All skills **recommend options with trade-offs** but let the user make the final decision
- `product-manager` has `disable-model-invocation: true` so Claude won't auto-trigger it
- `propose` has `disable-model-invocation: true` -- manual invoke only
- `analyze` has `disable-model-invocation: true` -- manual invoke only

## Document Output System

All skills create and maintain markdown files following these rules:
- **Small outputs** (< 15 lines per item) go inline in the master doc
- **Large outputs** get their own `.md` file, referenced from the master doc
- **Update, never recreate** -- modify existing docs in place with changelog entries
- **Reference, don't duplicate** -- link to other skill docs instead of copying content
- Each skill has its own file naming convention (see "Document Output" section in each SKILL.md)
- The propose pipeline creates a full `docs/[project-slug]/` directory with master README linking all phase docs

## Spec Sharder

The spec-sharder sits BEFORE all other skills in the pipeline:
```
Raw ideas / brain dump -> spec-sharder -> organized task list -> /propose [item]
```
- Two modes: Organize (scattered points) and Shard (decompose big task)
- Every item gets AI commentary: approach, feasibility, complexity, dependencies, risks
- Output is a markdown file with `/propose` commands ready to execute
- Can merge new points into existing shards

## Auto Mode

Product Manager and Propose pipeline both support Auto Mode:
- User answers discovery questions (can't automate understanding the problem)
- All subsequent phases run automatically without approval gates
- Only pauses on CRITICAL issues or UX Tier 2+ improvements
- Uses sensible defaults (boring tech, WCAG AA, standard security assessment)

## Role Clarifications

- **Product Manager** vs **Product Architect**: PM owns the *problem* (what/why), Architect owns the *solution design* (how)
- **Product Architect** vs **UI Designer**: Architect defines structure and tech stack, Designer defines visual system within those constraints
- **UI Designer** vs **UX Reviewer**: Designer creates the design system proactively, UX Reviewer evaluates what was built against usability principles
- **Product Architect** vs **Tech Lead**: Architect prevents future problems (designs systems), Tech Lead solves current ones (fixes bugs)
- **QA Tester** vs **Vulnerability Tester**: QA finds functional bugs, Vulnerability Tester finds security bugs
- **Developer** is Claude's default behavior -- skills add the surrounding roles

## Inter-Skill Communication

Every skill has a defined Inter-Skill Communication Protocol that specifies:
1. What it receives from other skills (and when)
2. What it hands off to other skills (and when)
3. Escalation triggers (when to involve other skills)
4. Feedback loops (how fixes cycle back through verification)

Key communication paths:
```
PM -> Architect -> UI Designer -> Implementation -> QA -> Tech Lead
                                                    |
                                            Vulnerability Tester
                                                    |
                                              UX Reviewer
```

## Skills vs Subagents vs Custom Agents

- **Skills** (`~/.claude/skills/`): Change Claude's mindset in same conversation. Best for iterative work.
- **Custom Subagents** (`~/.claude/agents/`): Separate Claude instances with own context, tools, model. Best for isolation and parallel work.
- **Custom Agents (SDK)**: Building your own AI apps with Anthropic SDK. Completely different thing.

## Planned Future Skills (Not Yet Built)

**Tier 1 (next to build):**
- `code-reviewer` -- PR/diff review for quality, security, SOLID
- `devops` -- Dockerfiles, CI/CD, deployment, infra-as-code
- `api-designer` -- Deep API design (REST/GraphQL)

**Tier 2:**
- `db-engineer` -- Schema optimization, query performance, indexing
- `technical-writer` -- API docs, READMEs, changelogs

**Tier 3:**
- `performance-analyst` -- Bundle size, render perf, caching
- `refactorer` -- Code smells, design patterns, complexity reduction

**Tier 4:**
- `standup` -- Daily standup from git history
- `changelog` -- Release notes from commits
- `onboard` -- Explain codebase to new developers
