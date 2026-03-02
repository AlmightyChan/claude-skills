---
name: drift-detection
version: 1.0.0
description: "Drift analysis between plans, docs, and code. Proactively use when checking plan-vs-reality alignment, stale issues, outdated documentation, or scope creep."
---

# Drift Detection

- **Agents:** critic, researcher

## Overview

Cross-reference documented plans against implementation reality to identify drift, gaps, and misalignment. Produces brutally specific, actionable findings -- not vague observations.

**Core principle:** Plans diverge from reality. The question is not whether drift exists, but where it matters most. Adapted from the plan-synthesizer agent's cross-reference methodology.

## When to Use

- Checking whether a plan's tasks match actual implementation state
- Identifying stale issues that are already resolved or no longer relevant
- Finding documentation that describes features that don't exist (or vice versa)
- Detecting scope creep -- features implemented but not in the original plan
- Pre-release reality checks against documented requirements
- Periodic project health assessments

## Methodology

### Step 1: Understand Project Context

Before analyzing drift, establish:
- What type of project is this? (app, library, CLI, infrastructure)
- What are the documented goals? (read plan files, READMEs, issue trackers)
- What's the project's maturity level? (active development, maintenance, pre-release)

Use Glob and Read to gather context from:
- `docs/plan/`, `PLAN.md`, `TODO.md`, `ROADMAP.md`
- `README.md`, `CHANGELOG.md`
- `ISSUES.md`, issue tracker references
- `docs/prd/`, `docs/specs/`

### Step 2: Cross-Reference Analysis

Compare documented state against code reality. Use semantic matching:

| Documented | Look For in Code |
|-----------|-----------------|
| "user authentication" | `auth/`, `login`, session handling |
| "API endpoints" | `routes/`, `handlers/`, `controllers/` |
| "database layer" | `models/`, `migrations/`, `schema` |
| "test coverage" | `tests/`, `__tests__/`, test runner config |

Classify each documented feature into:
1. **Fully aligned** -- Docs and code match
2. **Documented but not implemented** -- In docs/issues but no matching code
3. **Implemented but not documented** -- Code exists but docs don't mention it
4. **Partially implemented** -- Some code exists but incomplete

### Step 3: Identify Drift by Type

#### Plan Drift
- Plan has low completion rate (<30%) with many items
- Phases marked "complete" but code shows otherwise
- Milestones overdue with significant work remaining
- Tasks reordered or skipped without documented reason

#### Issue Drift
- High-priority issues stale >90 days
- Issues marked "in progress" but no recent commits
- Duplicate issues indicating confusion about scope
- Issues already resolved but still open

#### Documentation Drift
- README describes features that don't exist
- API docs don't match actual exports or endpoints
- CHANGELOG missing recent significant changes
- Setup instructions reference outdated tooling or versions

#### Scope Drift
- Many features documented but few implemented (overcommit)
- Many features implemented but not documented (underdocumented)
- Features implemented that were not in any plan (scope creep)

### Step 4: Prioritize Findings

Apply priority formula:
```
Priority = BaseSeverity + CategoryWeight + BlockerBonus + QuickWinBonus
```

| Factor | Weight | Criteria |
|--------|--------|----------|
| BaseSeverity | 1-4 | Critical=4, High=3, Medium=2, Low=1 |
| CategoryWeight | 0-2 | Security/blocking=2, Quality=1, Documentation=0 |
| BlockerBonus | +2 | Blocks other work or prevents release |
| QuickWinBonus | +1 | Can be resolved in <30 minutes |

### Step 5: Produce Report

Output must be **brutally specific**. Every finding must be actionable.

**Good output:**
- "Close issue #45 -- already implemented in `src/auth/login.js`"
- "Issue #23 is stale -- the feature was removed in v2.0 (commit abc123)"
- "Cannot release: 3 tests missing for auth module, security issue #78 still open"
- "README claims dark mode support -- no code exists for this feature"

**Bad output (never produce this):**
- "Some issues may be stale"
- "Documentation could be more up to date"
- "Consider reviewing the plan"

## Finding Format

```
### [DRIFT-TYPE] [SEVERITY] {Title}

**Category:** {plan|issue|documentation|scope}
**Evidence:** {specific file paths, issue numbers, code references}
**Impact:** {what this drift causes -- blocked work, user confusion, false confidence}
**Action:** {exact action to take -- "Close #45", "Update README section X", "Add tests for Y"}
**Effort:** {quick-win (<30m) | moderate (1-4h) | significant (>4h)}
```

## Constraints

- Every finding must include specific evidence (file paths, issue numbers, commit references)
- Use Glob, Grep, and Read for deterministic data gathering -- do not speculate
- Do not generate vague observations -- if you can't be specific, you don't have enough data
- Do not recommend "reviewing" something -- recommend the specific action to take
- Distinguish between intentional deviations (documented decisions) and unintentional drift
- When uncertain whether something is drift or intentional, note the uncertainty and suggest verification

## Related Skills

- `sync-docs` -- Documentation-code sync (drift-detection is broader, covering plans and issues too)
- `review-architecture` -- Architecture drift as a subset of structural analysis
- `verification-before-completion` -- Verifying that claimed completions match reality

