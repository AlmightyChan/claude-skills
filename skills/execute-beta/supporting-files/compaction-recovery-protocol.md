# Compaction Recovery Protocol

Recovery procedure for the execute-beta lead when context compaction occurs during agent teams execution. Compaction destroys in-memory team state but leaves disk artifacts intact. This protocol restores execution from disk state.

---

## 1. Detection

Compaction has occurred if any of these are true:

- You have no memory of creating a team or spawning teammates, but `build-status--{plan-slug}.md` exists in the working directory.
- You cannot recall the team name, teammate names, or current wave number.
- The conversation context contains a compaction marker or you notice a discontinuity in your reasoning.

If in doubt, assume compaction occurred and run this protocol. The cost of a false positive (re-reading state unnecessarily) is negligible compared to acting on stale state.

---

## 2. Re-read State

Read these files in parallel — they are independent:

1. **This protocol** — `supporting-files/compaction-recovery-protocol.md` (you are reading it now).
2. **Build status** — `build-status--{plan-slug}.md` in the working directory. Contains completed task entries, dispatch rules, and the plan slug.
3. **Plan file** — The plan at `PATH_TO_PLAN`. Contains task definitions, team members, spawn waves, and acceptance criteria.
4. **Spawn prompts** — `supporting-files/teammate-spawn-prompts.md`. Needed to reconstruct teammate dispatch prompts.

After reading, extract:
- Plan slug (from build-status filename)
- Team name: `exec-{plan-slug}`
- Completed tasks (status = PASS or SKIPPED in build-status)
- Failed tasks (status = FAIL with retry count)
- Current wave (highest wave number with in-progress or incomplete tasks)

---

## 3. Assess Team State

Check whether the team still exists and teammates are responsive:

1. Read `~/.claude/teams/exec-{plan-slug}/config.json`. If the file does not exist, the team was never created or was already deleted — skip to step 5.
2. Send a status check to each teammate listed for the current wave:
   ```
   SendMessage({
     type: "message",
     recipient: "{teammate-name}",
     content: "Status check after compaction. Report your current task state.",
     summary: "Post-compaction status check"
   })
   ```
3. Wait 30 seconds for responses.
4. Classify each teammate:
   - **Responsive + working**: Leave in place, do not replace.
   - **Responsive + idle**: Can be reused for new tasks.
   - **Unresponsive**: Mark for replacement.

---

## 4. Reconcile Tasks

Cross-reference the build-status file with the shared task list:

1. Run `TaskList({})` to get all tasks and their current status.
2. For each task in the plan:
   - **PASS in build-status**: Verify artifacts exist on disk. If yes, mark as completed in task list (if not already). If artifacts missing, queue for re-execution.
   - **FAIL in build-status**: Check retry count. If < 3, queue for retry. If >= 3, mark as requiring user intervention.
   - **in_progress in build-status but no responsive teammate**: Check disk for partial artifacts. If substantial progress exists, queue for completion by a new teammate. If no artifacts, queue for full re-execution.
   - **Not in build-status**: Task was never started. Queue normally.

---

## 5. Rebuild Team

If the team is stale, unresponsive, or missing:

1. `TeamDelete({ team_name: "exec-{plan-slug}" })` — clean up stale team config. Skip if no team exists.
2. `TeamCreate({ team_name: "exec-{plan-slug}", description: "Executing: {plan name} (recovered)" })`
3. Determine the current wave from the reconciliation in step 4.
4. Spawn replacement teammates only for incomplete tasks in the current wave.
5. Use the spawn prompts from `supporting-files/teammate-spawn-prompts.md` with updated context:
   - Include: "This is a recovery spawn after compaction. Prior work may exist on disk — check before overwriting."
   - Include: worktree path, build-status path, task IDs, plan path.

---

## 6. Resume

Continue execution from the next incomplete task per build-status:

1. Re-read the `## Dispatch Rules` section of `build-status--{plan-slug}.md`.
2. Follow the standard dispatch loop from execute-beta step 5d (wave-based teammate spawning).
3. Skip completed tasks. Retry failed tasks (respecting strike count).
4. Continue wave progression normally.

---

## 7. Prevention

Minimize the impact of future compactions:

- **Concise spawn prompts**: Keep teammate prompts focused. Large prompts contribute to context bloat.
- **Prompt teammate shutdown**: Shut down completed teammates immediately. Do not let idle teammates consume team slots.
- **Wave size limit**: Max 3-4 active teammates per wave. More teammates = faster compaction.
- **Proactive state re-reads**: After every wave completion, re-read build-status from disk before planning the next wave. Do not rely on in-memory state across wave boundaries.
- **Disk is truth**: When in-memory state and disk state conflict, disk wins. Always.
