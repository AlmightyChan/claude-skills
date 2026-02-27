---
name: ci-fix
description: "CI failure diagnosis and fix. Proactively use when a CI pipeline fails, build logs show errors, or PR checks report lint, type, test, or build failures."
version: 0.1.0
---

# CI Fix

- **Agents:** builder

## Overview

Diagnose CI failures by classifying error types, then apply targeted fixes based on the failure category. Adapted from the ci-fixer agent's diagnostic workflow with BASECAMP's quality constraints.

**Core principle:** CI failures have patterns. Classify first, fix second. Never guess at fixes.

## When to Use

- CI pipeline reports failure (lint, type check, test, build)
- PR checks show errors that need code changes
- Build logs contain error messages needing diagnosis
- PR review comments request specific code changes

## Methodology

### Step 1: Classify Failure Type

Parse error messages from CI logs or PR check output. Classify into categories:

| Type | Error Patterns | Example |
|------|---------------|---------|
| **lint** | `error <rule>: <message>`, ESLint/SwiftLint output | `error no-unused-vars: 'x' is defined but never used` |
| **type** | `error TS\d+:`, Swift compiler type errors | `error TS2322: Type 'string' is not assignable to type 'number'` |
| **test** | `FAIL <path>`, `Expected ... Received`, XCTest failures | `FAIL tests/auth.test.ts -- Expected 200, received 401` |
| **build** | `error: <message>`, linker errors, missing modules | `error: no such module 'MyFramework'` |
| **format** | Prettier/SwiftFormat diff output | `Code style issues found in src/api.ts` |

If the failure doesn't match any pattern, report it as unclassifiable rather than guessing.

### Step 2: Analyze PR Comment Intent (if applicable)

When fixing PR review comments, classify the reviewer's intent:

| Intent Signal | Classification | Action |
|--------------|---------------|--------|
| `error handling`, `try/catch`, `handle error` | add-error-handling | Wrap identified code in error handling |
| `validate`, `check null`, `verify`, `ensure` | add-validation | Add input validation |
| `refactor`, `simplify`, `clean up`, `extract` | refactor | Restructure identified code |
| `bug`, `fix`, `wrong`, `incorrect`, `should be` | fix-bug | Investigate and fix the specific bug |
| `test`, `coverage`, `spec` | add-test | Add test for specified behavior |

### Step 3: Apply Fix by Category

**Lint failures:**
- Run the project's lint auto-fix command if available (`npm run lint -- --fix`, `swiftlint fix`)
- For unfixable lint errors, read the specific rule and fix manually
- Never disable the lint rule to make it pass

**Type errors:**
- Read the file at the error location
- Understand the type mismatch from the error message
- Fix the type by correcting the code, not by casting or using `any`/`as!`

**Test failures:**
- Read the failing test to understand what it expects
- Read the implementation code to understand current behavior
- Fix the implementation to match the test expectation
- Never modify test assertions to make tests pass

**Build failures:**
- Check for missing imports, missing files, or configuration issues
- Verify file is registered in the build system (e.g., Xcode project.pbxproj)
- Fix missing dependencies or incorrect module references

**Format failures:**
- Run the project's format command (`npm run format`, `swift-format --in-place`)

### Step 4: Verify Fix

After applying the fix, run the same CI command locally to verify:
- The original error no longer appears
- No new errors were introduced
- All other checks still pass

## Iron Constraints

- **NEVER modify test assertions to make tests pass** -- fix the actual code
- **NEVER disable linting rules or skip checks** to resolve failures
- **NEVER suppress warnings or errors** to make CI green
- **MUST report back if fix cannot be determined with confidence** -- never guess
- Only fix issues explicitly identified in CI logs or PR comments
- Do not refactor unrelated code while fixing CI issues
- Commit messages must accurately describe the fix applied

## Finding Format

```
### CI Failure: [TYPE]

**Source:** {CI log / PR comment / build output}
**Classification:** {lint|type|test|build|format}
**Error:** {exact error message}
**File:** {file:line}
**Root Cause:** {why the error occurs}
**Fix Applied:** {what was changed and why}
**Verification:** {command run and result}
```

## Constraints

- Fix only what the CI failure identifies -- no scope expansion
- If a fix requires understanding broader context, read related files first
- When multiple errors exist, fix them in dependency order (build > type > lint > format)
- File not found errors should be reported, not resolved by creating new files
- If git conflicts exist, abort and report rather than resolving

## Related Skills

- `deslop` -- Slop patterns that may trigger lint CI failures
- `tdd` -- Test-driven approach prevents test failures
- `systematic-debugging` -- For complex test failures requiring investigation

## Completion Token

When this skill completes successfully, output: `[SKILL_COMPLETE:ci-fix]`
