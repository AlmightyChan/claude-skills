---
name: forge-execute
version: 0.1.1
description: "Multi-plan orchestrator for forge projects. Reads manifest, schedules tiers, spawns execute teammates, auto-merges."
argument-hint: "[project-name]"
model: opus
disallowed-tools: [EnterPlanMode]
---

# Forge Execute

- **Agents:** orchestrator-invoked, agent-teams

Multi-plan orchestrator for forge projects. Reads a forge manifest, schedules tier-by-tier execution via agent teams, auto-merges between tiers, batches questions, and runs a review team.

## Variables

PROJECT_NAME: $ARGUMENTS
FORGE_PROJECT_DIRECTORY: projects/$ARGUMENTS/
PLAN_DIRECTORY: projects/$ARGUMENTS/plans/
MANIFEST_PATH: projects/$ARGUMENTS/plans/manifest.md
STATUS_PATH: projects/$ARGUMENTS/status.md
FORGE_STATUS_PATH: projects/$ARGUMENTS/build-status--forge-$ARGUMENTS.md
WORKTREE_CAP: 3
TEAM_NAME: forge-exec-$ARGUMENTS

## Message Protocol

Strict protocol to prevent lead context saturation. Teammates send ONLY these types:

| Message Type | Direction | Max Chars | Content |
|-------------|-----------|-----------|---------|
| `PLAN_COMPLETE` | teammate -> lead | 200 | plan slug, task count, pass/fail |
| `QUESTION` | teammate -> lead | 500 | plan slug, task ID, question text |
| `ESCALATION` | teammate -> lead | 300 | plan slug, task ID, strike count, root cause |
| `ANSWER` | lead -> teammate | 500 | task ID, user's answer |

No per-task progress messages. No heartbeats. Lead monitors progress by reading `FORGE_STATUS_PATH` from disk.

## Workflow

### Step 1 — Validate Input

- If no `PROJECT_NAME`, stop and ask via AskUserQuestion.
- Verify `FORGE_PROJECT_DIRECTORY`, `MANIFEST_PATH` exist. If manifest missing: "Run `/forge-plan {project-name}` first."
- Read `STATUS_PATH` Prerequisites section. If unchecked items exist, present via AskUserQuestion: "Resolve prerequisites" / "Acknowledge and continue" / "Mark as resolved".

### Step 2 — Ingest Manifest

- Read `manifest.md`. Parse: DAG, parallelism tiers, component plans table, execution strategy (including build command), high-conflict files, foundation exports.
- **Verify** each component plan exists (do NOT read full contents — teammates read them). Check for `integration.md` (may not exist for single-component projects).
- If no build command in Execution Strategy, infer from Project Topology type. If ambiguous, ask user.
- Present execution summary: tier count, plans per tier, estimated total units.

### Step 3 — Resume Detection

- Check for `FORGE_STATUS_PATH`. If found:
  - Parse per-plan status (COMPLETE/IN_PROGRESS/PENDING/FAILED) and tier progress.
  - Offer resume or restart via AskUserQuestion.
  - Skip completed tiers entirely.
- Iterate only tiers present in manifest's Parallelism Tiers section (handle no-foundation case).
- Stale team check: if `~/.claude/teams/{TEAM_NAME}/config.json` exists, `TeamDelete` first.

### Step 4 — Initialize

- Write `FORGE_STATUS_PATH` with plan status table (plan, tier, status, branch, merged), tier progress, question log, merge log.
- **Provisioning:** Read the first component plan's `## Provisioning` section (if it exists). Run provisioning once (generate project CLAUDE.md, initialize decision log) before any teammates are spawned. This prevents provisioning races between parallel teammates.
- `TeamCreate({ team_name: "{TEAM_NAME}", description: "Forge execution: {N} plans across {M} tiers" })`

### Step 5 — Tier Loop

For each tier present in manifest's Parallelism Tiers:

**5a. Create worktrees:** For each plan in the tier (up to WORKTREE_CAP; if tier exceeds cap, process in sequential batches):
```bash
git worktree add .claude/worktrees/feat-{plan-slug} -b feat/{plan-slug}
```
Pre-flight: verify `.gitignore`, check worktree cap. Create sequentially (fast, avoids race conditions).

**5b. Spawn execute teammates:** For each plan, spawn using assembled prompt from `supporting-files/teammate-orchestrator-prompt.md`:
```
Agent({
  description: "Execute plan: {plan-slug}",
  prompt: "{assembled prompt}",
  team_name: "{TEAM_NAME}",
  name: "exec-{plan-slug}",
  subagent_type: "general-purpose",
  model: "sonnet"
})
```
**Sonnet for teammates, Opus for lead.** Teammates are procedural orchestrators; complex decisions escalate to the lead. Branch name source: manifest Component Plans table `Branch` column.

**5c. Monitor + Rolling Merge:**
After spawning all tier teammates, render a plan status table in the conversation:
```
| Plan | Tier | Teammate | Status | Branch | Merged |
|------|------|----------|--------|--------|--------|
| <slug> | 1 | exec-<slug> | running | feat/<slug> | — |
```
Re-render the table after each `PLAN_COMPLETE` message with updated status and merge result (`merged` / `failed`). When all plans in the tier are merged, render the final tier table before advancing to the next tier.

As teammates report via SendMessage:
- On `PLAN_COMPLETE`: immediately run merge pipeline for that plan:
  1. Pre-merge build in worktree (using build command from manifest)
  2. `git merge feat/{slug} --no-ff -m "merge: {slug} (tier {N})"`
  3. If conflict: check High-Conflict Files table, escalate true conflicts to user
  4. Post-merge build on main
  5. If build fails: identify responsible plan, dispatch fix builder, re-verify
  6. Worktree + branch cleanup
  7. Update `FORGE_STATUS_PATH` merge log (states: PENDING / MERGED / FAILED)
  8. Shut down that teammate via `SendMessage` shutdown_request
- On `QUESTION`: add to question buffer
- On `ESCALATION`: collect for user attention

**5d. Question routing:**
Present each question to user as it arrives via AskUserQuestion (no batching in v0.1.0 — simpler, nearly as good UX). Route answer back:
```
SendMessage({ type: "message", recipient: "exec-{slug}",
  content: "ANSWER for {task-id}: {user answer}",
  summary: "Answer for {task-id}" })
```
**Decision log:** After routing each answer, append entry to `{project-dir}/docs/decision-log.md` (lead has full context: question + answer + plan + task).

**5e. Tier complete:** When all tier plans are merged:
- Re-read `FORGE_STATUS_PATH` from disk (compaction resilience)
- Advance to next tier

### Step 6 — Integration Plan

- After all tiers merge to main.
- If `integration.md` doesn't exist (single-component project), skip to step 7.
- Spawn single teammate. For integration, `{worktree_path}` in the spawn prompt is set to the **repo root absolute path** (not a worktree — integration runs on main).
- Handle any questions.

### Step 7 — Review Team

After integration passes, spawn review team. Fixed composition (3 reviewers + conditional designer):

| Reviewer | Focus |
|----------|-------|
| `review-auditor` | Architecture coherence, code quality, error handling, boundary conditions, security |
| `review-validator` | Integration seams, cross-component contracts, validation completeness |
| `review-integration` | End-to-end flow tracing, data flow verification |
| `review-designer` | UX coherence, accessibility, edge-case states *(skip for non-UI projects — detect from Project Topology)* |

- All run in parallel as teammates on the same team.
- Collect findings, classify: BLOCKING / IMPORTANT / MINOR.
- Address BLOCKING findings (spawn fix builders, max 2 cycles).
- Write unaddressed IMPORTANT/MINOR findings to `{project-dir}/ISSUES.md` for downstream `/iterate` consumption.

### Step 8 — Document Lifecycle

- Archive plan files: `mv plans/*.md {project-dir}/docs/archive/plan/`
- Move `FORGE_STATUS_PATH` to `{project-dir}/docs/builds/`
- Move per-plan `build-status--{slug}.md` files to `{project-dir}/docs/builds/`

### Step 9 — Report, Ship, and Complete

- Summary: plans executed, tiers completed, task counts, files changed, review findings.
- Options via AskUserQuestion:
  1. "Create PR" — invoke `Skill('github')`
  2. "Tag release" — tag main with project version
  3. "Iterate on findings" — invoke `/iterate {project-dir}` (if ISSUES.md has entries)
  4. "Keep as-is"
- `TeamDelete({ team_name: "{TEAM_NAME}" })`
- Write `## Completion: DONE` to archived `FORGE_STATUS_PATH`.
- Output: `[SKILL_COMPLETE:forge-execute]`

## Session Resumability

State detection (check in order):
1. No `FORGE_STATUS_PATH` -> start from step 1
2. `FORGE_STATUS_PATH` exists, no `## Completion: DONE` -> parse progress, offer resume (step 3)
3. `## Completion: DONE` present -> already complete, inform user

## Error Handling

| Error | Response |
|-------|----------|
| Merge conflict | Check High-Conflict Files, escalate to user if true conflict |
| Plan failure (3 strikes) | Cascade-skip dependents per DAG. Options: Skip / Abort / Retry |
| Teammate unresponsive | Send status check via SendMessage. If no response in 30s, respawn with recovery context (reuse execute's resume check via build-status file). |

## Compaction Recovery

If context compaction occurs, follow `supporting-files/compaction-recovery-protocol.md`:
1. Detect: no memory of team, but `FORGE_STATUS_PATH` exists on disk
2. Re-read: `FORGE_STATUS_PATH`, manifest, recovery protocol (in parallel)
3. Assess: read team config, send status checks to teammates
4. Reconcile: cross-reference forge-status with build-status files and worktrees
5. Rebuild: TeamDelete stale -> TeamCreate -> respawn only incomplete tier teammates
6. Resume: continue from next incomplete tier
7. Prevent: concise spawn prompts, prompt teammate shutdown, cap at WORKTREE_CAP

## Stage Handoff

```
- Output file: build-status--forge-{project-name}.md (in {project-dir}/docs/builds/)
- Plans executed: {N}/{total}
- Plans failed: {N} (with reasons)
- Tiers completed: {M}/{total}
- Tasks: {completed}/{total} across all plans
- Files changed: aggregate count
- Review findings: {N} BLOCKING addressed, {M} IMPORTANT/{P} MINOR written to ISSUES.md
- Key decisions: from decision log
- Open questions: unresolved escalations
```

## Related Skills

- `forge-plan` — Upstream: produces the manifest and component plans
- `execute` — Inner skill: each teammate runs execute in teammate mode
- `github` — Downstream: handles PR creation
- `iterate` — Downstream: handles IMPORTANT/MINOR review findings

## Completion Token

`[SKILL_COMPLETE:forge-execute]`
