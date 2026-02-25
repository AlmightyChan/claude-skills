---
name: execute-beta
description: "Use when you have an implementation plan ready for agent-teams-driven execution"
argument-hint: [path to plan document]
model: opus
hooks:
  Stop:
    - hooks:
        - type: command
          command: >-
            uv run $HOME/.claude/hooks/validators/validate_file_contains.py
            -d "$PWD" -e .md --prefix build-status --max-age 900
            --contains '## '
version: 0.1.0
---

# Execute (Beta — Agent Teams)

Read and execute the implementation plan at `PATH_TO_PLAN` using Claude Code's experimental agent teams feature. The lead creates a team, spawns builder/validator teammates in waves, and coordinates via the shared task list and SendMessage.

This is a beta variant of the `execute` skill. It uses `TeamCreate`/`SendMessage` for inter-agent coordination instead of direct `Task` tool dispatch.

## Variables

PATH_TO_PLAN: $ARGUMENTS

## Workflow

1. **Validate input**: If no `PATH_TO_PLAN` is provided, STOP and ask the user (AskUserQuestion). Reject paths containing `..` or pointing outside the current project directory.

2. **Read the plan**: Read the file at `PATH_TO_PLAN`. Think hard about it. Also read:
   - `supporting-files/compaction-recovery-protocol.md` — recovery procedure if compaction occurs during execution.
   - `supporting-files/teammate-spawn-prompts.md` — templates for assembling teammate spawn prompts.

2.5. **Determine parallelism mode**:
   - Check if `PATH_TO_PLAN` arguments include `--parallelism serial|topology|aggressive`.
   - If not, check the plan's `## Project Topology` section for `Parallelism mode:`.
   - If neither exists, default to `serial`.
   - `serial`: all tasks sequential, ignore `Parallel: true` flags (safest, always correct).
   - `topology`: parallelize across different build units only (recommended for multi-target/multi-module/monorepo projects).
   - `aggressive`: also parallelize within build units for interpreted languages using file-set independence (for Python, TypeScript projects).

3. **Worktree isolation**: Check current branch with `git branch --show-current`.
   - If on `main` or `master`:
     - Follow `.claude/rules/worktree-conventions.md` for branch derivation, sanitization, creation, dispatch path rules, and Xcode worktree builds.
     - Derive branch name from plan file name (e.g., `docs/plan/add-auth.md` -> `feat/add-auth`).
     - Create worktree: `git worktree add .claude/worktrees/<branch-name> -b <branch-name>`
     - If it fails (branch exists, dirty state): ask user whether to proceed in current directory or abort.
     - All subsequent work uses absolute paths within the worktree. Worktree applies to the whole team — all teammates share the same worktree.
   - If already on a feature branch: proceed in current directory.

4. **Create task list and team**: Before dispatching any work, populate the shared task list and create the team.
   - For each task in the plan's `## Step by Step Tasks`:
     ```
     TaskCreate({ subject: "{task-id}: {brief description}", description: "{full task text from plan}", activeForm: "Executing {task-id}" })
     ```
   - After all tasks are created, set dependencies:
     ```
     TaskUpdate({ taskId: "{id}", addBlockedBy: ["{dep-id-1}", "{dep-id-2}"] })
     ```
   - Assign owners based on the plan's `Assigned To` field:
     ```
     TaskUpdate({ taskId: "{id}", owner: "{agent-name}" })
     ```
   - Maintain a mapping of plan task IDs to TaskCreate-returned task IDs for use in all subsequent `TaskUpdate` calls.
   - If resuming (see resume check below): mark already-completed tasks as `TaskUpdate({ taskId: "{id}", status: "completed" })`.
   - **Create the team**:
     ```
     TeamCreate({ team_name: "exec-{plan-slug}", description: "Executing: {plan name}" })
     ```

5. **Execute tasks**:

   **5a. Plan-scoped build status file**: Derive a plan-specific filename to allow concurrent executions of different plans in the same working directory. Take the plan filename from `PATH_TO_PLAN` (e.g., `docs/plan/plan-iterate-skill.md` → `plan-iterate-skill`), strip the path and `.md` extension, and use it as the slug: `build-status--{plan-slug}.md`. All subsequent references to the build status file in this execution use this derived name.

   **5b. Resume check**: Before dispatching any tasks, check for an existing `build-status--{plan-slug}.md` in the working directory.
   1. If found, parse completed task entries (status = PASS).
   2. For each completed task, verify its output artifacts exist on disk (check files listed in the task's action items).
   3. If artifacts exist: mark task as SKIPPED.
   4. If artifacts are missing: queue for re-execution.
   5. Present summary via AskUserQuestion: "Found {completed}/{total} tasks completed. Resume from task {next-task-id}?" with options:
      - "Resume (Recommended)" — skip verified tasks, continue from next incomplete
      - "Restart from scratch" — delete `build-status--{plan-slug}.md`, execute all tasks
   - If no `build-status--{plan-slug}.md` exists, proceed normally.
   - **Stale team check**: Check for `~/.claude/teams/exec-{plan-slug}/config.json`. If found (stale team from prior execution), `TeamDelete({ team_name: "exec-{plan-slug}" })` before creating the new team.

   **5c. Dispatch rules persistence**: At the start of execution, write a `## Dispatch Rules` header into `build-status--{plan-slug}.md` containing these critical rules:
   - Builder/validator pairs: spawn builder teammate, wait for completion, run build gate, spawn or message validator teammate. 3-strike retry via SendMessage.
   - Topology-aware dispatch: read `## Project Topology` from plan. If present, use build unit analysis for parallelism. If absent, default to serial. Respect parallelism mode flag (`serial | topology | aggressive`).
   - Wave-based spawning: read plan's Spawn Waves. Spawn Wave N, wait for completion, shut down, spawn Wave N+1.
   - Parallel dispatch: only when plan marks `Parallel: true` AND dependencies resolved AND file-overlap guard passes AND build unit does not overlap with any currently-running task's build unit.
   - Gap-flagging: scan builder output (via SendMessage) for escalation signals before dispatching validator.
   - Gotcha relay: scan all agent output for `## Gotchas Identified` section.
   - Type registry: build and include for parallel batches.
   - Final acceptance: dispatch as validator teammate, not inline.
   If `build-status--{plan-slug}.md` already exists and contains a `## Dispatch Rules` section (from a prior or resumed execution), overwrite it with the current rules to prevent stale rule drift.
   Before dispatching EACH task, re-read the `## Dispatch Rules` section of `build-status--{plan-slug}.md` to reload these rules. This makes the orchestrator resilient to context compaction.

   **5d. Wave-based teammate spawning**: For each spawn wave in the plan:

   1. Read wave's teammate list and task assignments from the plan's `### Spawn Waves` section.
   2. For each teammate in wave:
      a. Assemble spawn prompt from `supporting-files/teammate-spawn-prompts.md` (builder template, validator template, or non-cycling template as appropriate).
      b. Inject runtime values: worktree path, task IDs, build-status path, plan path, context files.
      c. Spawn the teammate:
         ```
         Task({
           description: "{name}: {role}",
           prompt: "{assembled spawn prompt}",
           team_name: "exec-{plan-slug}",
           name: "{teammate-name}",
           subagent_type: "{agent-type}",
           model: "sonnet"
         })
         ```
      d. For builders: spawn and wait. For validators: spawn only after paired builder completes and build gate passes.
   3. Monitor via `TaskList` polling + incoming `SendMessage` from teammates.
   4. When a builder goes idle or sends a completion message:
      a. Run pre-validator build gate (lead runs independently — see 5e).
      b. Build fails on builder's files → `SendMessage` fix instructions to builder (increment strike count).
         ```
         SendMessage({
           type: "message",
           recipient: "{builder-name}",
           content: "Build FAILED on files you touched. Errors:\n{build_errors}\nStrike {N}/3. Fix and mark complete.",
           summary: "Build gate failure for {task-id} (strike {N})"
         })
         ```
      c. Build passes → update task status, spawn paired validator teammate.
   5. When a validator completes (via SendMessage):
      a. PASS → update build-status, mark task completed.
      b. FAIL → `SendMessage` to builder with findings (increment strike count).
      c. 3 strikes → stop task, `AskUserQuestion` to the user.
   6. When all wave tasks complete:
      a. `SendMessage` shutdown acknowledgment to all wave teammates (no formal shutdown command — teammates complete their Task naturally).
      b. Proceed to next wave.

   **Task status updates**: Throughout the dispatch loop, keep the shared task list in sync:
   - Before dispatching a task: `TaskUpdate({ taskId: "{id}", status: "in_progress" })`
   - After a task passes validation: `TaskUpdate({ taskId: "{id}", status: "completed" })`
   - After a task fails and is not retried: `TaskUpdate({ taskId: "{id}", status: "cancelled" })`
   - After a task is skipped (resume): `TaskUpdate({ taskId: "{id}", status: "completed" })`
   - Use `TaskList({})` to review overall progress at wave boundaries or when re-reading state after compaction.

   **5e. Pre-validator build gate**: After a builder completes but before dispatching its paired validator, the lead MUST independently run a full-target build (not `-only-testing`, not a module-only build). Do not trust the builder's build claim. For Xcode projects, use `xcodebuild build -scheme {scheme} -derivedDataPath ./DerivedData 2>&1 | xcsift` via Bash; for other project types, use the project's standard build command. Do NOT use `mcp__xcode__BuildProject` (hangs from Claude Code CLI due to TCC — see `.claude/rules/xcode-mcp.md`).
   - If the build passes: spawn the validator teammate.
   - If the build fails on files the builder touched: `SendMessage` fix instructions to builder (counts toward 3-strike limit).
   - If the build fails on files NOT touched by this builder (cross-builder contamination): identify the responsible builder/task, `SendMessage` fix instructions to that builder, then retry the build gate. Do not spawn the validator until the build passes.

   **5f. Builder spawn prompt requirements**: Every builder spawn prompt MUST include these constraints (embedded in the spawn prompt template):
   1. "If you create new source files in an Xcode project, use `ToolSearch` to load and use `mcp__xcode__XcodeWrite` (which auto-registers files in project.pbxproj). Do NOT use the generic `Write` tool for Xcode source files, as it does not register them — unregistered files are invisible to the compiler."
   2. "For Xcode project builds and tests, use `xcodebuild` via Bash piped through `xcsift` for structured output (e.g., `xcodebuild build -scheme MyScheme -derivedDataPath ./DerivedData 2>&1 | xcsift`). Do NOT use `mcp__xcode__BuildProject` or `mcp__xcode__RunSomeTests` — they hang from Claude Code CLI due to TCC permissions. See `.claude/rules/xcode-mcp.md` for the full hybrid strategy. You MAY use Xcode MCP read-only tools (`DocumentationSearch`, `XcodeRefreshCodeIssuesInFile`, `XcodeListNavigatorIssues`) for inspection and diagnostics."
   3. "Write bottom-up: define types before referencing them. If a type from a dependency task doesn't exist on disk yet, do not reference it — write a compilation-safe stub or defer to the dependency."
   4. If the task includes a `**Context Files**` field, prepend to the spawn prompt: "Before starting, read the following reference document(s) and apply their decisions to your implementation: {context file paths}. For Brand OS documents: apply visual identity, voice/tone, motion, interaction patterns, and micro-copy guidelines specified there. For PRD documents: follow the functional requirements, behavioral specs (Given-When-Then), non-functional requirements, and technical constraints specified there. When both are loaded: the PRD defines WHAT to build and the Brand OS defines HOW it should look, sound, and feel."

   **5g. 3-strike retry via SendMessage**: When a validator reports FAIL for a builder's work:
   ```
   SendMessage({
     type: "message",
     recipient: "{builder-name}",
     content: "Validation FAILED for {task-id}. Findings:\n{detailed_findings}\nStrike {N}/3. Fix the issues and mark complete.",
     summary: "Fix request for {task-id} (strike {N})"
   })
   ```
   If the builder is no longer active (has shut down), spawn a replacement builder teammate with context about the prior failure:
   ```
   Task({
     description: "{name}-retry: Fix {task-id}",
     prompt: "{spawn prompt with failure context appended}",
     team_name: "exec-{plan-slug}",
     name: "{name}-retry",
     subagent_type: "builder",
     model: "sonnet"
   })
   ```
   After 3 strikes, stop and ask the user via `AskUserQuestion`.

   **5h. File-overlap guard**: Before dispatching a batch of `Parallel: true` builder tasks within a wave, extract the file paths each task will create or modify (from the plan's action items). If any file path appears in more than one task in the batch, those tasks CANNOT run in parallel — serialize them. Only dispatch tasks in parallel whose file sets are completely disjoint. This check is the final gate before parallel dispatch and overrides `Parallel: true` if overlap is detected. Log any overrides in `build-status--{plan-slug}.md`: "File overlap detected: {file} appears in tasks {task-a} and {task-b}. Serializing."

   **5i. Non-cycling agents** (researcher, critic, designer, specialist): These run once and return results via SendMessage. No validator pair. Spawn as teammates, collect output via `SendMessage`, append to `build-status--{plan-slug}.md`, and proceed to the next task.
   - **Researcher**: Spawn with research question. Output is a document written to docs/. Proceed when the teammate reports completion.
   - **Critic**: Spawn with artifact to review. Output is a PASS/REVISE/DROP verdict via SendMessage. On REVISE: `SendMessage` changes to the builder. On DROP: stop and ask the user.
   - **Designer**: Spawn with design question. Output is a design document or ADR. Proceed when document exists.
   - **Specialist**: Spawn with skill name in the prompt (e.g., "Invoke Skill('security-review') then audit src/auth/"). If the plan specifies `Specialist Skills`, include them in the prompt. Output is a findings report.

   **5j. Gotcha relay**: After any teammate completes (builder, validator, critic, designer, specialist, researcher), scan their SendMessage output for a `## Gotchas Identified` section. If found, extract entries. Read `docs/logs/gotcha-log.md` and check for duplicates (same title or substantially same description). For each new gotcha, append to the top of the log (below header) using the agent entry format:
   ```
   ## [{tag}] {Title}
   **Date:** {YYYY-MM-DD} | **Last Seen:** {YYYY-MM-DD} | **Occurrences:** 1
   **Source:** agent, {agent-name}

   {Description in 2-4 sentences.}

   ---
   ```
   If a duplicate is found, increment its Occurrences count and update Last Seen date.

   **5k. Compaction recovery**: If compaction occurs during execution:
   1. Re-read `supporting-files/compaction-recovery-protocol.md`.
   2. Re-read `build-status--{plan-slug}.md`.
   3. Re-read the plan file at `PATH_TO_PLAN`.
   4. Run `TaskList` to check task states.
   5. Check `~/.claude/teams/exec-{plan-slug}/config.json` for team state.
   6. If team is stale or unresponsive: `TeamDelete`, `TeamCreate`, spawn replacement teammates for incomplete wave.
   7. Resume from next incomplete task per build-status.
   Follow the full protocol in `supporting-files/compaction-recovery-protocol.md`.

   **Running tasks table**: Maintain a list of currently-running teammates with their task ID, teammate name, and build unit. Before spawning a new builder, check whether its build unit overlaps with any running teammate's build unit. If overlap exists in a compiled language, wait for the conflicting teammate to complete before spawning. Update the table when teammates complete.

   **Parallel dispatch type registry**: Before spawning a batch of `Parallel: true` builder teammates that pass the file-overlap guard, scan the codebase (or prior task outputs) for all public types, protocols, enums, and function signatures the parallel tasks will consume or extend. Compile into a type registry block and include in each builder's spawn prompt as a `## Shared Type Registry` section. Format: `- {TypeName} ({file path}): {one-line summary}`. This prevents parallel builders from creating conflicting type definitions. Skip if no shared types exist for the batch.

6. **Acceptance gate**: After all tasks complete, dispatch the plan's `final-acceptance` task as a **validator teammate** (not inline command execution). The validator receives the full final-acceptance task description from the plan, including the functional completeness sweep and cross-task integration check. The spawn prompt must include:
   - The plan's `## Acceptance Criteria` section (verbatim)
   - The plan's `## Validation Commands` section (verbatim)
   - The final-acceptance task's action items from `## Step by Step Tasks` (including functional completeness sweep and cross-task integration check)
   - Instruction: "Execute every validation command. For the functional completeness sweep, read every file produced by the build and check for stub patterns. For the cross-task integration check, trace at least one end-to-end user flow across task boundaries."
   After the validator reports via SendMessage: if PASS, proceed to report. If FAIL, present findings and ask the user whether to remediate or proceed. Do NOT offer merge/PR/discard options until the user acknowledges any unmet criteria.

7. **Document lifecycle**: After the acceptance gate passes (or the user acknowledges unmet criteria and chooses to proceed), manage the pipeline documents:
   - Read `## Project Directory` from the plan. If the value is a project path (e.g., `projects/cadence`):
     1. Create `{project-dir}/docs/` if it doesn't exist.
     2. Identify all technical documents referenced in the plan's `## Relevant Files` section (PRDs, Brand OS, feature specs — files matching `prd-*.md`, `brand-os-*.md`, `feature-*.md`, `srd-*.md`, `architecture-*.md`, `service-*.md`).
     3. For each document currently in a staging directory (`docs/prd/`, `docs/specs/`): move it to `{project-dir}/docs/` using `mv`. Skip documents that are already in the project directory.
     4. Archive the plan: `mv {plan-path} docs/archive/plan/`.
     5. Move the `build-status--{plan-slug}.md` to `{project-dir}/docs/` as an execution record.
   - If the project directory is `basecamp`:
     1. Archive the plan: `mv {plan-path} docs/archive/plan/`.
     2. Leave technical documents in their staging directories (BASECAMP is both the project and the docs root).
     3. Move the `build-status--{plan-slug}.md` to `docs/builds/`.
   - Report all file moves to the user in the report step.
   - **Worktree consideration**: If executing in a worktree, perform document moves in the worktree. They will be included in the merge.
   - **Team cleanup**: After document lifecycle completes, clean up the team:
     ```
     TeamDelete({ team_name: "exec-{plan-slug}" })
     ```

8. **Report**: Present results with: tasks completed, files changed, verification results, document moves performed, worktree location, team metadata (name, waves executed, total teammates spawned). Offer options: merge to main, push and create PR, keep branch, or discard.

## Agent Compatibility Notes

The agent teams model introduces behavioral differences from the sub-agent model:

- **Prompt-enforced validator restrictions**: Validator tool restrictions (no Write, Edit, NotebookEdit) are enforced via spawn prompt instructions, not `disallowedTools` frontmatter. Monitor validator SendMessage output for signs of file modification. If detected, flag as a gotcha and consider re-validating with a fresh validator.
- **Sonnet for teammates, Opus for lead**: All teammates run on Sonnet for cost efficiency. The lead (orchestrator) runs on Opus for complex coordination decisions. If a task requires Opus-level reasoning (security review, complex architectural decisions), note this in the plan and the lead handles the decision-making.
- **Builder exploration delays are normal**: Builders perform codebase exploration before writing code. This may add latency to initial SendMessage output. Do not interpret exploration-phase delays as hangs or timeouts.
- **Full MCP access for all teammates**: All teammates have access to the same MCP servers as the lead. Spawn prompts include MCP usage instructions where relevant (Xcode, Context7).
- **No resume for teammates**: Agent teams do not support the `resume` parameter. If a teammate needs to continue work, spawn a replacement with prior context in the prompt. See step 5g for the retry pattern.
- **Communication is via SendMessage**: Teammates' plain text output is NOT visible to the lead. All communication must go through `SendMessage`. Spawn prompts explicitly instruct teammates to use SendMessage for reporting.

## Stage Handoff

When this skill completes, produce a structured handoff for the next stage. This handoff is part of the Report output — include it in the chat summary.

**Schema:**
- **Output file:** `build-status--{plan-slug}.md` (final location per document lifecycle step)
- **Tasks completed:** {N}/{total} (list task IDs)
- **Tasks failed:** {N} (list task IDs with failure reasons)
- **Files changed:** List of all files created or modified during execution
- **Documents moved:** List of document moves performed (source → destination) during lifecycle step
- **Verification results:** Summary of final acceptance outcome (PASS/FAIL with details)
- **Team metadata:** Team name, waves executed, total teammates spawned, teammates per wave
- **Key decisions:** 3-5 bullet points of implementation decisions made during execution
- **Constraints established:** Any runtime constraints discovered (performance, compatibility)
- **Open questions:** Unresolved items from builder gap-flagging or validator findings
- **Scope:** Worktree branch and merge status

## Related Skills

- `plan-beta` — Upstream: produces implementation plans targeting the agent teams execution model
- `plan` — Also compatible: plans produced by the original `plan` skill can be executed with this skill (they lack spawn wave metadata, so all tasks default to sequential waves)
- `execute` — Original variant: executes plans using direct sub-agent dispatch

## Completion Token

When this skill completes successfully, output: `[SKILL_COMPLETE:execute-beta]`
