---
name: test-coverage-review
description: "Test quality and coverage review. Proactively use when validating test adequacy, checking for meaningful assertions, verifying edge case coverage, or assessing async error testing."
version: 0.1.0
---

# Test Coverage Review

- **Agents:** validator

## Overview

Test quality assessment that goes beyond "do tests exist?" to evaluate whether tests actually exercise the code, contain meaningful assertions, and cover edge cases. Merges test coverage checking with test quality evaluation.

**Core principle:** A test that exists but doesn't meaningfully exercise the code is worse than no test -- it provides false confidence.

## When to Use

- Validating test adequacy after implementation
- Checking whether assertions are meaningful (not trivial)
- Verifying edge case and error path coverage
- Assessing async/concurrent code test quality
- Reviewing test suite health for a module or feature
- Post-build validation of test completeness

## Review Methodology

### Step 1: Source-to-Test Mapping

Detect the project's test file naming convention:
- Check for: `tests/`, `__tests__/`, `test/`, `spec/`
- Check naming: `.test.`, `.spec.`, `test_`, `_test`, `Tests` (Swift)
- Map each source file to its corresponding test file
- Flag source files with no corresponding test file

### Step 2: Five-Dimension Quality Check

For each test file, evaluate:

#### A. Imports Tested

- Test file actually imports/requires the source module
- Test exercises the public API of the module, not internal helpers
- No tests that only test mock behavior without touching real code

**Detection:** Check that the test file's import statements reference the source file under test. A test file that only imports test utilities and mocks is suspicious.

#### B. Meaningful Assertions

Assertions must check specific expected values, not trivial truths.

**Trivial assertion patterns to flag:**
- `expect(true).toBe(true)` / `XCTAssertTrue(true)`
- `expect(1).toBe(1)` / `XCTAssertEqual(1, 1)`
- `expect(result).toBeDefined()` without checking value
- `assert result is not None` without checking content
- Assertions that only verify type/shape without values

**Meaningful assertion patterns:**
- Checking specific return values against expected outputs
- Verifying state changes after operations
- Asserting error messages match expected text
- Checking collection contents (not just length)
- Verifying side effects occurred (API called, event emitted)

#### C. Test Names Match Source Functionality

- Test describes/its correspond to source functions/methods
- Test names describe behavior, not implementation
- Test names follow project naming convention

#### D. Edge Case Coverage

Check for tests covering:
- Null/nil/undefined inputs
- Empty strings, empty arrays, empty objects
- Zero and negative numbers
- Boundary values (min, max, off-by-one)
- Invalid input formats
- Concurrent/interleaved operations
- Error/failure paths

**Detection:** Search test files for edge case keywords: `null`, `undefined`, `nil`, `empty`, `error`, `invalid`, `edge`, `boundary`, `zero`, `negative`, `overflow`, `timeout`.

#### E. Async Error Path Testing

For code with async operations:
- Network failure scenarios tested
- Timeout handling tested
- Cancellation handling tested
- Race condition scenarios considered
- Error propagation through async chains verified

### Step 3: Test Depth Analysis

Analyze source code complexity to determine adequate test depth:

- Extract conditionals and branches from source code
- If source has 3+ branches: classify as high complexity, flag if fewer than 3 test cases
- If source has async/await patterns: flag if no error path tests exist
- If source has state mutations: flag if no before/after state assertions exist

### Step 4: Adaptive Focus

Adjust review depth based on existing test state:

- **Tests exist:** Verify coverage depth -- are edge cases covered? Are assertions meaningful?
- **No tests detected:** Report missing tests with specific recommendations for what to test first (prioritize: public API, error paths, edge cases)
- **Tests exist but trivial:** Flag as "coverage theater" -- tests provide false confidence

## Checklist

For each source-test pair:

- [ ] Test file imports and exercises the source module
- [ ] No trivial assertions (true==true, 1==1, defined-only checks)
- [ ] Test names describe behavior, not implementation details
- [ ] Null/empty/boundary inputs have dedicated test cases
- [ ] Error paths have dedicated test cases
- [ ] Async operations have failure scenario tests
- [ ] State-changing operations verify before and after state
- [ ] No tests that only verify mock behavior
- [ ] Test count is proportional to source complexity
- [ ] Integration points have at least one cross-boundary test

## Finding Format

```
### [SEVERITY] file:line

**Category:** {missing-import|trivial-assertion|missing-edge-case|missing-error-test|no-async-error|coverage-gap}
**Confidence:** {high|medium|low}
**Description:** {What the test quality issue is}
**Evidence:** {Exact test code or missing test scenario}
**Source Reference:** {The source code being undertested, with file:line}
**Recommendation:** {Specific test to add or assertion to strengthen}
```

Severity levels: BLOCKING, IMPORTANT, MINOR
- BLOCKING: No tests for critical path, or tests that provide false confidence (trivial assertions on complex logic)
- IMPORTANT: Missing edge cases, incomplete error path coverage, or weak assertions
- MINOR: Naming convention deviation, missing documentation-level tests

## Constraints

- This skill validates test QUALITY, not just test EXISTENCE
- Do not write tests -- identify gaps and recommend what to test
- A test file that exists but doesn't meaningfully exercise the code is flagged as a gap
- Do not count lines of test code as a quality metric -- a 5-line test with meaningful assertions beats a 50-line test with trivial checks
- Tests that only verify constant values without testing runtime behavior do not satisfy functional completeness
- Flag but do not fail for missing tests on generated code, configuration, or trivial getters/setters

## Related Skills

- `tdd` -- Ensures tests are written before code (this skill reviews test quality after the fact)
- `review-code-quality` -- General code quality (this skill focuses on test quality dimension)
- `verification-before-completion` -- Completion verification (this skill provides the test quality input)

## Completion Token

When this skill completes successfully, output: `[SKILL_COMPLETE:test-coverage-review]`
