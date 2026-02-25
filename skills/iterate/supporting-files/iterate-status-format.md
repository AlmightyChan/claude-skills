# iterate-status.md Format

This file defines the format for `iterate-status.md`, the per-session status file that tracks active background agents during an iterate session. It is written to the project root alongside ISSUES.md.

**Purpose:** Orchestration state must survive context compaction. The iterate skill re-reads this file before every dispatch decision to prevent overlapping work after memory loss.

---

## Format

```markdown
# Iterate Session Status

**Session started:** {ISO 8601 timestamp, e.g., 2026-02-22T14:30:00Z}
**Project:** {project name from PROJECT.md}

## Active Agents

| Task ID | Issue | Target Files | Worktree Branch | Status | Started |
|---------|-------|-------------|-----------------|--------|---------|
| {task_id} | {ISSUE-ID} | {comma-separated file paths} | {branch name} | {in-progress|completed|failed} | {ISO 8601} |

## Queued

| Issue | Target Files | Blocked By |
|-------|-------------|------------|
| {ISSUE-ID} | {comma-separated file paths} | {ISSUE-ID of blocking agent} |

## Fix Attempts

| Issue | Attempts | Last Failure |
|-------|----------|-------------|
| {ISSUE-ID} | {integer} | {brief description of most recent failure} |
```

The Queued section is only present when tasks are blocked by file overlap with active agents. Omit the section entirely if no tasks are queued.

The Fix Attempts section tracks how many times a fix for an issue has failed, persisting the counter that triggers systematic-debugging routing (threshold: 2). Omit the section entirely if no fix attempts have failed. Remove an issue's entry when it is marked Resolved or Trashed.

---

## Example

```markdown
# Iterate Session Status

**Session started:** 2026-02-22T14:30:00Z
**Project:** Cadence

## Active Agents

| Task ID | Issue | Target Files | Worktree Branch | Status | Started |
|---------|-------|-------------|-----------------|--------|---------|
| task_a1b2c3 | BUG-005 | Views/Onboarding/OnboardingView.swift, Auth/Auth.swift | iterate/bug-005 | in-progress | 2026-02-22T14:32:00Z |
| task_d4e5f6 | FEAT-008 | Views/Plans/PlansView.swift, UseCases/ManualSuggestionUseCase.swift | iterate/feat-008 | completed | 2026-02-22T14:35:00Z |
| task_g7h8i9 | BUG-029 | Views/Shared/EmptyStateView.swift | iterate/bug-029 | failed | 2026-02-22T14:40:00Z |

## Queued

| Issue | Target Files | Blocked By |
|-------|-------------|------------|
| BUG-006 | Views/Onboarding/OnboardingView.swift | BUG-005 |
```

---

## Re-Read Protocol

**Always re-read `iterate-status.md` from disk before:**
- Dispatching a new background agent
- Checking whether a queued task can be unblocked
- Reporting session status to the user
- Making any dispatch decision

Do not rely on conversation memory for agent state. Context compaction may have removed earlier agent dispatch records. The file on disk is the single source of truth.

---

## Cleanup Rules

**On session exit:**

1. Remove all entries with status `completed` or `failed`
2. Flag any entries with status `in-progress` as `abandoned` (change status to `abandoned`)
3. Remove the Queued section entirely
4. If no entries remain, delete the `iterate-status.md` file

**On session start (re-entering a project):**

1. If `iterate-status.md` exists, check for `abandoned` entries
2. Report abandoned entries to the user: "{ISSUE-ID} was in-progress when the last session ended. The worktree branch `{branch}` may contain partial work."
3. User decides: resume the work, discard the worktree, or leave it for later
4. Remove abandoned entries after user decision

---

## Status Values

| Status | Meaning |
|--------|---------|
| `in-progress` | Agent is currently running in a background worktree |
| `completed` | Agent finished successfully; changes are ready for user review |
| `failed` | Agent encountered an error; see agent output for details |
| `abandoned` | Agent was in-progress when the session ended without cleanup |

---

## Branch Naming

Worktree branches follow the pattern: `iterate/{issue-id-lowercase}`

Examples:
- `iterate/bug-005`
- `iterate/feat-008`
- `iterate/polish-003`
