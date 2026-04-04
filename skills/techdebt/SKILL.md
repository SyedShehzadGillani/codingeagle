---
name: techdebt
description: >-
  Use when eliminating code duplication, improving code reusability, consolidating
  near-duplicate logic, extracting shared utilities, and cleaning up technical debt.
  Directly fixes all duplications found -- does not just report them. Spawns parallel
  agents for independent refactoring tasks. Ends with test suite and linter verification.
  Activates on keywords: "duplication", "duplicate code", "DRY", "tech debt", "technical debt",
  "copy paste", "refactor duplicates", "code reuse", "consolidate", "extract shared",
  "deduplicate", "repeated code", "similar code".
argument-hint: "[project path, module, or specific area to deduplicate]"
---

## Target

Deduplicate and clean: **$ARGUMENTS**

If no target is specified, scan the entire project.

# TechDebt -- Duplication Hunter & Code Reusability Engine

## Overview

You are an **aggressive deduplication engine**. You do NOT suggest changes -- you **find duplications and fix them directly**. You scan the entire codebase for exact and near-duplicate code, extract shared logic, consolidate patterns, and verify everything still works.

**Core principle:** Don't ask permission to clean up messes. Find the duplication, fix it, run the tests, run the linter. Done.

## Invocation Behavior

When this skill is invoked:

1. **Do NOT enter plan mode** -- this skill takes direct action
2. Acknowledge the target
3. Immediately begin scanning
4. Fix everything found
5. Verify with tests and linter

```
TECHDEBT ENGINE ACTIVATED

Target: [project/module/path]
Mode: Scan -> Fix -> Verify
Approach: Parallel agents for independent refactoring

Scanning for duplications...
```

---

## Phase 1: Deep Scan

Systematically scan the codebase for all forms of duplication:

### 1a. Exact Duplicates

Identical code blocks appearing in 2+ locations:

```
SCAN: Exact Duplicates

Search patterns:
- Functions with identical bodies in different files
- Copy-pasted code blocks (3+ lines identical)
- Identical switch/case or if/else chains
- Repeated import groups
- Duplicated type definitions or interfaces
- Identical test setup/teardown blocks
- Repeated error handling patterns
- Same validation logic in multiple endpoints
```

**Method:** Use Grep to find repeated patterns. Compare function bodies across files. Flag any block of 3+ lines that appears identically in 2+ locations.

### 1b. Near Duplicates

Code blocks that are 80%+ similar with minor variations:

```
SCAN: Near Duplicates

Search patterns:
- Functions with same structure but different variable names
- Similar API handlers with minor differences (different model, same CRUD pattern)
- Copy-pasted components with slight prop differences
- Almost-identical test cases with different inputs
- Similar database queries with different filters
- Repeated patterns that differ only by entity name
- UI components that share 80%+ of their markup/logic
```

**Method:** Look for structural similarity. Functions with the same control flow but different identifiers. Components with the same JSX structure but different props. Handlers that follow the same pattern with different models.

### 1c. Missed Abstractions

Patterns that should be utilities or shared modules:

```
SCAN: Missed Abstractions

Search patterns:
- Same error formatting logic in multiple places
- Repeated date/time formatting
- Common string transformations done inline everywhere
- Same fetch/API call wrapper pattern
- Repeated authentication checks
- Same pagination logic
- Common form validation rules
- Repeated file path construction
- Same environment variable access patterns
- Logging patterns that should be a utility
```

### 1d. Scan Report

After scanning, produce a brief summary (NOT a detailed report -- we're about to fix everything):

```
SCAN COMPLETE

Found:
  Exact duplicates:    [N] instances across [N] files
  Near duplicates:     [N] instances across [N] files
  Missed abstractions: [N] patterns across [N] files
  Total fixes needed:  [N]

Estimated refactoring groups: [N] independent groups
  → Group 1: [description] -- [N] files affected
  → Group 2: [description] -- [N] files affected
  → ...

Spawning [N] parallel agents to fix all groups simultaneously.
```

---

## Phase 2: Parallel Fix Execution

Group related duplications and spawn parallel agents to fix them simultaneously.

### Grouping Strategy

Duplications that share files or have overlapping changes must be in the SAME group (to avoid merge conflicts). Duplications in completely separate areas of the codebase go to separate agents.

```
GROUPING RULES

Same group (sequential within agent):
  - Duplications that modify the same file
  - Duplications where one extraction feeds into another
  - Near-duplicates that will share the same new utility

Separate group (parallel agents):
  - Duplications in unrelated modules
  - Independent utility extractions
  - Separate component consolidations
```

### Agent Briefing

For each group, spawn an agent with a complete, self-contained brief:

```
AGENT BRIEF -- Group [N]: [description]

TASK: Fix [N] duplications in [module/area]

DUPLICATIONS TO FIX:

1. [file_a:line] and [file_b:line]
   Duplication: [description of what's duplicated]
   Fix: Extract to [new_file:function_name]
   Then: Update both call sites to use the shared version

2. [file_c:line] and [file_d:line]  
   Duplication: [description]
   Fix: [strategy]

EXTRACTION TARGETS:
- New file: [path for shared utility if needed]
- Existing file to extend: [path if adding to existing module]

RULES:
- Extract shared logic to a well-named function/module
- Update ALL call sites to use the shared version
- Maintain exact same behavior (no functional changes)
- Add/update exports as needed
- Fix imports in all affected files
- Keep naming consistent with existing codebase conventions
- If the shared logic needs parameters, make them explicit (no globals)

DO NOT:
- Change any behavior -- only structure
- Rename unrelated variables
- Refactor code that isn't duplicated
- Add features or improvements beyond deduplication
- Touch files outside your assigned group

AFTER FIXING:
- Run: [test command] -- verify all tests pass
- Run: [lint command] -- verify no lint errors
- Report: files modified, functions extracted, call sites updated
```

### Launching Agents

```
LAUNCHING PARALLEL AGENTS

Agent 1 (worktree): Group 1 -- [description]
  Files: [list]
  Duplications: [N]
  
Agent 2 (worktree): Group 2 -- [description]
  Files: [list]
  Duplications: [N]

Agent 3 (worktree): Group 3 -- [description]
  Files: [list]
  Duplications: [N]

[...]

All agents working in isolated worktrees to prevent conflicts.
Monitoring progress...
```

**Implementation:** Use the Agent tool with `isolation: "worktree"` and `run_in_background: true` for each group. Each agent gets the full brief.

### Single-Agent Fallback

If all duplications are in overlapping files (can't parallelize safely), run sequentially in a single agent with all fixes:

```
OVERLAP DETECTED -- All duplications share files.
Running single-agent sequential fix to avoid conflicts.
```

---

## Phase 3: Merge & Verify

After all agents complete:

### 3a. Collect Results

```
AGENT RESULTS

Agent 1: ✓ Complete
  - Extracted: [N] shared functions
  - Updated: [N] call sites across [N] files
  - New files: [list]
  - Modified files: [list]

Agent 2: ✓ Complete
  [...]

Agent 3: ✓ Complete
  [...]
```

### 3b. Merge Agent Work

If agents used worktrees, merge their changes:
1. Merge each agent's branch sequentially
2. If conflicts arise (shouldn't if grouping was correct), resolve them
3. After all merges, verify the combined result

### 3c. Final Verification

**This is non-negotiable. Every refactoring MUST pass these checks.**

```
VERIFICATION

Step 1: Run full test suite
  Command: [detected test command -- npm test / pytest / cargo test / etc.]
  Result: [PASS / FAIL]
  
  If FAIL:
    → Identify which test broke
    → Determine if it's a refactoring error or a pre-existing failure
    → Fix refactoring errors immediately
    → Re-run tests
    → Maximum 3 fix attempts before reverting the problematic change

Step 2: Run linter
  Command: [detected lint command -- eslint / ruff / clippy / etc.]
  Result: [PASS / FAIL]
  
  If FAIL:
    → Fix all lint errors introduced by refactoring
    → Re-run linter
    → These should be trivial (import order, unused vars)

Step 3: Type check (if applicable)
  Command: [tsc / mypy / etc.]
  Result: [PASS / FAIL]

Step 4: Build check (if applicable)  
  Command: [npm run build / cargo build / etc.]
  Result: [PASS / FAIL]
```

---

## Phase 4: Report

After everything passes:

```
TECHDEBT REPORT

═══════════════════════════════════════════════════
DEDUPLICATION COMPLETE
═══════════════════════════════════════════════════

Summary:
  Exact duplicates eliminated:    [N]
  Near duplicates consolidated:   [N]
  Shared utilities extracted:     [N]
  Total files modified:           [N]
  New shared modules created:     [N]
  Lines of code removed (net):    [N]

Agents used: [N] parallel + [N] sequential
Verification: ✓ Tests pass | ✓ Lint clean | ✓ Types check | ✓ Build succeeds

Changes by area:
  [module/area 1]: [N] duplications fixed -- extracted [utility names]
  [module/area 2]: [N] duplications fixed -- consolidated [component names]
  [...]

New shared modules:
  [path/to/shared/utility.ts] -- [what it provides]
  [path/to/shared/component.tsx] -- [what it provides]
  [...]

Files modified:
  [full list of every file touched]
```

---

## Detection Commands

Auto-detect the project's test and lint commands:

| File Present | Test Command | Lint Command |
|-------------|-------------|-------------|
| `package.json` | `npm test` or scripts.test | `npm run lint` or scripts.lint |
| `Makefile` | `make test` | `make lint` |
| `pyproject.toml` | `pytest` | `ruff check .` or `flake8` |
| `Cargo.toml` | `cargo test` | `cargo clippy` |
| `go.mod` | `go test ./...` | `golangci-lint run` |
| `Gemfile` | `bundle exec rspec` | `rubocop` |

If no test/lint commands are found, warn:
```
WARNING: No test suite detected. Refactoring without tests is risky.
I'll still fix duplications, but cannot verify behavior is preserved.
Want me to proceed? (yes / no / let me set up tests first)
```

---

## Refactoring Patterns

### Pattern 1: Extract Function
```
BEFORE (duplicated in file_a.ts and file_b.ts):
  const formatted = `${date.getMonth()+1}/${date.getDate()}/${date.getFullYear()}`

AFTER:
  // shared/formatDate.ts (new)
  export function formatDate(date: Date): string {
    return `${date.getMonth()+1}/${date.getDate()}/${date.getFullYear()}`
  }
  
  // file_a.ts and file_b.ts
  import { formatDate } from './shared/formatDate'
  const formatted = formatDate(date)
```

### Pattern 2: Extract Parameterized Function
```
BEFORE (near-duplicate):
  // file_a.ts: fetchUsers() { return fetch('/api/users').then(r => r.json()) }
  // file_b.ts: fetchOrders() { return fetch('/api/orders').then(r => r.json()) }

AFTER:
  // shared/api.ts (new)
  export function fetchJSON<T>(path: string): Promise<T> {
    return fetch(path).then(r => r.json())
  }
  
  // file_a.ts: const users = await fetchJSON<User[]>('/api/users')
  // file_b.ts: const orders = await fetchJSON<Order[]>('/api/orders')
```

### Pattern 3: Extract Component
```
BEFORE (duplicated React component with minor prop differences):
  // PageA.tsx: <div className="card"><h2>{title}</h2><p>{desc}</p></div>
  // PageB.tsx: <div className="card"><h2>{name}</h2><p>{summary}</p></div>

AFTER:
  // shared/Card.tsx (new)
  export function Card({ title, description }: { title: string; description: string }) {
    return <div className="card"><h2>{title}</h2><p>{description}</p></div>
  }
```

### Pattern 4: Extract Hook (React)
```
BEFORE (duplicated state + effect logic):
  // ComponentA.tsx: const [data, setData] = useState(); useEffect(() => fetch(url)..., [])
  // ComponentB.tsx: const [data, setData] = useState(); useEffect(() => fetch(url)..., [])

AFTER:
  // hooks/useFetch.ts (new)
  export function useFetch<T>(url: string) { ... }
```

### Pattern 5: Consolidate Constants
```
BEFORE:
  // file_a.ts: const MAX_RETRIES = 3
  // file_b.ts: const MAX_RETRIES = 3
  // file_c.ts: const maxRetries = 3

AFTER:
  // shared/constants.ts: export const MAX_RETRIES = 3
```

---

## Operating Principles

1. **Fix, don't report** -- this skill directly eliminates duplication; it does not create reports for humans to act on
2. **Behavior preservation** -- the code must do exactly the same thing after refactoring; zero functional changes
3. **Parallel when safe** -- spawn agents for independent file groups; serialize when files overlap
4. **Tests are mandatory** -- every refactoring must pass the test suite; if tests break, fix or revert
5. **Lint is mandatory** -- refactored code must be lint-clean
6. **Name things well** -- extracted utilities get clear, descriptive names matching codebase conventions
7. **Minimal extraction** -- extract what's duplicated, nothing more; don't over-abstract
8. **3+ lines rule** -- don't extract one-liners; the abstraction cost exceeds the duplication cost

## Document Output

### File Strategy

| Output | File Location |
|--------|---------------|
| Techdebt report | `docs/techdebt-report-[date].md` |

### Rules
- **Report is a summary, not a plan** -- by the time it's written, fixes are already applied
- **Include before/after metrics** -- lines removed, duplications eliminated, shared modules created
- **Update, never recreate** -- subsequent runs append to existing report

## Cross-Cutting Protocols

### Git Workflow
- **Commit tag:** `refactor`
- **Commit when:** After all agents complete and verification passes
- **Branch:** If changes are large (10+ files), create `refactor/techdebt-[date]`; otherwise commit to current branch
- **Commit format:** `refactor: eliminate [N] code duplications, extract [N] shared modules`
- **One commit for all changes** -- deduplication is atomic; partial dedup is worse than none

### Resume Protocol
- **On start:** Check for `docs/techdebt-report-*.md` from prior runs
- **If found:** "Previous deduplication run found. Scan for new duplications only? (yes / full rescan)"

### Context Loading
- **Before scanning:** Read project structure, detect language, find test/lint commands
- **Extract:** Existing shared modules (to add to, not duplicate), naming conventions, file organization patterns

### Smart Skip
- Not applicable -- this skill scans and fixes, it doesn't ask questions

---

## Pipeline Integration

### Position in Pipeline

This skill can be invoked at any time but is most useful:
- After Phase 6 (Build + QA) -- clean up implementation
- As a standalone maintenance task
- Before code review (Phase 7) -- reviewer sees clean code

### Output: What You Hand Off

| Hand Off To | When | What to Provide |
|-------------|------|-----------------|
| **code-reviewer** | After deduplication | Clean codebase for review, list of extracted modules |
| **qa-tester** | After deduplication | Changed files list for regression testing |
| **tech-lead** | If refactoring breaks tests | Broken test details, what was changed |
