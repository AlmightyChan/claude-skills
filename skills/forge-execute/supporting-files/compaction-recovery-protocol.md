# Compaction Recovery Protocol — Forge Execute

When context compaction occurs during a forge-execute session, the lead loses in-memory state about the team, active teammates, and current tier progress. This protocol recovers from that state using on-disk artifacts.

## Detection

You are in a compaction recovery state if:
- You have no memory of creating a team or spawning teammates
- But `FORGE_STATUS_PATH` (projects/{project}/build-status--forge-{project}.md) exists on disk
- And/or `~/.claude/teams/{TEAM_NAME}/config.json` exists

## Recovery Steps

### 1. Re-read State (parallel)
Read these files simultaneously:
- `FORGE_STATUS_PATH` — plan status table, tier progress, merge log
- `MANIFEST_PATH` — DAG, tiers, component plans, build command
- This protocol file (you're reading it now)

### 2. Assess Team
- Read `~/.claude/teams/{TEAM_NAME}/config.json` for listed teammates
- Send status check to each listed teammate via SendMessage:
  `SendMessage({ type: "message", recipient: "{name}", content: "STATUS_CHECK: report current state", summary: "Status check" })`
- Wait 30 seconds for responses

### 3. Reconcile
Cross-reference three sources:
- **Forge status file**: which plans are COMPLETE/IN_PROGRESS/PENDING/FAILED
- **Build status files**: per-plan `build-status--{slug}.md` in worktrees — check task completion
- **Worktrees**: `git worktree list` — which worktrees still exist

Build a reconciled view:
| Plan | Forge Status | Build Status Tasks | Worktree Exists | Action |
|------|-------------|-------------------|-----------------|--------|
| {slug} | COMPLETE | 10/10 PASS | No | Skip |
| {slug} | IN_PROGRESS | 5/10 PASS | Yes | Respawn |
| {slug} | PENDING | N/A | No | Queue |
| {slug} | FAILED | 3/10 PASS | Yes | Evaluate |

### 4. Rebuild Team
- `TeamDelete` the stale team (if it exists and has no responsive members)
- `TeamCreate({ team_name: "{TEAM_NAME}", description: "Forge execution (recovered): {N} plans remaining" })`
- Respawn teammates ONLY for incomplete plans in the current tier
- Each respawned teammate gets the same spawn prompt from `teammate-orchestrator-prompt.md`
- Execute's built-in resume check (via build-status file) handles per-task recovery

### 5. Resume Execution
- Continue from the next incomplete tier per forge status
- Completed tiers are skipped entirely
- Merged plans within the current tier are skipped
- Only IN_PROGRESS and PENDING plans get teammates

### 6. Prevention
To minimize compaction risk:
- Keep spawn prompts concise (800 token budget per teammate-orchestrator-prompt.md)
- Shut down teammates promptly after their plan completes (via shutdown_request)
- Cap active worktrees at WORKTREE_CAP (3)
- Lead monitors via disk reads, not message accumulation
- No per-task progress messages from teammates (protocol enforcement)
