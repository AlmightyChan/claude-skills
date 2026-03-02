---
name: review-code-quality
version: 1.0.0
description: "Code quality review lens. Proactively use when reviewing code for style issues, error handling gaps, potential bugs, maintainability problems, or duplication."
---

# Code Quality Review

- **Agents:** auditor

## Overview

Structured code quality review across six dimensions. Every finding is grounded in specific code evidence with actionable remediation.

**Core principle:** Quality issues compound. A missing null check today becomes a production crash tomorrow. Catch them at review time.

## When to Use

- Reviewing code for style and consistency issues
- Checking error handling completeness
- Hunting for potential logic bugs
- Assessing maintainability of new or modified code
- Detecting duplication that should be extracted
- Post-implementation quality sweeps

## Review Methodology

### Six Review Dimensions

#### 1. Style & Consistency

- Naming follows project conventions (check existing patterns, not personal preference)
- Indentation and formatting consistent within the file
- Import ordering follows project convention
- No mixed paradigms within a module (e.g., callbacks and async/await in the same flow)

#### 2. Best Practices

- Constants extracted, not hardcoded magic numbers/strings
- Functions have single responsibility (one behavior per function)
- No premature abstraction (three instances before extracting)
- Dependencies injected where testability matters
- Guard clauses for early returns, not deep nesting

#### 3. Potential Bugs & Logic Errors

- Off-by-one errors in loops, slices, range checks
- Null/nil/undefined access without guards
- Inverted conditionals (checking opposite of intent)
- Race conditions in concurrent code
- Type coercion bugs (implicit conversions, loose equality)
- Unreachable code after early returns or throws

#### 4. Error Handling

- Every error path has explicit handling (no empty catch blocks)
- Errors are categorized and surfaced with context
- Recovery paths exist for recoverable errors
- User-facing error messages are helpful, not generic
- Resources cleaned up in error paths (connections, file handles, streams)

#### 5. Maintainability

- Functions are appropriately sized (cognitive complexity, not line count)
- Control flow is linear and readable (minimal nesting depth)
- Side effects are explicit, not hidden
- Dependencies are clear from imports/parameters
- Comments explain WHY, not WHAT (code should explain the what)

#### 6. Duplication

- No copy-paste code blocks (3+ similar lines warrant extraction)
- Shared logic extracted to common utilities
- Configuration-driven variation preferred over code branching
- Test helpers extracted for repeated setup patterns

## Checklist

For each file under review:

- [ ] Names are clear and follow project conventions
- [ ] No magic numbers or hardcoded strings
- [ ] All error paths handled explicitly
- [ ] No empty catch/except blocks
- [ ] No unreachable code
- [ ] Guard clauses used for preconditions
- [ ] No unnecessary nesting (max 3 levels)
- [ ] No duplicated code blocks (3+ lines)
- [ ] Resources acquired are released (connections, handles)
- [ ] Side effects documented or obvious from function name

## Finding Format

```
### [SEVERITY] file:line

**Category:** {style|practices|bug|error-handling|maintainability|duplication}
**Confidence:** {high|medium|low}
**False Positive Risk:** {yes|no} -- {reason if yes}
**Description:** {What the issue is}
**Evidence:** {Exact code snippet}
**Suggestion:** {Specific fix}
```

Severity levels: HIGH, MEDIUM, LOW
- HIGH: Likely bug, data loss risk, or critical maintainability concern
- MEDIUM: Quality degradation, technical debt, or readability impact
- LOW: Style preference, minor improvement opportunity

## Constraints

- Ground every finding in specific code -- file path, line number, exact snippet
- Distinguish project conventions from personal preferences -- only flag deviations from established project patterns
- When uncertain whether something is a bug or intentional, note confidence as LOW
- Do not propose rewrites or redesigns -- flag issues, suggest fixes
- A finding marked `falsePositiveRisk: yes` should include why it might be intentional
- Focus on the files under review, not adjacent code (flag cross-file concerns for architecture review)

## Related Skills

- `review-security` -- Security-specific review dimension
- `review-architecture` -- Cross-file structural concerns
- `review-performance` -- Performance-specific patterns
- `deslop` -- AI-specific code quality patterns

