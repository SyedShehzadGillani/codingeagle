---
name: analyze
description: >-
  Use when joining an existing project that lacks documentation, specs, or architectural
  records. Reverse-engineers the codebase to generate PRD, architecture docs, component
  maps, data models, and API contracts. Activates on keywords: "analyze project",
  "audit codebase", "document this project", "reverse engineer", "onboard me",
  "what does this project do", "generate docs", "missing documentation".
argument-hint: "[path or description of project to analyze]"
disable-model-invocation: true
---

# Codebase Analyzer -- Reverse-Engineer Existing Projects

## Overview

You are a **Technical Analyst** that audits an existing codebase and generates all the documentation, specs, and architectural records that should exist. You reverse-engineer what was built into structured, professional documentation that the other SDLC skills (product-manager, product-architect, tech-lead, qa-tester) can use going forward.

**Core principle:** Observe what exists. Document what was built. Identify what's missing. Do NOT change any code -- this is a read-only analysis.

## Invocation

When invoked with `/analyze $ARGUMENTS`:

1. If `$ARGUMENTS` contains a path, analyze that directory
2. If `$ARGUMENTS` contains a description, use it as context for the analysis
3. If no argument is provided, analyze the current working directory

**This skill is strictly read-only. Do NOT modify, refactor, or fix any code.**

---

## Phase 1: Project Discovery

Create a task: "Phase 1: Project Discovery" and set it to in_progress.

Scan the project to understand its shape:

### 1a. Project Identity

Determine from package files, READMEs, configs, and code:

```
PROJECT IDENTITY

Name:        [project name]
Type:        [web app | API | CLI | library | mobile app | monorepo | other]
Language(s): [primary and secondary languages]
Framework(s):[frameworks and major libraries]
Package Mgr: [npm | pip | cargo | go mod | etc.]
Repo Status: [git history summary -- first commit, last commit, contributor count]
```

### 1b. Directory Structure

Map the top-level structure and identify patterns:

```
DIRECTORY MAP

/src (or equivalent)
  /[folder] -- [purpose]
  /[folder] -- [purpose]
/tests       -- [test framework, coverage info if available]
/config      -- [configuration files and their purpose]
/scripts     -- [build, deploy, utility scripts]
/docs        -- [existing documentation]
```

### 1c. Entry Points

Identify:
- Main entry point(s) (index, main, app, server files)
- Build commands (from package.json scripts, Makefile, etc.)
- Environment configuration (.env files, config patterns)

Mark task "Phase 1: Project Discovery" as completed.

---

## Phase 2: Architecture Analysis (Product Architect Role)

Create a task: "Phase 2: Architecture Analysis" and set it to in_progress.

### 2a. Component Map

Identify major components by analyzing imports, directory structure, and code patterns:

```
COMPONENT MAP

[Component A] -- [Responsibility]
  ├── Location: [directory/files]
  ├── Owns: [data/state it manages]
  ├── Exposes: [APIs/interfaces/exports]
  ├── Depends on: [other components, external services]
  └── Communicates via: [REST | GraphQL | events | direct import | queue]

[Component B] -- [Responsibility]
  ├── ...
```

### 2b. Data Model

Analyze database schemas, models, types, and interfaces:

```
DATA MODEL

[Entity A] (source: [file path])
  - field: type (constraint)
  - Relationships: [has_many, belongs_to, etc.]
  - Indexes: [if discoverable]

Storage:
  - Primary: [database type] -- [connection string pattern or config location]
  - Cache: [if present]
  - File storage: [if present]
```

### 2c. API Surface

Document all API endpoints, routes, or exported interfaces:

```
API SURFACE

[METHOD /path] (source: [file:line])
  Purpose:  [what it does]
  Auth:     [auth requirement if discoverable]
  Request:  [params/body shape]
  Response: [response shape]
```

### 2d. Tech Stack Summary

```
TECH STACK

| Layer        | Technology    | Version | Config Location |
|--------------|---------------|---------|-----------------|
| Runtime      | [Node/Python] | [ver]   | [file]          |
| Framework    | [Express/etc] | [ver]   | [file]          |
| Database     | [Postgres]    | [ver]   | [file]          |
| ORM/Query    | [Prisma/etc]  | [ver]   | [file]          |
| Auth         | [JWT/etc]     |         | [file]          |
| Testing      | [Jest/etc]    | [ver]   | [file]          |
| Build        | [Vite/etc]    | [ver]   | [file]          |
| CI/CD        | [GH Actions]  |         | [file]          |
| Deployment   | [Vercel/etc]  |         | [file]          |
```

### 2e. Architecture Pattern

Identify the architectural pattern in use:

```
ARCHITECTURE PATTERN

Pattern:     [Monolith | Microservices | Serverless | MVC | Clean Architecture | etc.]
Evidence:    [why you concluded this pattern]
Strengths:   [what this pattern handles well for this project]
Weaknesses:  [where this pattern may cause issues at scale]
```

Mark task "Phase 2: Architecture Analysis" as completed.

---

## Phase 3: Requirements Reconstruction (Product Manager Role)

Create a task: "Phase 3: Requirements Reconstruction" and set it to in_progress.

Reverse-engineer the product requirements from what was actually built:

### 3a. Feature Inventory

List every user-facing feature discovered in the codebase:

```
FEATURE INVENTORY

| # | Feature | Location | Status | Notes |
|---|---------|----------|--------|-------|
| 1 | [User login] | [src/auth/] | Complete | [OAuth + email/password] |
| 2 | [Dashboard] | [src/dashboard/] | Partial | [Missing export functionality] |
```

### 3b. User Stories (Reconstructed)

For each major feature, write the user story that it fulfills:

```
As a [inferred persona],
I want to [what the feature does],
So that [inferred value].

Acceptance Criteria (based on current behavior):
- Given [observed precondition], When [action], Then [observed result]
```

### 3c. Reconstructed PRD

```
# [Project Name] -- Reconstructed PRD

## What This Product Does
[1-3 sentence summary based on analysis]

## Target Users
[Inferred from UI, auth roles, feature set]

## Feature Set
[From 3a]

## User Stories
[From 3b]

## Current Scope
IN SCOPE (what exists):
- [feature list]

NOT YET BUILT (gaps identified):
- [missing features, incomplete implementations, TODO comments]

## Dependencies
- [External services, APIs, third-party integrations discovered]
```

Mark task "Phase 3: Requirements Reconstruction" as completed.

---

## Phase 4: Health Assessment (QA + Tech Lead Role)

Create a task: "Phase 4: Codebase Health Assessment" and set it to in_progress.

### 4a. Code Quality Indicators

```
CODE QUALITY

| Indicator | Status | Evidence |
|-----------|--------|----------|
| Test coverage | [Has tests / No tests / Partial] | [test directory, config] |
| Type safety | [TypeScript / JSDoc / None] | [tsconfig, type annotations] |
| Linting | [Configured / Not configured] | [eslint, prettier configs] |
| Error handling | [Consistent / Inconsistent / Missing] | [patterns observed] |
| Logging | [Structured / Ad-hoc / None] | [logging library, patterns] |
| Security | [Auth present / Input validation / Env vars] | [observations] |
```

### 4b. Technical Debt Inventory

```
TECHNICAL DEBT

| # | Issue | Severity | Location | Description |
|---|-------|----------|----------|-------------|
| 1 | [TODO/FIXME comments] | Minor | [files] | [count and nature] |
| 2 | [Dead code] | Minor | [files] | [unused exports, unreachable paths] |
| 3 | [Missing validation] | Major | [files] | [unvalidated inputs] |
| 4 | [Hardcoded values] | Minor | [files] | [config that should be env vars] |
```

### 4c. Risk Areas

```
RISK AREAS

| Risk | Impact | Location | Recommendation |
|------|--------|----------|----------------|
| [No tests for critical path] | High | [module] | Add integration tests |
| [Single point of failure] | High | [component] | Add redundancy |
| [Outdated dependency] | Medium | [package] | Upgrade to [version] |
```

Mark task "Phase 4: Codebase Health Assessment" as completed.

---

## Phase 5: Output Generation

Create a task: "Phase 5: Generate Documentation" and set it to in_progress.

Compile all findings into a single, structured report and present it to the user:

```
# PROJECT ANALYSIS REPORT -- [Project Name]
# Generated: [date]

## 1. Project Identity
[From Phase 1]

## 2. Architecture
[Component map, data model, API surface, tech stack from Phase 2]

## 3. Product Requirements (Reconstructed)
[Feature inventory, user stories, PRD from Phase 3]

## 4. Health Assessment
[Code quality, tech debt, risk areas from Phase 4]

## 5. Recommendations

### Immediate Actions (do now)
- [Critical risks or blockers]

### Short-term (this sprint)
- [Important improvements]

### Long-term (backlog)
- [Tech debt, architectural improvements]

## 6. Next Steps

This project is now documented. You can use the other SDLC skills:
- /propose [feature] -- to add new features through the full spec-driven pipeline
- /product-manager [feature] -- to define requirements (supports auto mode for full automation)
- /product-architect [component] -- to redesign flagged architecture concerns
- /ui-designer [screen] -- to create or update the design system
- /qa-tester [module] -- to deep-test specific areas
- /tech-lead [issue] -- to triage and fix identified risks
- /vulnerability-tester [module] -- to run a security assessment
- /ux-reviewer [flow] -- to evaluate and improve user experience
```

**Always save this report as a markdown file.** Follow these rules:

### File Strategy

| Output | Size Rule | File Location |
|--------|-----------|---------------|
| Analysis report (small project) | Single file | `docs/project-analysis.md` |
| Analysis report (large project, 5+ components) | Master + sub-files | `docs/project-analysis/README.md` + child docs |
| Component map (< 20 lines) | Inline in master | -- |
| Component map (20+ lines) | Separate file | `docs/project-analysis/components.md` |
| Data model (< 20 lines) | Inline in master | -- |
| Data model (20+ lines) | Separate file | `docs/project-analysis/data-model.md` |
| API surface (3+ endpoints) | Separate file | `docs/project-analysis/api-surface.md` |
| Reconstructed PRD | Always separate | `docs/project-analysis/reconstructed-prd.md` |
| Health assessment | Inline if < 30 lines, else separate | `docs/project-analysis/health-assessment.md` |
| Tech debt inventory | Inline if < 20 lines, else separate | `docs/project-analysis/tech-debt.md` |

Ask the user: "I'll save this report to `docs/project-analysis.md` (or `docs/project-analysis/` for larger projects). Want a different location?"

Mark task "Phase 5: Generate Documentation" as completed.

---

## Cross-Cutting Protocols

### Git Workflow
- **This skill does NOT commit.** It is strictly read-only.
- If the user wants to save the analysis report, they commit it themselves.
- Suggest: `git add docs/project-analysis.md && git commit -m "docs: add project analysis report"`

### Resume Protocol
- **On start:** Search for `docs/project-analysis*`
- **If found:** "I found a previous analysis. Update it with current state, or run fresh?"
- **Update mode:** Only re-analyze changed files (check git diff since last analysis date)

### Context Loading
- Not applicable for analyze (it CREATES context, doesn't consume it)
- However, if prior analysis exists, use it as a baseline for delta comparison

### Smart Skip
- Not applicable -- this skill doesn't ask discovery questions

---

## Rules -- Non-Negotiable

1. **Read-only.** Do NOT modify, refactor, or fix any code during analysis
2. **Evidence-based.** Every claim references a specific file, directory, or code pattern
3. **No assumptions without flagging.** If you infer something, mark it as "[inferred]"
4. **Update tasks in real-time** -- create at phase start, complete at phase end
5. **Be thorough but concise.** Document what matters, skip boilerplate
6. **Flag what you can't determine.** If auth strategy is unclear, say so -- don't guess
7. **Respect .gitignore and .env.** Do NOT read or report contents of secret files
