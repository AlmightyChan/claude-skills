---
name: review
version: 1.0.0
description: "Parallel code review across quality, security, performance, and architecture. Dispatches 4 auditor agents simultaneously, aggregates findings, and optionally fixes issues."
argument-hint: "[focus area or file paths]"
model: opus
---

# Review

- **Agents:** orchestrator-invoked

Parallel review across all four lenses. Dispatches auditors simultaneously, aggregates findings, and optionally fixes.

## Variables

FOCUS: $ARGUMENTS

## Phase 1: Identify Scope

Run `git diff --stat HEAD` to see what changed. If FOCUS is provided, narrow scope to the specified area or paths. Otherwise review all changed files.

Count source files: `git ls-files | grep -E '\.(ts|tsx|js|py|swift|go|rs)$' | wc -l`. If under 50, skip architecture review.

## Phase 2: Parallel Dispatch

Dispatch all auditor agents simultaneously using the Agent tool with `run_in_background: true`.

**Agent 1 — Code Quality**
Dispatch an auditor agent. Prompt: "Invoke Skill('review-code-quality'). Review scope: {FOCUS or changed files}. Return all findings in the review-code-quality finding format."

**Agent 2 — Security**
Dispatch an auditor agent. Prompt: "Invoke Skill('review-security'). Review scope: {FOCUS or changed files}. Return all findings in the review-security finding format."

**Agent 3 — Performance**
Dispatch an auditor agent. Prompt: "Invoke Skill('review-performance'). Review scope: {FOCUS or changed files}. Return all findings in the review-performance finding format."

**Agent 4 — Architecture** _(skip if source file count < 50)_
Dispatch an auditor agent. Prompt: "Invoke Skill('review-architecture'). Review scope: {FOCUS or changed files}. Return all findings in the review-architecture finding format."

Wait for all dispatched agents to complete before proceeding.

## Phase 3: Aggregate Findings

Collect all agent outputs. Deduplicate findings that appear in multiple lenses (same file + line). Produce a summary table:

```
| Lens         | CRITICAL | HIGH | MEDIUM | LOW |
|--------------|----------|------|--------|-----|
| Quality      |          |      |        |     |
| Security     |          |      |        |     |
| Performance  |          |      |        |     |
| Architecture |          |      |        |     |
| TOTAL        |          |      |        |     |
```

Then list all findings grouped by lens, ordered by severity (CRITICAL → HIGH → MEDIUM → LOW). Include every finding in full — do not summarize or truncate.

## Phase 4: Optional Fix

After presenting the aggregated findings, ask the user (AskUserQuestion):

"Found {TOTAL} findings across {N} lenses. Would you like to fix them?"

Options:
1. "Fix all HIGH and CRITICAL (Recommended)" — dispatch builder agents for each HIGH/CRITICAL finding, grouped by file to minimize context switching.
2. "Fix specific findings" — user specifies which findings to fix by ID or description.
3. "No fix, findings only" — stop here.

For each fix batch dispatched to a builder: include the finding's evidence, suggestion, and file path in the dispatch prompt. After all builders complete, re-run the relevant review lens to verify the fix landed.

## Related Skills

- `review-code-quality` — Quality lens
- `review-security` — Security lens
- `review-performance` — Performance lens
- `review-architecture` — Architecture lens (50+ file codebases only)
