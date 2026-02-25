---
name: deslop
description: "AI slop detection and cleanup. Proactively use when cleaning AI-generated code, detecting debug remnants, placeholder functions, hardcoded secrets, or AI verbosity patterns."
version: 0.1.0
---

# Deslop

## Overview

Scan codebases for AI-generated slop patterns and produce a structured report with certainty-based triage. Patterns are categorized by type and severity, with auto-fix strategies for high-certainty items.

**Core principle:** AI-generated code has predictable failure modes. Systematic detection catches what manual review misses.

## When to Use

- Cleaning up AI-generated code before commit or PR
- Detecting debug statement remnants (`console.log`, `print()`, `dump()`)
- Finding placeholder functions and `TODO` stubs in production paths
- Scanning for hardcoded secrets or API keys
- Identifying AI verbosity patterns in comments or documentation
- Post-build quality sweep before release

## Methodology

### Step 1: Determine Scope

Identify files to scan based on mode:
- **all**: Scan all source files (exclude `.gitignore`-ed, generated, `dist/`, `build/`, `*.min.js`)
- **diff**: Scan only files changed since base branch (`git diff --name-only origin/main..HEAD`)
- **path**: Scan files under specified directory

Use Glob to collect file list. Use Grep to scan for patterns.

### Step 2: Pattern Detection

Read `supporting-files/slop-categories.md` for the full pattern taxonomy. The categories are:

1. **Console Debugging** -- `console.log`, `print()`, `debugPrint()`, `dump()`, `dbg!()`
2. **Unsafe Error Handling** -- force-unwraps (`!`), bare `.unwrap()`, `try!`
3. **Placeholder Code** -- `TODO`, `unimplemented!()`, empty function bodies, `fatalError()` placeholders
4. **Error Handling Issues** -- empty catch blocks, `except: pass`
5. **Hardcoded Secrets** -- `password=`, `api_key=`, `sk-`, `ghp_`, `AKIA`, `eyJ...`
6. **Documentation Issues** -- excessive docstrings, stale file references
7. **Code Smells** -- boolean blindness, mutable globals, dead code
8. **Verbosity Patterns** -- AI preambles ("Certainly!"), hedging language, redundant comments

For each file, run targeted Grep searches for patterns in the relevant categories.

### Step 3: Certainty Classification

Classify each finding by certainty:

| Level | Threshold | Action | Examples |
|-------|-----------|--------|----------|
| HIGH | >95% confidence | Auto-fixable, add to fixes | Direct regex match on `console.log` in production code |
| MEDIUM | 75-95% confidence | Review context before fixing | `.unwrap()` that might be intentional |
| LOW | <75% confidence | Flag only, no auto-fix | Heuristic detections, potential false positives |

### Step 4: Triage and Report

Group findings by category and severity. For each finding:

```
[SEVERITY] [CATEGORY] file:line -- pattern description
  Certainty: HIGH | MEDIUM | LOW
  Fix Strategy: remove | replace | add_logging | flag
  Evidence: {matched code snippet}
```

### Step 5: Summary

Produce summary counts:
```
Files scanned: N
Findings: HIGH: N, MEDIUM: N, LOW: N
Auto-fixable: N
Flagged for review: N
```

## Constraint Priority

When applying fixes, follow this priority order:
1. **Preserve behavior** -- Never change observable behavior
2. **Minimal diffs** -- Remove the slop, nothing else
3. **Prefer deletion** -- Remove the problematic code rather than rewriting it
4. **No new dependencies** -- Don't add imports or packages to fix slop
5. **Respect conventions** -- Follow the project's existing patterns

## Iron Constraints

- **NEVER modify test files** -- Tests may intentionally use patterns that look like slop (e.g., `print()` for test output)
- **NEVER remove error logging in production paths** -- Distinguish debug logging (slop) from production error logging (intentional)
- **NEVER auto-fix MEDIUM or LOW certainty items** -- These require human judgment
- **Respect .gitignore** -- Skip files that are gitignored
- **Skip generated files** -- `dist/`, `build/`, `*.min.js`, `*.generated.*`

## Finding Format

```
### [SEVERITY] [CATEGORY] file:line

**Certainty:** {HIGH|MEDIUM|LOW}
**Pattern:** {pattern name from slop-categories.md}
**Fix Strategy:** {remove|replace|add_logging|flag}
**Evidence:** {exact code snippet}
**Context:** {why this is slop, not intentional code}
```

## Constraints

- Report only -- do not modify files unless explicitly asked
- HIGH certainty items go in the fixes list
- MEDIUM/LOW items go in findings summary only
- Always show the evidence (code snippet) with each finding
- Distinguish between test files and production code
- If a pattern match appears intentional based on surrounding context, downgrade certainty

## Related Skills

- `review-code-quality` -- General code quality review (deslop focuses on AI-specific patterns)
- `review-security` -- Security review (deslop catches hardcoded secrets as a subset)
- `ci-fix` -- CI failure fixing (deslop findings may trigger lint/format CI failures)

## Supporting Files

- `supporting-files/slop-categories.md` -- Complete pattern taxonomy with severity levels, exclusions, and auto-fix strategies

## Completion Token

When this skill completes successfully, output: `[SKILL_COMPLETE:deslop]`
