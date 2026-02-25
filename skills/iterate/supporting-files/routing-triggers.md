# Routing Triggers

This file defines the 5 patterns that cause the iterate skill to hand off to a discipline skill. Routing is an enhancement, not a requirement -- if the target skill is unavailable, the iterate skill falls back to inline problem-solving.

---

## Trigger Table

| # | Pattern | Detection Heuristic | Target Skill | Announcement | Context to Pass |
|---|---------|-------------------|--------------|--------------|-----------------|
| 1 | Fix fails twice on same issue | Counter: track fix attempts per issue ID. Increment when a fix is applied and subsequently fails (test failure, build error, or user reports the fix did not work). Trigger on count reaching 2. | `systematic-debugging` | "This looks like it needs systematic debugging. Two fix attempts for {ISSUE-ID} have not resolved it -- switching to a structured investigation." | Issue ID, issue description, the two failed fix attempts (what was tried and how each failed), affected files, error messages or test output from each failure |
| 2 | New behavior with testable acceptance criteria | The user describes work that adds new behavior (not fixing broken behavior), AND the ISSUES.md entry or user description contains testable acceptance criteria (checkboxes, "should X when Y" patterns, or quantifiable outcomes). | `tdd` | "This is a good candidate for TDD. The acceptance criteria are testable -- starting with a failing test." | Issue ID, acceptance criteria list, affected files, relevant existing test files in the same module |
| 3 | Multiple symptoms suggest shared root cause | Two or more open issues share at least one affected file in their Files field, AND the issues were discovered or worked on in the same session, AND the symptoms are in the same functional area (e.g., both involve state management, both involve the same data model). | `systematic-debugging` | "These issues might share a root cause. {ISSUE-A} and {ISSUE-B} both touch {shared files} -- investigating systematically before fixing individually." | All related issue IDs with descriptions, shared file paths, symptom descriptions for each issue, any error messages |
| 4 | Issue about to be marked Resolved | The iterate skill is about to change an issue's Status from Open to Resolved. This triggers unconditionally for every resolution -- there is no heuristic threshold. | `verification-before-completion` | "Before marking {ISSUE-ID} as resolved, verifying the fix." | Issue ID, acceptance criteria from the ISSUES.md entry, the verification command to run (test command, build command, or manual check), the files that were changed as part of the fix |
| 5 | Regression prevention needed for critical fix | An issue with `priority/critical` is about to be marked Resolved, AND the fix involved code changes (not just configuration). | `tdd` | "This was a critical fix -- writing a regression test to prevent recurrence." | Issue ID, description of the bug that was fixed, the root cause, the files changed, the fix applied, suggestion for what the regression test should assert |

---

## Detection Details

### Trigger 1: Fix Fails Twice

**State persistence:** Fix attempt counts are persisted in the Fix Attempts section of `iterate-status.md` — NOT in conversation memory. Re-read `iterate-status.md` before checking the counter. This ensures the counter survives context compaction.

**Increment when:** A fix is applied (code change committed or presented to user) and subsequently determined to have failed. After incrementing, write the updated count to `iterate-status.md` immediately. Failure is determined by:
- A test command related to the fix returns failures
- A build command fails after the fix
- The user states the fix did not work (e.g., "that didn't fix it", "still broken", "same error")

**Reset when:** The issue is marked Resolved or Trashed. Remove the entry from the Fix Attempts section of `iterate-status.md`.

**Does NOT count as a failure:** User requesting a different approach before the fix is tested.

**Interaction with the 3-strike rule (SKILL.md):** The fix attempt counter and the 3-strike rule operate on the same counter. The counter does NOT reset when systematic-debugging fires. Concrete scenario:
1. Attempt 1 fails → counter = 1. Continue with next fix attempt.
2. Attempt 2 fails → counter = 2 → **systematic-debugging fires.** The orchestrator announces the methodology shift and investigates the root cause.
3. Systematic-debugging produces a fix. If that fix also fails → counter = 3 → **3-strike rule fires.** Stop retrying. Present failure context to user and offer alternatives.
4. If the systematic-debugging fix succeeds → issue proceeds to verification and resolution. Counter is removed on Resolved/Trashed.

### Trigger 2: New Behavior Detection

**Positive signals (any one sufficient):**
- User says "add", "implement", "create", "new feature", "build"
- ISSUES.md entry has prefix FEAT-
- Acceptance criteria contain "should", "must", "returns", "displays"

**Negative signals (any one disqualifies):**
- User says "fix", "broken", "regression", "not working"
- ISSUES.md entry has prefix BUG-
- The work is correcting existing behavior, not adding new behavior

### Trigger 3: Shared Root Cause Detection

**Conditions (all required):**
- Two or more issues have at least one overlapping file in their Files field
- The issues are being worked on or discussed in the same session
- The symptoms are functionally related (same subsystem, same data flow)

**This is a judgment call.** The skill should surface the observation and let the user confirm before routing. Example: "BUG-017 and BUG-038 both involve plan feedback persistence and touch PlansViewModel.swift -- they might share a root cause. Want me to investigate them together?"

### Trigger 4: Resolution Verification

**Unconditional.** Every issue being marked Resolved passes through verification. No exceptions, no threshold.

**Pre-check (before any verification):** Walk through every acceptance criterion checkbox in the ISSUES.md entry. If ANY box is unchecked, STOP — the issue cannot be marked Resolved regardless of verification outcome. Either complete the remaining criteria or leave the issue Open with a note.

The skill must produce one of two concrete verification proofs:

**Automated verification:**
1. Identify or write a test that exercises the specific fixed behavior (not just "does it build")
2. A successful build alone (`BuildProject — 0 errors`) is NEVER sufficient verification for behavioral issues. The test must assert the specific behavior described in the issue.
3. Present the exact command to the user: `Verification (automated): {command}`
4. Run the command and show the actual output
5. Confirm the output demonstrates the acceptance criteria are met
6. Only then mark Resolved

**Manual verification (required for UI/behavioral issues):**
1. Write a step-by-step physical verification checklist tied to the acceptance criteria
2. Present it to the user: `Verification (manual — your confirmation required):`
3. Wait for explicit user confirmation that the steps pass
4. Only mark Resolved after user confirms — a non-response is not confirmation
5. "I'll check later" means the issue stays Open

**Selection heuristic:**
- If the issue involves button handlers, navigation, UI state, data persistence across launches, lifecycle behavior, visual appearance, or animation — use **manual**. These cannot be verified by compilation or unit tests.
- If acceptance criteria reference specific behavior that can be tested in code (function output, data transformation, calculation), use automated
- If the project has an existing test suite covering the affected area, prefer extending it
- If the fix spans both logic and UI, use BOTH: run the automated test AND present manual steps
- If no test infrastructure exists and the fix is a build/config change, run the build as automated verification AND provide manual steps for behavioral confirmation

### Trigger 5: Regression Test for Critical Fixes

**Conditions (all required):**
- Issue priority is `critical`
- Status is transitioning from Open to Resolved
- The fix involved code changes (not just config/asset changes)

**Exclusions:**
- Config-only changes (e.g., changing a plist value)
- Asset-only changes (e.g., adding an app icon)
- Documentation-only changes

---

## Context Format for Validator Dispatch

The validator is the most critical quality gate. Its dispatch prompt must include structured context so it can independently verify acceptance criteria without relying on the builder's claims.

Format the validator prompt as:

```
## Validate: {ISSUE-ID}: {Title}

**Issue description:** {Description from ISSUES.md}

**Acceptance criteria:**
- [ ] {Criterion 1}
- [ ] {Criterion 2}

**Files changed by builder:** {list of files modified in the worktree branch}

**Behavioral outcome:** {If UI/behavioral: "User does X → Y happens." If logic: "Function returns X given Y."}

**What to verify:**
1. Each acceptance criterion is met by the code changes
2. No regressions introduced in touched files
3. Code compiles without new warnings
4. If behavioral: describe the expected observable behavior for manual verification checklist

**What NOT to do:** Do not modify any files. Do not trust the builder's self-reported results. Verify independently.
```

---

## Context Format for Discipline Skills

Each discipline skill expects context in its invocation. The iterate skill passes context as part of the Task tool prompt.

### For systematic-debugging

The skill expects:
- A clear description of what is broken (the symptom)
- Evidence gathered so far (error messages, stack traces, test output)
- What has already been tried and failed
- The affected files

Format the context as:

```
## Issue: {ISSUE-ID}: {Title}

**Symptom:** {What is happening}

**Evidence:**
{Error messages, test output, stack traces}

**Previous attempts:**
1. {What was tried} -- {How it failed}
2. {What was tried} -- {How it failed}

**Affected files:** {file list}
```

### For tdd

The skill expects:
- A clear description of the behavior to implement
- Testable acceptance criteria
- The target files for implementation
- Existing test files in the module (if any)

Format the context as:

```
## Behavior: {ISSUE-ID}: {Title}

**What should happen:** {Description of desired behavior}

**Acceptance criteria:**
- [ ] {Criterion 1}
- [ ] {Criterion 2}

**Implementation files:** {target files}
**Existing tests:** {test files in same module, or "none"}
```

### For verification-before-completion

The skill expects:
- The claim being made (e.g., "BUG-005 is resolved")
- The verification command or checklist
- The acceptance criteria to verify against

Format the context as:

```
## Verifying: {ISSUE-ID}: {Title}

**Claim:** {ISSUE-ID} is resolved.

**Acceptance criteria:**
- [ ] {Criterion 1}
- [ ] {Criterion 2}

**Verification command:** {command to run, or "manual checklist" if no automated check}

**Files changed:** {list of files modified in the fix}
```

---

## Fallback Behavior

When a discipline skill is unavailable (not installed, fails to load, or errors during execution):

1. Log a warning: "Could not load {skill-name} skill -- falling back to inline approach"
2. Do not block progress. Continue with inline problem-solving:
   - For systematic-debugging fallback: Apply the 4-phase methodology inline (read errors, reproduce, check changes, trace data flow)
   - For tdd fallback: Write the test first inline, verify it fails, then implement
   - For verification-before-completion fallback: Run the verification command directly and check output against acceptance criteria
3. Note the fallback in the session summary so the user knows routing did not occur
