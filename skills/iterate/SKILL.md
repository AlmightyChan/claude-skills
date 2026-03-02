---
name: iterate
version: 1.0.0
description: >-
  Post-build project refinement and iteration. Proactively use when iterating on
  a completed project, fixing bugs after initial build, polishing features,
  preparing for release, or when the user says "let's iterate", "let's work
  through issues", or "let's polish this".
argument-hint: "[path to project directory]"
hooks:
  SessionEnd:
    - hooks:
        - type: command
          command: >-
            bash -c 'cd "$PWD" && git diff --name-only | grep -q ISSUES.md ||
            (echo "ISSUES.md was not updated during this session" >&2; exit 2)'
---

# Iterate

- **Agents:** orchestrator-invoked

A conversational refinement session for post-build projects. Read the project state, work through issues, track everything in ISSUES.md.

## Variables

- `PROJECT_DIR`: `$ARGUMENTS` (defaults to current working directory if not provided)

## Session Lifecycle

### LOAD

Load project state using tiered reads. The goal is minimum context cost for maximum situational awareness. Full file contents are loaded on-demand, not upfront.

**Tier 0 — Always loaded (small, essential for orientation):**

1. Read `PROJECT_DIR/iterate-status.md` if it exists. This tracks active background agents from the current or a prior session.
2. Read the **ISSUES.md index** — not the full file. Use Grep to extract issue headers, statuses, and priorities:
   - `Grep pattern="^## |^\*\*Status:\*\*|^\*\*Labels:\*\*" path=PROJECT_DIR/ISSUES.md` (content mode)
   - This produces a lightweight index: every issue ID, title, status, and priority in ~1 line each.
3. Read `PROJECT_DIR/PROJECT.md` **only if** this is the first session on this project or if orientation is needed (unfamiliar codebase, no iterate-status.md from a prior session). For continuing sessions, defer until a specific issue requires project context.

**Tier 1 — Loaded on-demand (when working on a specific issue):**

4. When the user selects an issue to work on, read that issue's **full entry** from ISSUES.md using a targeted read (Grep for the issue ID header, then Read with offset/limit to capture the full entry up to the next `---` separator).
5. Read the source files listed in the issue's `Files:` field.

**Tier 2 — Loaded only when their feature triggers:**

6. Supporting files (`supporting-files/*.md`) are reference material. Load each only when the corresponding feature fires:
   - `routing-triggers.md` — when a discipline skill routing trigger is detected
   - `observation-categories.md` — when performing a proactive observation scan
   - `issues-md-format.md` — when creating a new ISSUES.md entry (and only if the format is not already in context)
   - `project-md-template.md` — when creating a new PROJECT.md
   - `iterate-status-format.md` — when creating a new iterate-status.md

**CRITICAL:** Before any dispatch decision, status summary, or issue state reference later in the session, re-read **iterate-status.md** and the **ISSUES.md index** (not the full file) from disk. Context compaction will lose in-memory state. The files are the source of truth, always. Full issue entries are re-read on-demand when actively working on them.

### ORIENT

Handle missing files and present the session summary. All summary data comes from the Tier 0 index — do not read full ISSUES.md entries at this stage.

**If ISSUES.md is missing:**
Create an empty ISSUES.md with the standard header. Read `supporting-files/issues-md-format.md` for the exact format.

**If PROJECT.md is missing and this is a first session:**
Scan the codebase (directory structure, key files, README if present) and propose a PROJECT.md draft following the template in `supporting-files/project-md-template.md`. Present the draft to the user for review before using it. If the user declines, proceed without it but warn that orientation will be less precise.

**If iterate-status.md exists with in-progress agents from a prior session:**
Flag these as abandoned from the prior session. Offer to clean them up.

**Display the status summary (derived from the ISSUES.md index, not full entries):**
- Project name (from PROJECT.md if loaded, or iterate-status.md project field, or directory name)
- Open issue count by priority: X critical, Y high, Z medium, W low
- Last session date (from iterate-status.md if it exists)
- Then ask: "What do you want to work on?"

Keep the summary brief -- a few lines, not a wall of text.

### WORK LOOP

This is the core of the session. The user describes issues or goals, and you determine the right approach for each.

#### Triage

When the user describes an issue or goal:

**First: Ensure issue exists in ISSUES.md (compaction-safe).** Every change, fix, or feature — no matter how small — MUST have an ISSUES.md entry before any work begins. This is a compaction resilience measure: if context compaction occurs mid-work, the issue description, acceptance criteria, and target files are preserved on disk.

- If the user references an **existing issue** (by ID or description): read that issue's **full entry** from ISSUES.md using a targeted read — Grep for the issue ID header to find the line number, then Read with offset/limit to capture the full entry. Also read the source files from the issue's `Files:` field. This is when Tier 1 data enters context — not before.
- If the user describes a **new issue** (not already tracked): create the ISSUES.md entry immediately, before determining the approach. Auto-increment the ID, fill in the description, acceptance criteria, and target files based on the user's description. Follow the format in `supporting-files/issues-md-format.md`. Announce: "Logged as {ISSUE-ID} in ISSUES.md." Then proceed.

This rule has no exceptions. Even single-line fixes get an entry — the cost of one entry is negligible; the cost of losing context mid-fix is not.

Then determine the approach:

**(a) Inline fix** (FR-021): For trivial single-file changes under ~20 lines -- fix directly in the main conversation. No worktree dispatch. If the current branch is `main`, create a feature branch first (e.g., `iterate/fix-{issue-id}`) and switch to it before making changes. Examples: typo fix, single constant change, one-line logic correction. After fixing, the Verification Gate still applies before updating ISSUES.md status.

**(b) Background dispatch** (FR-006): For multi-file or complex changes -- dispatch a builder in a background worktree. This frees the conversation for the next issue while the builder works.

**(c) Discipline skill methodology** (FR-014, FR-015, FR-016): When patterns match a discipline skill's triggers, announce the methodology shift and follow it inline. See the Discipline Skill Methodology section below.

Do not ask the user which approach to use. Decide based on scope and announce what you are doing.

#### Background Dispatch

Before dispatching, re-read `iterate-status.md` from disk.

**File overlap check (FR-007):** Extract the target file set from the ISSUES.md entry's `Files` field. If the `Files` field is absent or vague, ask: "Which files will this change touch?" Check the target files against all active agents' file sets in `iterate-status.md`. If overlap exists: queue the task and inform the user ("BUG-003 touches the same files as the in-progress BUG-001 fix -- I'll start it once BUG-001 completes"). If no overlap: proceed with dispatch.

**Concurrent agent cap:** Do not dispatch more than 4 concurrent background agents. This aligns with the worktree cap (4 active worktrees, per `.claude/rules/worktree-conventions.md`) which is the binding constraint when using worktree isolation. If 4 agents are active (status `in-progress` in iterate-status.md), queue additional tasks until an active agent completes.

**Dispatch (FR-006):** Use the Task tool with:
- `run_in_background: true`
- `isolation: "worktree"` (see `.claude/rules/worktree-conventions.md` for branch naming, dispatch path rules, and merge-back procedure)
- Builder agent prompt including: issue description, acceptance criteria, target files, and the project directory path
- **ISSUES.md ownership:** Builders must NEVER modify ISSUES.md directly. All ISSUES.md mutations (status changes, new entries, verification fields) are orchestrator-only. ISSUES.md is a shared mutable resource — if both the builder and orchestrator write to it, merge conflicts will occur on worktree merge.
- **Behavioral outcome (required for UI/behavioral issues):** If the issue involves UI, navigation, user interaction, or lifecycle behavior, include a `Behavioral:` line in the dispatch prompt describing the expected user-observable outcome. Format: "User does X → Y happens." Example: "User taps Accept → suggestion is removed from carousel and plan status changes to accepted." This gives the builder concrete success criteria beyond "build passes."

**Update iterate-status.md (FR-022):** After dispatch, write an entry with: agent task ID, issue ID, target files, worktree branch, status (in-progress), and timestamp. See `supporting-files/iterate-status-format.md` for the exact format.

**On completion (FR-008):** When a background agent finishes:

1. **Gap-flagging:** Scan builder output for escalation signals: "assumed", "unclear", "missing information", "not specified", "ambiguous", "best guess", "workaround", "placeholder", "temporary", "not sure", "guessing", "fallback", "approximation", "TODO". If found, present the flagged sentences to the user before proceeding. The builder may have made assumptions that invalidate the fix.

2. **Pre-validator build gate:** Independently run a full-project build (not the builder's claimed result). Do not trust the builder's build claim. If the build fails on files the builder touched, re-dispatch the builder to fix (counts toward the 3-strike limit). If the build passes, proceed.

3. **Validator dispatch:** Dispatch a validator agent (read-only) against the builder's worktree branch to independently verify the work meets acceptance criteria. Do not trust the builder's self-reported verification. Use the validator prompt template in `supporting-files/routing-triggers.md` (Context Format for Validator Dispatch) to structure the prompt with issue description, acceptance criteria, changed files, and behavioral outcome.

4. **Present results:** Show the validator's report and the branch diff to the user. Then use AskUserQuestion: "Merge this fix into the main branch?" with options:
   1. "Merge" — proceed with merge
   2. "Reject — move on" — skip this fix, update iterate-status.md, move on
   3. "Revise — redispatch with feedback" — ask for feedback, then redispatch builder

5. **On approval (Merge selected):** Check the current branch. If on `main`, create a feature branch first (e.g., `iterate/session-{timestamp}`) and switch to it. Then merge the worktree branch (`git merge <branch>`). If conflicts exist, present the diff for manual resolution. Update iterate-status.md.

6. **Post-merge integration check:** After merging, run a full-project build against the merged result (not the worktree branch). Non-overlapping file sets can still cause integration failures when combined. If the build fails, investigate — the issue may be the merge, not the builder's work.

7. **Unblock queued tasks:** After updating iterate-status.md, check the Queued section. If any queued task was blocked by the now-completed agent's issue ID, dispatch it automatically. Remove the entry from Queued and add it to Active Agents.

8. **After merge, the Verification Gate still applies** before marking Resolved. Merging code is not the same as verifying behavior. See the Verification Gate section.

- On rejection (Reject or Revise selected): note feedback, update iterate-status.md. If "Revise" was selected, use AskUserQuestion: "What feedback should the builder address in the next attempt?" Then redispatch with that feedback.

**3-strike rule:** If a builder fails validation 3 times for the same issue, stop retrying. Present the failure context to the user and use AskUserQuestion: "Builder has failed 3 times on this issue. How would you like to proceed?" with options:
   1. "Try a different approach" — describe the alternative, then dispatch
   2. "I'll fix it manually" — leave the issue Open, move on
   3. "Move on — skip this issue" — mark as deferred, move on

**On failure:** Report the failure with error context. Use AskUserQuestion: "How would you like to proceed?" with options:
   1. "Retry" — redispatch the same builder
   2. "Try a different approach" — describe the alternative, then dispatch
   3. "Move on" — skip this issue for now

#### Issue Management

**Updating status (FR-009):** When a fix passes the Verification Gate (see above), update the issue's Status from Open to Resolved in ISSUES.md. Populate the `Verification` field with the type, command/steps, and result. An issue cannot be marked Resolved without a populated Verification field.

**Archival on resolve:** Immediately after marking an issue Resolved, move its entire entry (from `## ` header through the `---` separator) from ISSUES.md to `ISSUES-ARCHIVE.md`. If `ISSUES-ARCHIVE.md` does not exist, create it with the header:

```markdown
# {Project Name} — Resolved Issues Archive

Resolved issues moved from ISSUES.md. Reference only — not loaded by the iterate skill.

---
```

Append the resolved entry to the end of ISSUES-ARCHIVE.md. This keeps ISSUES.md concise (open issues only) and eliminates the need for threshold-based archival at session exit. The archive file is never loaded by the iterate skill but can be referenced manually if needed.

**Adding new issues (FR-010):** When a new issue is discovered during the session (by user report or proactive observation), append a new entry to ISSUES.md. Auto-increment the ID from the highest existing ID of the same prefix type (BUG-XXX, FEAT-XXX, POLISH-XXX). Follow the format in `supporting-files/issues-md-format.md`.

#### Verification Gate

**Every issue must pass a concrete verification proof before its status changes to Resolved.** This gate is the primary quality control for the iterate skill. Violations of these rules were the #1 source of false resolutions in prior sessions.

##### Hard Rules (no exceptions)

1. **Acceptance Criteria Lock:** An issue CANNOT be marked Resolved if any acceptance criterion checkbox is unchecked. The orchestrator is responsible for checking each box: as each criterion is confirmed met (via automated test result or user confirmation), update the checkbox in ISSUES.md from `- [ ]` to `- [x]`. If a criterion cannot be verified yet (e.g., depends on another system not yet working), the issue stays Open. Add a note explaining which criteria are blocked and why. Partial completion is not resolution.

2. **"Build passes" is not verification.** A successful compile/build is a prerequisite, not proof. `BuildProject — 0 errors` proves the code compiles. It does NOT prove buttons work, navigation flows correctly, data persists, or UI renders as intended. Never record a build result as the sole verification for a behavioral issue.

3. **The builder does not self-certify.** A builder reporting "PASS" in its completion report is a claim, not evidence. The orchestrator (this skill) must independently verify via one of the two verification types below.

##### Anti-Pattern (do NOT do this)

This is the exact pattern that caused false resolutions in prior sessions:

> Verification (automated): Xcode MCP BuildProject — 0 errors, 3.2s
> Result: PASS

This verifies compilation, not behavior. A button handler could be empty, navigation could be broken, and data could be lost — and this would still pass. Never use this as sole verification for a behavioral issue.

##### Verification Types

**(a) Automated verification:** A test that exercises the specific fixed behavior. The test must:
- Target the exact behavior described in the issue (not just "the module builds")
- Actually run and produce output that is shown to the user
- Demonstrate the acceptance criteria are met through assertions, not just the absence of errors

Example:
> Verification (automated): `swift test --filter PaywallTests.testRestorePurchasesCallsSubscriptionManager`
> Result: PASS (1 test, 0 failures)
> BUG-005 acceptance criteria met. Marking Resolved.

**(b) Manual verification:** When the issue involves UI behavior, navigation, visual appearance, user interaction, lifecycle behavior, or anything that cannot be exercised by a unit test — present a step-by-step checklist the user must physically perform in the running app. Use AskUserQuestion to get explicit confirmation. Do not mark the issue Resolved until the user explicitly confirms.

Example:
> Present the checklist, then use AskUserQuestion:
> "Please verify in the running app:
> 1. Launch the app on a device or simulator
> 2. Navigate to the onboarding paywall screen
> 3. Tap 'Restore Purchases'
> 4. Verify a loading spinner appears
> 5. Verify success/error feedback is displayed after restore completes
>
> Does this pass?"
> Options: "Yes — mark Resolved" / "No — describe what failed"

##### When to Use Which Type

| Issue involves... | Verification type | Rationale |
|---|---|---|
| Logic, data transformation, calculation | Automated (unit test) | Deterministic, repeatable |
| API contracts, type mismatches, compilation | Automated (build + type check) | Compiler is the verifier |
| Button handlers, navigation, UI state | **Manual** | No unit test can click a button |
| Data persistence across launches | **Manual** | Requires app restart cycle |
| Lifecycle behavior (foreground/background) | **Manual** | Requires OS-level interaction |
| Visual appearance, layout, animation | **Manual** | Requires human eyes |
| Concurrency, race conditions | **Manual** + automated if possible | Timing-dependent |

For issues that span both categories (e.g., a logic fix that manifests as a UI change), use BOTH: run the automated test AND present manual verification steps.

##### Rules

- If automated verification is chosen, the test must actually run and pass — do not mark Resolved based on the test existing but not being run.
- If manual verification is chosen, do not mark Resolved until the user explicitly confirms. A non-response is not confirmation. "I'll check later" means the issue stays Open.
- Record the verification type and result in the ISSUES.md entry's `Verification` field when marking Resolved. This creates an audit trail.
- When batching multiple fixes, each issue gets its own verification. Do not batch-resolve issues under a single "build passes" verification.

#### Discipline Skill Methodology

Read `supporting-files/routing-triggers.md` for trigger definitions and thresholds.

SKILL.md instructions cannot programmatically invoke other skills. Instead, announce the methodology shift and follow the referenced skill's methodology inline. This matches the pattern used in builder agent definitions.

**Triggers and actions:**

| Pattern | Action |
|---------|--------|
| Fix fails twice on same issue (FR-014) | "This looks like it needs systematic debugging -- I'll follow the systematic-debugging skill's 4-phase methodology." Then execute: (1) Root Cause Investigation, (2) Pattern Analysis, (3) Hypothesis and Testing, (4) Implementation. **Counter persistence:** Fix attempt counts are persisted in the Fix Attempts section of iterate-status.md (not in-memory). Re-read before checking. |
| New behavior with testable criteria (FR-015) | "This is a good candidate for TDD -- I'll follow the RED-GREEN-REFACTOR cycle." Then execute the TDD methodology inline. |
| Multiple symptoms suggest shared root cause (FR-014) | "These issues might share a root cause -- investigating systematically before fixing individually." Route to systematic-debugging 4-phase methodology. |
| Issue about to be marked Resolved (FR-016) | Before changing status, run verification-before-completion: identify the verification command, run it, read full output, confirm the fix actually works. Only then mark Resolved. |
| Regression prevention for critical fix (FR-015) | "This was a critical fix -- writing a regression test to prevent recurrence." Route to TDD for a regression test covering the root cause. |

If a discipline skill's SKILL.md is unavailable (cannot be read), log a warning and fall back to inline problem-solving. Do not block progress.

#### Proactive Observations

Read `supporting-files/observation-categories.md` for the full list of categories, scan heuristics, and default severities.

When you notice a potential issue during work (dead code, unhandled error paths, hardcoded values, TODOs, empty handlers, etc.):
1. Surface it once with severity context and a brief explanation of why it matters
2. Use AskUserQuestion: "Log this as a new issue in ISSUES.md?" with options:
   1. "Yes, log it" — append a new entry to ISSUES.md
   2. "No, skip it" — do not log, move on
3. Defer to user direction -- do not insist on fixing it

Do not resurface the same observation after the user has acknowledged or dismissed it.

#### PROJECT.md Drift Detection

When encountering code or files that contradict PROJECT.md (FR-019):
- Mention the discrepancy once: "PROJECT.md says Auth is in Auth/ but I found auth logic in Utilities/ -- you may want to update the manifest."
- Do not auto-correct PROJECT.md
- Do not repeat the observation if already mentioned

### EXIT

When the user ends the session (says "done", "that's it", "let's stop", etc.):

**1. Determine exit type:**
Use AskUserQuestion: "How do you want to exit?" with options:
   1. "Checkpoint (Recommended)" — ship changes and clear context, but preserve iterate-status.md so the next session can resume where this one left off. Use this when you plan to continue iterating in a new session.
   2. "Full exit" — ship changes and clean up all session state. Use this when you're done iterating on this project for now.

**2. Pending agents:**
If background agents are still in-progress: warn the user and offer to wait for completion or abandon. Do not auto-merge unreviewed changes.

**3. Update ISSUES.md (FR-009, FR-010):**
Ensure all status changes from this session are persisted. Re-read ISSUES.md from disk, apply any pending updates.

**4. Session summary (FR-011):**
Produce a structured summary:
- **Resolved:** N issues (list them)
- **Added:** N issues (list them)
- **Still open:** N issues (show top 3 by priority)
- **Suggested next priorities:** Top 3 issues to tackle next session

**5. Ship changes (FR-012):**
If the session produced code changes:
   - Present a brief summary: files changed, issues resolved, issues added.
   - Check the current branch with `git branch --show-current`.
   - Use AskUserQuestion with shipping options appropriate to the branch:
     - If on a feature branch:
       1. "Create PR (Recommended)" — commit, push feature branch, create PR targeting main
       2. "Commit only (local, no push)" — commit on the current branch, no push
       3. "Leave uncommitted" — skip
     - If on main:
       1. "Create branch and PR (Recommended)" — create a feature branch, commit, push, create PR targeting main
       2. "Commit only (local, no push)" — commit without pushing
       3. "Leave uncommitted" — skip
   - Invoke `Skill('github')` to handle the selected option. The GitHub skill handles commit conventions, PR templates, worktree cleanup, and push operations.

**6. Archive integrity check:**
Verify that no Resolved issues remain in ISSUES.md (they should have been moved to ISSUES-ARCHIVE.md on resolve). If any are found, move them now.

**7. Clean up iterate-status.md:**
Behavior depends on exit type selected in step 1:
- **Checkpoint exit**: Keep iterate-status.md intact. Remove only completed and failed agent entries. Preserve queued tasks, fix attempt counters, and session metadata so the next session resumes seamlessly.
- **Full exit**: Remove completed and failed entries. Flag any in-progress entries as abandoned. If no entries remain, delete the file.

## Edge Cases

Most edge cases are handled inline in their respective sections. This is a quick-reference index.

| # | Case | Handling | Defined In |
|---|------|----------|-----------|
| 1 | No PROJECT.md | Propose draft from template, user reviews | ORIENT |
| 2 | No ISSUES.md | Create empty with standard header | ORIENT |
| 3 | ISSUES-ARCHIVE.md missing | Create with standard header on first resolve | Issue Management (Archival on resolve) |
| 4 | Background builder fails | Report with error context, offer retry/alt/skip | WORK LOOP (On failure) |
| 5 | File overlap detected | Queue task, auto-dispatch when blocker completes | WORK LOOP (File overlap check, Unblock queued tasks) |
| 6 | Discipline skill unavailable | Log warning, fall back to inline | Discipline Skill Methodology |
| 7 | Pending agents on exit | Warn user, offer wait or abandon | EXIT (Pending agents) |
| 8 | PROJECT.md drift | Mention once, do not auto-correct | PROJECT.md Drift Detection |
| 9 | Context compaction | Re-read iterate-status.md + ISSUES.md index (not full file) on state-dependent decisions; load full entries on-demand | Context Compaction Resilience |
| 10 | Git worktrees unavailable | Warn user, fall back to sequential dispatch | See below |
| 11 | Concurrent iterate sessions | Do not run two iterate sessions on the same project simultaneously — shared state files (ISSUES.md, iterate-status.md) will corrupt | N/A — hard constraint |

**Edge case 10 detail (not covered elsewhere):** If `git worktree add` fails (non-git project, shallow clone, unsupported environment), warn the user: "Git worktrees are not available -- parallel dispatch is disabled. Fixes will run sequentially in the main working tree." Fall back to sequential builder dispatch without `isolation: "worktree"`. All other skill behavior remains the same.

## Context Compaction Resilience

Long sessions will undergo context compaction, which loses in-memory state. This skill survives compaction through tiered re-reads — not full file reloads.

**After compaction, re-read using the same tiered strategy as LOAD:**

| What to re-read | When | How |
|-----------------|------|-----|
| iterate-status.md (full) | Before any dispatch or status decision | Read tool |
| ISSUES.md index (headers + status + labels) | Before any summary, dispatch, or status decision | Grep for `^## \|^\*\*Status:\*\*\|^\*\*Labels:\*\*` |
| Specific issue entry (full) | When actively working on that issue | Targeted Read with offset/limit |
| PROJECT.md | Only if orientation is needed for the current task | Read tool |
| Supporting files | Only when their feature triggers | Read tool |

**Do NOT re-read the entire ISSUES.md after compaction.** The index grep gives you everything needed for summaries, dispatch decisions, and file overlap checks. Full entries are loaded on-demand for the issue being worked on.

State-dependent decisions include:
- Displaying issue summaries or counts → index grep
- Checking file overlap before dispatch → iterate-status.md + index grep
- Determining which agents are active → iterate-status.md
- Updating issue statuses → targeted read of the specific entry
- Producing the session summary on exit → index grep

Do not cache issue state, agent state, or project structure in conversation memory. The files are the source of truth. If you are unsure whether you have current state, re-read the appropriate tier. The cost of an index grep is negligible compared to reading 100KB+ of issue descriptions that won't be used.

**State reconciliation rule:** If iterate-status.md and ISSUES.md disagree (e.g., iterate-status.md shows an agent completed for BUG-005 but ISSUES.md still shows BUG-005 as Open), ISSUES.md is authoritative for issue status and iterate-status.md is authoritative for agent/orchestration status. Reconcile by updating the less-authoritative file to match. In practice: an issue is not Resolved until ISSUES.md says so, regardless of what iterate-status.md claims about agent completion.

## Stage Handoff

When this skill completes (session exit), produce a structured handoff as part of the session summary.

**Schema:**
- **Output file:** `{PROJECT_DIR}/ISSUES.md` (updated)
- **Issues resolved:** {N} (list issue IDs)
- **Issues remaining:** {N} open issues by priority
- **Session summary:** 2-3 sentences describing what was accomplished
- **Key decisions:** 3-5 bullet points of decisions made during the session
- **Open questions:** Unresolved items or deferred observations
- **Scope:** Files and modules touched during this session

## Supporting Files

- `supporting-files/issues-md-format.md` -- ISSUES.md entry format: prefixed IDs, labels, status, description, files, acceptance criteria
- `supporting-files/project-md-template.md` -- PROJECT.md 4-section template: Overview, Feature Map, Key Files, Current State
- `supporting-files/iterate-status-format.md` -- iterate-status.md format: active agents table, queued tasks, cleanup rules
- `supporting-files/routing-triggers.md` -- Discipline skill routing trigger definitions and thresholds
- `supporting-files/observation-categories.md` -- Proactive observation categories, scan heuristics, and default severities

