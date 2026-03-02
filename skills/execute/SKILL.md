---
name: execute
version: 1.2.0
description: "Use when you have an implementation plan ready for agent-driven execution"
argument-hint: [path to plan document]
model: opus
---

# Execute

- **Agents:** orchestrator-invoked

Read and execute the implementation plan at `PATH_TO_PLAN`.

## Variables

PATH_TO_PLAN: $ARGUMENTS

## Workflow

1. **Validate input**: If no `PATH_TO_PLAN` is provided, STOP and ask the user (AskUserQuestion). Reject paths containing `..` or pointing outside the current project directory.

2. **Read the plan**: Read the file at `PATH_TO_PLAN`. Think hard about it.

2.5. **Determine parallelism mode**:
   - Check if `PATH_TO_PLAN` arguments include `--parallelism serial|topology|aggressive`.
   - If not, check the plan's `## Project Topology` section for `Parallelism mode:`.
   - If neither exists, default to `serial`.
   - `serial`: all tasks sequential, ignore `Parallel: true` flags (safest, always correct).
   - `topology`: parallelize across different build units only (recommended for multi-target/multi-module/monorepo projects).
   - `aggressive`: also parallelize within build units for interpreted languages using file-set independence (for Python, TypeScript projects).

3. **Worktree isolation**:
   - **Teammate mode**: If TEAMMATE_MODE (spawn prompt sets this): skip worktree creation. Use WORKTREE_PATH from spawn prompt. All paths are absolute within WORKTREE_PATH.
   Check current branch with `git branch --show-current`.
   - If on `main` or `master`:
     - Follow `.claude/rules/worktree-conventions.md` for branch derivation, sanitization, creation, and dispatch path rules.
     - Derive branch name from plan file name (e.g., `docs/plan/add-auth.md` -> `feat/add-auth`).
     - Create worktree: `git worktree add .claude/worktrees/<branch-name> -b <branch-name>`
     - If it fails (branch exists, dirty state): ask user whether to proceed in current directory or abort.
     - All subsequent work uses absolute paths within the worktree.
   - If already on a feature branch: proceed in current directory.

4. **Create task list**: Before dispatching any work, populate the shared task list so all team members can see the full plan.
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

5. **Provisioning (Task 0)**: Read the plan's `## Provisioning` section. If it exists, proceed with provisioning. If it does not exist (older plans without provisioning), skip this step entirely — provisioning is optional for backward compatibility.
   - **Generate project CLAUDE.md**: Read the `### Project CLAUDE.md` subsection from the plan's Provisioning section. Read `## Project Directory` from the plan to determine {project-dir}. Write the content to `{project-dir}/CLAUDE.md`. If a CLAUDE.md already exists at that path, merge the new content: if the existing file already contains a `## Reference Documents` or `## Living-File Rules` section, append new entries to the existing section rather than creating a duplicate section header. Do not overwrite existing content outside the provisioning sections.
   - **Initialize decision log**: Read the `### Decision Log` subsection from the plan's Provisioning section. Write the template to `{project-dir}/docs/decision-log.md`. If the file already exists (from a prior execution), do not overwrite — log a note that an existing decision log was found and skip creation.
   - **Log provisioning**: Append a provisioning entry to `build-status--{plan-slug}.md` under a `### provisioning` heading using the build status entry format.
   - **Failure handling**: If provisioning artifacts cannot be written (permissions error, disk issue), log the failure in `build-status--{plan-slug}.md` and continue with execution. Provisioning failure should not block builder dispatch.

6. **Execute tasks**:
   - **Plan-scoped build status file**: Derive a plan-specific filename to allow concurrent executions of different plans in the same working directory. Take the plan filename from `PATH_TO_PLAN` (e.g., `docs/plan/plan-iterate-skill.md` → `plan-iterate-skill`), strip the path and `.md` extension, and use it as the slug: `build-status--{plan-slug}.md`. All subsequent references to the build status file in this execution use this derived name.
   - **Resume check**: Before dispatching any tasks, check for an existing `build-status--{plan-slug}.md` in the working directory.
     1. If found, parse completed task entries (status = PASS).
     2. For each completed task, verify its output artifacts exist on disk (check files listed in the task's action items).
     3. If artifacts exist: mark task as SKIPPED.
     4. If artifacts are missing: queue for re-execution.
     5. Present summary via AskUserQuestion: "Found {completed}/{total} tasks completed. Resume from task {next-task-id}?" with options:
        - "Resume (Recommended)" — skip verified tasks, continue from next incomplete
        - "Restart from scratch" — delete `build-status--{plan-slug}.md`, execute all tasks
   - If no `build-status--{plan-slug}.md` exists, proceed normally.
   - **Task status updates**: Throughout the dispatch loop, keep the shared task list in sync:
     - Before dispatching a task: `TaskUpdate({ taskId: "{id}", status: "in_progress" })`
     - After a task passes validation (or completes for non-cycling agents): `TaskUpdate({ taskId: "{id}", status: "completed" })`
     - After a task fails and is not retried: `TaskUpdate({ taskId: "{id}", status: "cancelled" })`
     - After a task is skipped (resume): `TaskUpdate({ taskId: "{id}", status: "completed" })`
     - Use `TaskList({})` to review overall progress at phase boundaries or when re-reading state after compaction.
     These updates are in ADDITION to the build status file entries — both tracking systems must stay in sync.
   - **Dispatch rules persistence**: At the start of execution, write a `## Dispatch Rules` header into `build-status--{plan-slug}.md` containing these critical rules:
     - Builder/validator pairs: dispatch builder, then validator. 3-strike retry on FAIL.
     - Topology-aware dispatch: read `## Project Topology` from plan. If present, use build unit analysis for parallelism. If absent, default to serial. Respect parallelism mode flag (`serial | topology | aggressive`).
     - Parallel dispatch: only when plan marks `Parallel: true` AND dependencies resolved AND file-overlap guard passes AND build unit does not overlap with any currently-running task's build unit. Use `run_in_background: true`.
     - Gap-flagging: scan builder output for escalation signals before dispatching validator.
     - Gotcha relay: scan all agent output for `## Gotchas Identified` section.
     - Type registry: build and include for parallel batches.
     - Final acceptance: dispatch as validator subagent, not inline.
     If `build-status--{plan-slug}.md` already exists and contains a `## Dispatch Rules` section (from a prior or resumed execution), overwrite it with the current rules to prevent stale rule drift.
     Before dispatching EACH task, re-read the `## Dispatch Rules` section of `build-status--{plan-slug}.md` to reload these rules. This makes the orchestrator resilient to context compaction.
   - Follow the plan's Team Orchestration section for agent dispatch, dependencies, and validation pairs.
   - **Running tasks table**: Maintain a list of currently-running tasks with their task ID, agent ID, and build unit. Before dispatching a new task, check whether its build unit overlaps with any running task's build unit. If overlap exists in a compiled language, wait for the conflicting task to complete before dispatching. Update the table when tasks complete.
   - **File-overlap guard**: Before dispatching a batch of `Parallel: true` builder tasks, extract the file paths each task will create or modify (from the plan's action items). If any file path appears in more than one task in the batch, those tasks CANNOT run in parallel — serialize them. Only dispatch tasks in parallel whose file sets are completely disjoint. This check is the final gate before parallel dispatch and overrides `Parallel: true` if overlap is detected. Log any overrides in `build-status--{plan-slug}.md`: "File overlap detected: {file} appears in tasks {task-a} and {task-b}. Serializing."
   - **Parallel collection loop**: When tasks are running in background (`run_in_background: true`), collect results using this pattern:
     1. Poll running tasks with `TaskOutput({ block: false, timeout: 5000 })` to check for completions.
     2. When a task completes: process its result (build gate, gap-flagging, gotcha relay, status update), remove from running tasks table, check if any blocked tasks are now dispatchable, dispatch them.
     3. If no tasks completed: block on the oldest running task with `TaskOutput({ block: true, timeout: 60000 })` to avoid busy-waiting.
     4. Repeat until all tasks in the current batch are complete.
     This replaces the implicit "dispatch one, wait, dispatch next" pattern with true concurrent execution.
   - **Parallel dispatch type registry**: Before dispatching a batch of `Parallel: true` builder tasks that pass the file-overlap guard, scan the codebase (or prior task outputs) for all public types, protocols, enums, and function signatures the parallel tasks will consume or extend. Compile into a type registry block and include in each builder's dispatch prompt as a `## Shared Type Registry` section. Format: `- {TypeName} ({file path}): {one-line summary}`. This prevents parallel builders from creating conflicting type definitions. Skip if no shared types exist for the batch.
   - **Pre-validator build gate**: After a builder completes but before dispatching its paired validator, the orchestrator MUST independently run a full-target build (not `-only-testing`, not a module-only build). Do not trust the builder's build claim. For Xcode projects, use `xcodebuild build -scheme {scheme} -derivedDataPath ./DerivedData 2>&1 | xcsift` via Bash; for other project types, use the project's standard build command. Do NOT use `mcp__xcode__BuildProject` (hangs from Claude Code CLI due to TCC — see `.claude/rules/xcode-mcp.md`).
     - If the build passes: dispatch the validator.
     - If the build fails on files the builder touched: re-dispatch the builder to fix (counts toward 3-strike limit).
     - If the build fails on files NOT touched by this builder (cross-builder contamination): identify the responsible builder/task, re-dispatch that builder to fix, then retry the build gate. Do not dispatch the validator until the build passes.
     This gate prevents the entire class of "builder claims success but project doesn't compile" failures.
   - **Builder dispatch prompt requirements**: Every builder dispatch prompt MUST include these constraints (append to the task-specific prompt):
     1. "If you create new source files in an Xcode project, use `ToolSearch` to load and use `mcp__xcode__XcodeWrite` (which auto-registers files in project.pbxproj). Do NOT use the generic `Write` tool for Xcode source files, as it does not register them — unregistered files are invisible to the compiler."
     2. "For Xcode project builds and tests, use `xcodebuild` via Bash piped through `xcsift` for structured output (e.g., `xcodebuild build -scheme MyScheme -derivedDataPath ./DerivedData 2>&1 | xcsift`). Do NOT use `mcp__xcode__BuildProject` or `mcp__xcode__RunSomeTests` — they hang from Claude Code CLI due to TCC permissions. See `.claude/rules/xcode-mcp.md` for the full hybrid strategy. You MAY use Xcode MCP read-only tools (`DocumentationSearch`, `XcodeRefreshCodeIssuesInFile`, `XcodeListNavigatorIssues`) for inspection and diagnostics."
     3. "Write bottom-up: define types before referencing them. If a type from a dependency task doesn't exist on disk yet, do not reference it — write a compilation-safe stub or defer to the dependency."
     4. If the task includes a `**Context Files**` field, prepend to the dispatch prompt: "Before starting, read the following reference document(s) and apply their decisions to your implementation: {context file paths}. For Brand OS documents: apply visual identity, voice/tone, motion, interaction patterns, and micro-copy guidelines specified there. For PRD documents: follow the functional requirements, behavioral specs (Given-When-Then), non-functional requirements, and technical constraints specified there. When both are loaded: the PRD defines WHAT to build and the Brand OS defines HOW it should look, sound, and feel."
     5. "Before starting, load domain-specific skills per `.claude/rules/subagent-skill-dispatch.md`. Scan your task description for domain signals (CI failure, security, performance, etc.) and invoke matching skills. Cap at 3 skills per task."
     6. If the task has a `Context Specs` field, treat it as equivalent to `Context Files` — read and apply the referenced spec documents before implementing.
     These constraints are placed in the dispatch prompt (not rules files) to survive context compaction and ensure every builder receives them regardless of loaded context.
   - **Builder/validator pairs**: After each completed pair, append a progress entry to `build-status--{plan-slug}.md` in the working directory using the build status entry format below. If a validator reports FAIL: re-read the failure details, re-dispatch the builder to fix, then re-dispatch the validator. If a task fails validation 3 times, stop and ask the user.
     - **Teammate mode**: If TEAMMATE_MODE: instead of AskUserQuestion, send via SendMessage:
       `SendMessage({ type: "message", recipient: "lead", content: "ESCALATION: {plan-slug}/{task-id} | strike 3/3 | {root cause}", summary: "3-strike failure {task-id}" })`
       Wait for lead's SendMessage response with instructions.
   - **Build status entry format**: Each task entry in `build-status--{plan-slug}.md` uses this format:
     ```
     ### {Task ID}
     - **Status**: PASS | FAIL | ABORTED | SKIPPED
     - **Agent**: {agent name}
     - **Started**: {ISO timestamp}
     - **Completed**: {ISO timestamp}
     - **Retry Count**: {N}
     - **Notes**: {failure reasons, validator findings, or other details}
     ```
     The resume check (above) parses this format. Always write entries in this format so future sessions can parse them reliably. For backward compatibility, the resume parser should also handle the legacy inline format (`task-id | STATUS | timestamp`) from older build-status files — extract task ID and status, treat other fields as unknown.
   - **Builder gap-flagging**: After a builder completes but before dispatching its paired validator, scan output for escalation signals: ambiguity, unclear, missing information, contradiction, not specified, underspecified, assumed. If found, extract the relevant sentences and present via AskUserQuestion: "Builder flagged potential gaps in task {task-id}:\n{extracted sentences}" with options:
     1. "Clarify and re-dispatch builder" — provide clarification, re-dispatch with clarification appended to prompt
     2. "Proceed with builder's interpretation" — dispatch validator as normal
     3. "Abort this task" — skip task and validator, note in `build-status--{plan-slug}.md` as ABORTED
     - **Teammate mode**: If TEAMMATE_MODE: instead of AskUserQuestion, send via SendMessage:
       `SendMessage({ type: "message", recipient: "lead", content: "QUESTION: {plan-slug}/{task-id}: {extracted sentences}", summary: "Gap-flag from {task-id}" })`
       Then continue to next task. Do NOT block — the lead will route the answer back via SendMessage, at which point re-dispatch the builder with the clarification.
   - **Decision log entry**: After gap-flagging resolution (either "Clarify and re-dispatch builder" or "Proceed with builder's interpretation"), append a decision log entry to `{project-dir}/docs/decision-log.md`:
     ```
     ## {YYYY-MM-DD} — {task-id}: {brief decision summary}
     **Context:** {extracted gap-flag sentences}
     **Decision:** {user's choice — clarification provided or "proceeded with builder interpretation"}
     **Impact:** {which files/features are affected}

     ---
     ```
     If the decision log file does not exist (provisioning was skipped or plan has no Provisioning section), skip the entry silently.
   - **Non-cycling agents** (researcher, critic, designer, auditor): These run once and return results. No validator pair. Dispatch, collect output, append to `build-status--{plan-slug}.md`, and proceed to the next task.
     - **Researcher**: Dispatch with research question. Output is a document written to docs/. Proceed when document exists.
     - **Critic**: Dispatch with artifact to review. Output is a PASS/REVISE/DROP verdict. On REVISE: feed changes back to the builder. On DROP: stop and ask the user.
     - **Designer**: Dispatch with design question. Output is a design document or ADR. Proceed when document exists.
     - **Auditor**: Dispatch with skill name in the prompt (e.g., "Invoke Skill('review-security') then audit src/auth/"). If the plan specifies review skills, include them in the prompt. Output is a findings report.
   - **Project status update**: After each builder/validator pair passes (or after a non-cycling agent completes), update the project's status file. Locate the file: check for `STATUS.md` or `status.md` in `{project-dir}` (case-insensitive search; prefer the one that exists). If found:
     1. Identify which milestone unit(s) the completed task corresponds to (from the plan's `## Units` or task metadata).
     2. Mark those units as `Complete` in the status file's unit tables.
     3. If all units in a milestone are now complete, update the milestone status to `Complete` in the phase overview table.
     4. Update the `Last updated` date.
     5. Add an activity log entry if the status file has a `## Recent Activity` section.
     If no status file exists, skip silently — the project may not use forge conventions. The build-status file still tracks per-task execution detail for resume capability.
   - **Gotcha relay**: After any agent completes (builder, validator, critic, designer, auditor, researcher), scan output for a `## Gotchas Identified` section. If found, extract entries. Read `docs/logs/gotcha-log.md` and check for duplicates (same title or substantially same description). For each new gotcha, append to the top of the log (below header) using the agent entry format:
     ```
     ## [{tag}] {Title}
     **Date:** {YYYY-MM-DD} | **Last Seen:** {YYYY-MM-DD} | **Occurrences:** 1
     **Source:** agent, {agent-name}

     {Description in 2-4 sentences.}

     ---
     ```
     If a duplicate is found, increment its Occurrences count and update Last Seen date.

7. **Acceptance gate**: After all tasks complete, dispatch the plan's `final-acceptance` task as a **validator subagent** (not inline command execution). The validator receives the full final-acceptance task description from the plan, including the functional completeness sweep and cross-task integration check. The dispatch prompt must include:
   - The plan's `## Acceptance Criteria` section (verbatim)
   - The plan's `## Validation Commands` section (verbatim)
   - The final-acceptance task's action items from `## Step by Step Tasks` (including functional completeness sweep and cross-task integration check)
   - Instruction: "Execute every validation command. For the functional completeness sweep, read every file produced by the build and check for stub patterns. For the cross-task integration check, trace at least one end-to-end user flow across task boundaries."
   After the validator returns: if PASS, proceed to report. If FAIL, present findings and ask the user whether to remediate or proceed. Do NOT offer merge/PR/discard options until the user acknowledges any unmet criteria.

8. **Document lifecycle**:
   - **Teammate mode**: If TEAMMATE_MODE: skip this step entirely. The lead handles document lifecycle.
   After the acceptance gate passes (or the user acknowledges unmet criteria and chooses to proceed), manage the pipeline documents:
   - **Final status file update**: Read the project's status file (`STATUS.md` or `status.md` in `{project-dir}`). Update the current phase/milestone header to reflect what was just completed. Mark any remaining incomplete units from this plan as complete. Update `Last updated` date. Add a summary activity log entry (e.g., "M5 completed — session deck (PR #12)"). This is the permanent record of execution — it replaces the build-status file as the committed artifact.
   - **Build-status cleanup**: The `build-status--{plan-slug}.md` is a **transient working document** used only during execution for resume capability and dispatch rules persistence. Do NOT commit it. Either delete it (`rm build-status--{plan-slug}.md`) or leave it untracked. If executing in a worktree, ensure it is in `.gitignore` or deleted before commit.
   - Read `## Project Directory` from the plan. If the value is a project path (e.g., `projects/cadence`):
     1. Create `{project-dir}/docs/` if it doesn't exist.
     2. Identify all technical documents referenced in the plan's `## Relevant Files` section (PRDs, Brand OS, feature specs — files matching `prd-*.md`, `brand-os-*.md`, `feature-*.md`, `srd-*.md`, `architecture-*.md`, `service-*.md`).
     3. For each document currently in a staging directory (`docs/prd/`, `docs/specs/`): move it to `{project-dir}/docs/` using `mv`. Skip documents that are already in the project directory.
     4. Archive the plan: `mv {plan-path} docs/archive/plan/`.
   - If the project directory is `basecamp`:
     1. Archive the plan: `mv {plan-path} docs/archive/plan/`.
     2. Leave technical documents in their staging directories (BASECAMP is both the project and the docs root).
   - Report all file moves to the user in the report step.
   - **Worktree consideration**: If executing in a worktree, perform document moves in the worktree. They will be included in the merge.

9. **Report and ship**:
   - **Teammate mode**: If TEAMMATE_MODE: instead of the PR/keep/discard flow, send completion report:
     `SendMessage({ type: "message", recipient: "lead", content: "PLAN_COMPLETE: {plan-slug} | {N}/{M} tasks PASS | merge ready", summary: "Plan {plan-slug} complete" })`
     Then stop. Do not create PRs or merge branches.
   Present results with: tasks completed, files changed, verification results, document moves performed, worktree location.
   - Use AskUserQuestion to offer shipping options:
     1. "Create PR (Recommended)" — commit all changes on the feature branch, push to origin, create a PR targeting main, and clean up the worktree. This is the standard workflow for reviewed changes.
     2. "Keep branch" — leave the worktree and branch as-is for further work.
     3. "Discard" — remove the worktree and delete the branch.
   - **Executing the selected option**: Invoke `Skill('github')` to handle whichever option the user selects. The GitHub skill has the commit conventions, PR templates, worktree cleanup procedures, and MCP tool preferences needed to execute cleanly. Pass it the operation context: branch name, files changed summary, and the user's choice.
   - For "Create PR": the GitHub skill handles commit → push → PR creation (with proper description from the stage handoff) → worktree cleanup.
   - For "Discard": `git worktree remove` + `git branch -d` per worktree conventions.

## Agent Compatibility Notes

The upgraded agent definitions (v2.0) operate within the existing build dispatch pattern without structural changes:

- **Builder exploration step**: Builders now perform a codebase exploration step before writing code. This may add latency to initial output. Do not interpret exploration-phase delays as hangs or timeouts — the builder is reading related files to understand context before implementing.
- **Builder debugging micro-cycle**: If a builder escalates on third failure (per its debugging micro-cycle), treat this as a task failure — same as current behavior when a builder reports inability to complete.
- **Validator structured protocol**: Validators now follow a structured verification protocol. Existing retry/escalation logic (3-strike rule) remains unchanged.
- **Agent gotcha reporting**: Critic and validator agents may include a `## Gotchas Identified` section in their output. These are extracted and logged by the gotcha relay (step 6).

## Stage Handoff

When this skill completes, produce a structured handoff for the next stage. This handoff is part of the Report output — include it in the chat summary.

**Schema:**
- **Output file:** `{project-dir}/STATUS.md` or `{project-dir}/status.md` (the project's canonical status file, updated during document lifecycle)
- **Tasks completed:** {N}/{total} (list task IDs)
- **Tasks failed:** {N} (list task IDs with failure reasons)
- **Files changed:** List of all files created or modified during execution
- **Documents moved:** List of document moves performed (source → destination) during lifecycle step
- **Verification results:** Summary of final acceptance outcome (PASS/FAIL with details)
- **Key decisions:** 3-5 bullet points of implementation decisions made during execution
- **Constraints established:** Any runtime constraints discovered (performance, compatibility)
- **Open questions:** Unresolved items from builder gap-flagging or validator findings
- **Scope:** Worktree branch and merge status

## Related Skills

- `plan` — Upstream: produces the implementation plan this skill executes
- `github` — Downstream: handles PR creation, branch push, worktree cleanup after execution completes

