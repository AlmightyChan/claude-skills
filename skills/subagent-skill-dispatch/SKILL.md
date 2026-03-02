---
version: 1.0.0
description: "Skill dispatch instructions for orchestrator when dispatching subagents via Task tool"
user-invocable: false
---

# Subagent Skill Dispatch

When dispatching ANY subagent via the Task tool, include relevant skill-loading instructions in the dispatch prompt.

## Skill Matching Table

| Domain Signal | Skill to Load | Target Agent |
|---------------|---------------|--------------|
| Bug, test failure, unexpected behavior | `systematic-debugging` | builder |
| New feature, behavior change, bugfix implementation | `tdd` | builder |
| Claiming done, marking complete, final check | `verification-before-completion` | builder, validator |
| CI failure, pipeline error, build log | `ci-fix` | builder |
| Security audit, auth, injection, secrets | `review-security` | auditor |
| Code review, quality check | `review-code-quality` | auditor |
