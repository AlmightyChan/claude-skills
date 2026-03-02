---
name: forge-plan
version: 1.0.0
description: "Transform a completed forge project into parallelizable implementation plans with worktree isolation"
argument-hint: "[project-name]"
model: opus
disallowed-tools: [EnterPlanMode]
---

# Forge Plan

- **Agents:** orchestrator-invoked

Transform a completed forge project into execution-ready implementation plans. Analyze forge output, group components into dependency cohorts, and produce parallelizable plans structured for worktree-isolated agent dispatch. Each plan is self-contained — the executing agent reads only that plan. Follow the `Instructions` and work through the `Workflow`.

## Variables

PROJECT_NAME: $ARGUMENTS
FORGE_PROJECT_DIRECTORY: `projects/$ARGUMENTS/`
PLAN_OUTPUT_DIRECTORY: `projects/$ARGUMENTS/plans/`
TEAM_MEMBERS: `/Users/ethanmuna/.claude/agents/*.md`
AGENT_ROSTER: builder, critic, validator, researcher, designer, auditor, writer
WORKTREE_CAP: 3

## Instructions

- **PLANNING ONLY**: Do NOT build, write code, or deploy agents for implementation. Your only output is plan documents saved to `PLAN_OUTPUT_DIRECTORY`.
- This skill runs as the orchestrator and writes all plan files directly. It does NOT dispatch writer or builder subagents for plan generation. The only subagents dispatched are critic and designer during the optional Quality Gate (step 10).
- Create `projects/$ARGUMENTS/plans/` directory if it does not exist.
- If no `PROJECT_NAME` is provided, stop and ask the user to provide it.
- Generate descriptive, kebab-case filenames based on the component grouping.
- Save plans to `PLAN_OUTPUT_DIRECTORY/<plan-slug>.md`.
- Apply the Plan Format for each plan.

### Task Specificity Requirements

Each task's action bullets MUST include:
- **File paths**: Exact files to create or modify (e.g., `src/auth/login.ts`, not "the auth module")
- **Verification command**: How to confirm the task succeeded (e.g., `pytest tests/test_auth.py`, `swift build`, `python -c "from module import func"`)
- **Expected outcome**: What the verification command should produce (e.g., "all tests pass", "builds with no errors", "prints 'Hello World'")
- **Compiled reality constraint**: Task descriptions and action items MUST NOT reference types, methods, properties, or enum cases that will be created by other tasks unless those tasks are listed in `Depends On`. If Task B references `PlanRepository.getPendingSuggestions()` and `PlanRepository` is created by Task A, then Task B MUST have `Depends On: Task A`. If the type doesn't exist yet and no dependency creates it, the task must define it itself. Never write task descriptions against planned API shapes from the spec — only reference what will exist at execution time.

### Per-Task Validation Cycling

For every builder task, create a corresponding validator task that depends on it. Do NOT place a single validator at the end. The validator task should reference the same files and verification commands as the builder task.

**Cost gate**: Set `validate: false` for tasks that only create directories, write config files, or perform setup that has no behavioral requirements to verify.

**Functional completeness requirement**: Every validator task MUST include a `**Functional completeness**:` field with 2-4 task-specific assertions. These assertions must name the specific functions, handlers, or components being validated and describe what REAL behavior looks like vs. stubs. Generic assertions like "verify it works correctly" are not acceptable. Derive assertions from the builder task's action items and the source requirements.
Tests that only verify constant values (e.g., `CadenceDesign.dotDiameter == 64`) without testing runtime behavior do NOT satisfy functional completeness.

**Parallel reconciliation checkpoints**: After every batch of `Parallel: true` builder tasks that produce types, protocols, or components consumed by later tasks, insert a reconciliation checkpoint task. This is a validator task that reads all files from the parallel batch, builds a type registry, and verifies cross-task compatibility (no duplicate types, no conflicting signatures, imports resolve). Note: The orchestrator's pre-dispatch type registry (in the execute skill) is a preventive input that reduces the chance of conflicts; the reconciliation checkpoint is a post-completion verification that detects conflicts that occurred despite the registry. Both are needed — prevention alone is insufficient when builders generate new types.

**Build unit constraint**: Before marking builder tasks as `Parallel: true`, consult the plan's `## Project Topology` section. A "build unit" is the smallest set of source files that must be built together — an Xcode target, a Go module, a Rust crate, an SPM module, or (for interpreted languages) a package directory.

Two tasks writing to the **same** build unit in a compiled language (Swift, Go, Rust, C/C++) cannot safely parallelize — the compiler processes all source files in the unit together, and concurrent writes create cross-compilation conflicts. For interpreted languages (Python, TypeScript, Ruby), tasks in the same build unit can parallelize if their file sets do not overlap. Tasks in **different** build units can always parallelize.

When to mark `Parallel: true`:
- Tasks write to **different** build units (e.g., app target vs. widget target, separate SPM modules)
- Tasks write to **different** projects in a monorepo (e.g., backend vs. frontend)
- The language has **no compilation step** (Python, TypeScript, Ruby) AND tasks touch different files
- Tasks are non-code (documentation, config, CI/CD) with no file overlap

When to mark `Parallel: false`:
- Tasks write to the **same** build unit in a compiled language
- Tasks touch **overlapping files** (even in interpreted languages)
- One task creates a type/module that another task imports

Assign a `Build Unit` field to each task matching the build unit path from `## Project Topology`. If the project topology shows a single build unit with `compilation_model: compiled`, mark all builder tasks as `Parallel: false`.

### Functional Verification Requirements

Verification commands must test **user-observable behavior**, not just code existence or compilation.

Rules:
- Every task with `Validate: true` that creates UI or endpoints MUST include a `Behavioral` field
- Verification commands MUST be executable without manual interaction (no "open browser and click")
- If a behavioral test requires a running server, the Verify command must handle startup and teardown (e.g., `npx start-server-and-test 'npm run dev' http://localhost:3000 'npx playwright test'`)
- The `Behavioral` field must be 1-2 sentences describing user action → observable outcome. Do not write test code in this field.

Examples:

BAD (validates existence, not behavior):
- **Verify**: `grep -r "onClick" src/components/DeleteButton.tsx`
- **Expected**: onClick handler exists

GOOD — UI task:
- **Verify**: `npx playwright test tests/e2e/delete-flow.spec.ts`
- **Expected**: Delete removes item from list, list count decrements
- **Behavioral**: User clicks delete → confirmation dialog appears → confirms → item removed from list

GOOD — API task:
- **Verify**: `pytest tests/test_auth.py -v`
- **Expected**: "5 passed" including auth, token, and error-handling tests
- **Behavioral**: POST /login with valid creds returns 200+token; invalid creds returns 401+error message

See calibration example in `supporting-files/plan-template.md`.

### Team Orchestration

The executor operates as team lead and never writes code directly. Deploy team members via `Task` tool with `subagent_type`. Track work with `TaskCreate`, `TaskUpdate`, `TaskList`, `TaskGet`. Set dependencies with `addBlockedBy`. Use `run_in_background: true` for parallel execution. Resume agents with the `resume` parameter for related follow-up work.

IMPORTANT: This section is embedded in every output plan for the downstream executor. Forge-plan itself does not dispatch builders/validators.

#### Orchestration Workflow
1. Create all tasks with `TaskCreate`
2. Set dependencies with `TaskUpdate` + `addBlockedBy`
3. Assign owners with `TaskUpdate` + `owner`
4. Deploy agents with `Task`
5. Monitor with `TaskList` and `TaskOutput`
6. Resume agents with `Task` + `resume` for follow-up
7. Mark complete with `TaskUpdate` + `status: "completed"`

## Workflow

IMPORTANT: **PLANNING ONLY** — Do not execute, build, or deploy. Output is plan documents.

### 1. Validate Input
- If no PROJECT_NAME provided, stop and ask.
- Validate `projects/$ARGUMENTS/` exists.
- Check forge completion: `status.md` must exist. If not: "Forge project incomplete. Run `/forge {name}` to finish."
- Check for unresolved CRITICAL decisions in `decisions.md`. Present to user if any exist.

### 2. Ingest Forge Artifacts
- Read: `discovery-notes.md`, `product-brief.md`, `components.md`, `spec-manifest.md`, `decisions.md`, `issues.md`, `review-findings.md`, `roadmap.md`, `status.md`, `CLAUDE.md`
- Read all spec files listed in `spec-manifest.md` (list `specs/` directory first, cross-reference against manifest to flag missing files)
- Cross-check: every component in `components.md` should have at least one spec in `spec-manifest.md`. Flag any orphaned components.
- Present project summary: name, description, component count, spec count, version scope

### 3. Analyze Dependencies
- Build component dependency graph from spec-manifest dependencies
- Detect foundation candidates (components depended on by 2+)
- Detect circular dependencies (merge or flag)
- Determine project topology using the detection logic defined below. Record the topology once; it will be embedded in every generated plan.

  Detect the project type by checking for project marker files:
  1. `.xcodeproj`/`.xcworkspace` → parse `project.pbxproj` for target count → `xcode-single` or `xcode-multi`
  2. `Package.swift` → parse for `.target()` entries → `spm-single` or `spm-multi`
  3. `Cargo.toml` → check for `[workspace]` → `rust-single` or `rust-workspace`
  4. `go.mod` → `go-module`
  5. `package.json` → check for `workspaces` → `node-single` or `node-monorepo`
  6. `pyproject.toml`/`setup.py`/`requirements.txt` → `python`
  7. Multiple project roots (e.g., `backend/` + `frontend/`) → `full-stack`
  8. Fallback → `unknown` (defaults to serial)

  Parallelism mode recommendations:
  - `xcode-single`, `spm-single`, `rust-single`, `unknown` → `serial`
  - `xcode-multi`, `spm-multi`, `rust-workspace`, `node-monorepo`, `full-stack` → `topology`
  - `python`, `node-single` with independent packages → `topology` or `aggressive`

- **Project Characteristic Detection**: Detect technology signals to inform review task generation in component plans:

  | Signal | Detection Method | Review Skill |
  |--------|-----------------|--------------|
  | Database layer | models/, migrations/, schema files, ORM config | review-database* |
  | API layer | routes/, controllers/, handlers/, OpenAPI specs | review-api* |
  | Frontend | components/, state management, React/Vue/Angular markers | review-frontend* |
  | Backend services | services/, domain logic, queue/worker patterns | review-backend* |
  | CI/CD | .github/workflows/, Dockerfile, CI config | review-devops* |
  | Large codebase | 50+ source files | review-architecture |
  | Any code changes | Always | review-code-quality |

  *Tier 2 skills — generate tasks only when the skill exists on disk (check with `ls ~/.claude/skills/{name}/SKILL.md`).

  Record detected characteristics in each plan's `## Project Topology` section. When generating review/auditor tasks in component plans, add appropriate specialist review tasks based on detected signals.

### 4. Group Components into Plans
- **Single-component short-circuit**: If the project has only one component (or all components form a single dependency-connected group with ≤200 estimated units), skip grouping. Produce one component plan and one manifest. Skip the integration plan — the component plan's Final Acceptance task covers what integration would verify. Note in the manifest: "Single-plan project — no integration plan needed."
- Run dependency-cohort grouping (see Grouping Algorithm)
- Estimate context budget per group
- Assign parallelism tiers (capped at 3 concurrent per tier)
- **APPROVAL GATE**: Present grouping to user via AskUserQuestion:
  - Plan count, which components in which plans
  - Dependency DAG, parallelism tiers
  - Foundation plan identification
  - Estimated units per plan
  - Options: Confirm / Request Changes / Abort
- If Request Changes: adjust grouping per user feedback, re-present. Loop until Confirm.
- If Abort: stop skill execution.
- Save approved grouping to `plans/grouping.json` after step 4.

### 5. Generate Component Plans
a. For each plan group: read `supporting-files/plan-template.md`, then apply it to produce a complete plan
b. Each plan is self-contained — the executing agent reads only that plan
c. Include worktree metadata (branch name: `feat/{plan-slug}`, base: main)
   Populate the Worktree section's `Blocks Plans` and `Depends On Plans` fields from the dependency DAG produced in step 4.
d. Assign Skills per task using the Skills Field rules (see Plan Format section)
e. Assign Context Specs per task (see Plan Format section)
f. Context Budget Check (distinct from the Grouping Algorithm's sizing — this phases WITHIN a single plan; the algorithm sizes plan GROUPS): if a plan exceeds ~200 operational units, split into sequential phases. Phase boundary rules:
   - Hard: keep builder/validator pairs in the same phase
   - Hard: never split a dependency chain across phases
   - Soft: research/setup tasks first, core implementation second
   - Heuristic: builder ~15-25 units, validator ~8-12 units, read/research ~1-5 units
   - If phased: add checkpoint blocks between phases:
     ```markdown
     ## Checkpoint: Phase {N} Complete

     **Plan file**: {absolute path to plan document}
     **Completed phases**: {list of completed phase numbers with one-line summary each}
     **Next phase**: {phase number and name}
     **Files produced so far**: {list of files created or modified in prior phases}
     **Resume instruction**: Reload the plan at {path}, skip to Phase {N+1}, and begin with task {first task ID in next phase}. All files listed above are on disk and should not be recreated.
     ```
g. Self-validate each plan: read it back after writing and confirm all required sections exist: Task Description, Objective, Project Directory, Worktree, Relevant Files, Interface Contract, Project Topology, Team Orchestration, Team Members, Step by Step Tasks, Acceptance Criteria, Validation Commands, Merge Readiness Checklist, Notes. If missing, fix inline before continuing.
h. Save to `projects/$ARGUMENTS/plans/{plan-slug}.md`

### 6. Generate Integration Plan
- Re-read all component plans and the grouping output to ensure full context before generating
- Read `supporting-files/integration-template.md`, then apply it
- Save to `projects/$ARGUMENTS/plans/integration.md`

### 7. Generate Manifest
- Read `supporting-files/manifest-template.md`, then apply it
- Save to `projects/$ARGUMENTS/plans/manifest.md`

### 8. Cross-Check Requirement Coverage
- For each spec in `spec-manifest.md`, verify at least one task across all component plans references it in `Context Specs`.
- For each task across all plans, confirm it traces to at least one spec. Flag untraceable tasks as potential scope creep.
- Document the mapping briefly in the manifest's Notes or in each plan's Acceptance Criteria section.
- If any spec has no corresponding task, add a task to the appropriate component plan.

### 9. Report
- Follow the Report section

### 10. Quality Gate (optional)
- Batched: critic + designer review ALL plans at once
- Up to 5 revision cycles, multi-plan scope. See Phase: Quality Gate section below.

## Grouping Algorithm

The grouping algorithm determines how forge components are partitioned into right-sized plans. The goal: each plan fits within ~200 operational units, maximizes parallelism, and respects dependency ordering.

### Inputs
- `components.md` — component names and descriptions
- `spec-manifest.md` — specs with dependency fields mapping to components
- `roadmap.md` — version scoping (default: v1 MVP scope)

### Algorithm

1. **Build component dependency graph.**
   For each spec in `spec-manifest.md`, record its `Component` field (maps spec → component).
   For each spec's `Dependencies` list, look up the dependency spec's component.
   If the dependency's component differs from the current spec's component,
   add a directed edge: current component → dependency component.

2. **Detect and resolve circular dependencies.**
   If any set of components forms a dependency cycle (direct or transitive),
   merge all components in the cycle into one group.

3. **Detect foundation components.**
   A component is a foundation candidate if 2+ other components depend on it.
   Foundation components form Plan Group 0 and execute before all others.
   Foundation-first is a hard constraint, not optional.

4. **Topological sort.**
   Sort remaining components into dependency layers.
   Layer 1 = depends only on foundation. Layer 2 = depends on layer 1. Etc.

5. **Merge layers into plan groups.**
   Starting from the deepest layer (leaves), merge layer N with layer N-1
   (moving toward root) until each group reaches 100-200 estimated operational units.
   Floor: if a group has fewer than 50 units, merge with an adjacent group in the same tier.
   Never split a dependency chain across groups.
   Overflow guard: if merging layer N into layer N-1 would exceed 200 units, keep them as separate groups.
   Apply the 50-unit floor rule only within the same tier (among groups at the same dependency depth).
   Independent subgraphs within a layer become separate plan files.

6. **Cap parallelism tiers.**
   Plans at the same dependency depth form a parallelism tier.
   If a tier has more than WORKTREE_CAP plans, partition into sequential batches of WORKTREE_CAP.
   (System-wide worktree cap is WORKTREE_CAP + 1 including main.)

7. **Compute critical path.**
   Longest sequential chain through the plan dependency graph.

### Cost Estimation Heuristic

Unit costs: builder ~15-25, validator ~8-12.
- Count specs in group × 20 units (midpoint for builder task per spec)
- Add validator tasks: spec count × 10 units (midpoint)
- Add 10 units for foundation/setup tasks per group
- Result = estimated operational units for the plan
- Note: complex specs may require 2-3 builder tasks; adjust upward if spec scope is large

### Output

For each plan group: component list, spec list, dependency ordering, parallelism tier, estimated units, worktree branch name.

## Plan Format

- IMPORTANT: Replace `<requested content>` with actual content. It's templated for you to replace.
- IMPORTANT: Anything NOT in `<requested content>` should be written EXACTLY as it appears.
- IMPORTANT: The section headings below MUST match exactly for self-validation in step 5g.

### Skills Field

Valid values (derived from subagent skill dispatch rules):

| Signal | Skill | Agent |
|--------|-------|-------|
| New feature, behavior change, bugfix | `tdd` | builder |
| Bug, test failure, unexpected behavior | `systematic-debugging` | builder |
| Claiming done, final check | `verification-before-completion` | builder, validator |
| CI failure, pipeline error | `ci-fix` | builder |
| Security audit, auth, injection | `review-security` | auditor |
| Code review, quality check | `review-code-quality` | auditor |

Default assignments:
- Builder tasks: `[tdd, verification-before-completion]`
- Validator tasks: `[verification-before-completion]`

### Context Specs Field

Every task MUST include a `Context Specs` field listing the forge spec files the dispatched agent reads before working. This is how agents stay aligned with the product specs.

**Always include:**
- `product-brief.md` — every task
- `decisions.md` — every task (agent filters to relevant entries)

**Route by task type:**

| Task Signal | Include These Specs |
|------------|-------------------|
| UI layout, views, screens, navigation | Feature spec + UI/brand spec (e.g., `specs/ux/brand-os.md`, `specs/ux/design-system.md`) |
| User-facing copy, voice, tone | Feature spec + brand/voice spec |
| API endpoints, data models, business logic | Feature spec + data/architecture spec |
| Auth, permissions, security | Feature spec + auth spec + security spec |
| State management, data flow | Feature spec + architecture spec |
| Integration, wiring, DI | All specs touched by components being wired |

**Routing rules:**
- Map each task's action items to the spec(s) that define those requirements
- For UI tasks: ALWAYS include both the functional spec (what to build) AND the brand/design spec (how it should look and behave)
- For cross-component tasks: include specs from all components involved
- Scope to 2-4 specs per task. If a task needs more, it may be too broad — consider splitting.

### Plan Template

See `supporting-files/plan-template.md`. Read it before generating each component plan.

## Manifest Format

See `supporting-files/manifest-template.md`. Read it before generating the manifest.

## Integration Plan Format

See `supporting-files/integration-template.md`. Read it before generating the integration plan.

## Session Resumability

State detection (check in order, first match wins):
1. No `plans/` directory → Start at step 1
2. `plans/` exists with `grouping.json` but no component plan `.md` files → Grouping approved, resume at step 5
3. `plans/` has some component plan `.md` files but not all listed in `grouping.json` → Verify existing plan files pass 5g self-validation before considering them complete. If a plan file exists but fails validation, treat it as incomplete and regenerate it. Resume step 5 for remaining and invalid plans.
4. All component plans exist, no `integration.md` → Resume at step 6
5. `integration.md` exists, no `manifest.md` → Resume at step 7
6. `manifest.md` exists → Plans complete. Display summary and offer Quality Gate.

### grouping.json Schema

```json
{
  "project": "<project-name>",
  "groups": [
    {
      "slug": "<plan-slug>",
      "name": "<human-readable plan name>",
      "components": ["<component-1>", "<component-2>"],
      "specs": ["specs/<domain>/<filename>.md"],
      "tier": 0,
      "estimated_units": 150,
      "branch": "feat/<plan-slug>",
      "is_foundation": true
    }
  ],
  "tiers": {
    "0": ["<plan-slug>"],
    "1": ["<plan-slug-a>", "<plan-slug-b>"]
  },
  "critical_path_hops": 3,
  "total_estimated_units": 450
}
```

On resume: present project name, resume point, and existing artifacts.

## Error Protocol

If a step fails, retry once. If it fails again, stop and report: step number, what failed, and suggested resolution. Do not loop.
If a subagent fails or times out, log "{agent} unavailable" in the plan's Notes section and continue with results from agents that completed. Do not block the pipeline on a single failed agent.

## Report

Present after all plans are generated:

```
Plans generated for {PROJECT_NAME}:

| Plan | Components | Tasks | Est. Units | Tier |
|------|-----------|-------|-----------|------|
| {slug} | {list} | {N}B + {N}V | ~{N} | {N} |

Integration: {N} tasks, ~{N} units
Total: {M} component plans + 1 integration plan
Critical path: {N} sequential tiers, up to {N} concurrent per tier
Grand total: ~{N} operational units

Next step: Run `/forge-execute {project-name}` to begin execution.
```

## Stage Handoff

When this skill completes, produce a structured handoff for the next stage. Include in the chat summary after the Report.

**Schema:**
- **Plan files:** `projects/{name}/plans/manifest.md` + component plans + `integration.md`
- **Task count:** {N} builder tasks + {N} validator tasks across {M} plans
- **Estimated phases:** Per-plan phasing details if applicable
- **Parallelism mode:** {serial | topology | aggressive}
- **Key decisions:** 3-5 bullet points of architectural/design decisions from the grouping phase
- **Constraints established:** Build unit constraints, dependency ordering, tier assignments, interface contracts
- **Open questions:** Unresolved MODERATE items from decisions.md
- **Scope:** Components and specs covered (from manifest)

---

## Phase: Quality Gate (Agent Review)

This phase dispatches designer and critic subagents to review all plan documents. Pattern: Quality Gate Cycling with multi-plan scope.

### User Opt-Out

Use AskUserQuestion:
- Question: "Would you like the plans to go through quality review before execution?"
- Options:
  1. "Yes, proceed with quality review (Recommended)" — Continue with this phase.
  2. "No, skip to execute" — Output `Next step: /forge-execute {project-name}` and stop.

If the user selects skip, output `Next step: /forge-execute {project-name}` and stop.

### Review Scope

Reviewers receive the manifest (as the structural overview) plus ALL component plans and the integration plan. Count total lines across all plan files. If total exceeds 800 lines, split into batches of ~400-600 lines each. Batch (a) always includes: manifest + foundation plan + integration plan. Remaining component plans are distributed across additional batches. Every batch includes the manifest for cross-reference.

### Review Criteria

Per-plan criteria (applied to each plan individually):
- Task specificity (file paths, verification commands, expected outcomes)
- Dependency correctness (no task references types from undeclared dependencies)
- Verification executability (commands can actually run)
- Team allocation (right agent types for right tasks)
- Acceptance criteria coverage (all objectives have verification)
- Scope control (task count matches objective)

Cross-plan criteria (applied across all plans):
- Dependency alignment between plans (plan A declares it blocks plan B; plan B declares it depends on plan A)
- Interface contract consistency (exports match imports — types, paths, signatures)
- No duplicate tasks across plans
- Tier ordering correctness (no plan in tier N depends on a plan in tier N or higher)

### Review Cycles (5-cycle cap)

**Cycle 1: Initial Review**

Dispatch both agents in parallel using `run_in_background: true`:

**Designer** — Assess architectural feasibility:

```
Task({
  description: "Design review of forge plans",
  prompt: "Read projects/{name}/plans/manifest.md, then all component and integration plans. Evaluate:\n1. Approach alignment with codebase patterns\n2. Unaddressed architectural risks\n3. Task boundary cleanliness (no cross-task knowledge leaks)\n4. Builder/validator pair correctness\n5. Specification level (not over/under-specified)\n6. CROSS-PLAN: Interface contract alignment, exports match imports\n7. CROSS-PLAN: Tier ordering, high-conflict file identification\n\nFormat: ## Verdict: {PASS|REVISE} / ## Findings / ## Required Changes / ## Gotchas Identified\nDo NOT write files.",
  subagent_type: "designer",
  run_in_background: true
})
```

**Critic** — Stress-test plan quality:

```
Task({
  description: "Critic review of forge plans",
  prompt: "Read projects/{name}/plans/manifest.md, then all component and integration plans. Evaluate:\n1. Task specificity (file paths, verify commands, expected outcomes)\n2. Dependency correctness\n3. Verification executability\n4. Team allocation\n5. Acceptance criteria coverage\n6. Scope control\n7. CROSS-PLAN: Duplicate tasks across plans\n8. CROSS-PLAN: Plan dependencies match tier ordering\n\nFormat: ## Verdict: {PASS|REVISE} / ## Findings / ## Required Changes / ## Gotchas Identified\nDo NOT write files.",
  subagent_type: "critic",
  run_in_background: true
})
```

**Subagent failure handling:** If a subagent fails or times out, log the failure, note "{agent} review unavailable" in the plan's Notes section, and continue with results from any agents that did complete. Do not block the pipeline on a failed agent.

**After Cycle 1:**

Collect verdicts from both agents. Present feedback to the user constructively — lead with strengths, then improvements.

- If all verdicts are PASS → Output `Next step: /forge-execute {project-name}` and stop.
- If any verdict is REVISE → Continue to next revision cycle.

**Revision Cycles (up to 4 additional)**

1. Parse Required Changes from REVISE verdicts into individual items.
2. Present changes to user via AskUserQuestion (up to 4 per round). Each question includes: reviewer source, change description, reasoning, impact of not applying. Options: Approve / Deny / Revise.
3. Apply approved changes. Log denied changes in plan Notes: "Reviewer suggested: {summary} — deferred by user."
4. Re-dispatch agents that returned REVISE.
5. PASS → finalize. REVISE with cycles remaining → repeat.

After cycle 5: finalize regardless. Log unresolved issues in Notes.

### Gotcha Log Orchestration

After quality gate: extract `## Gotchas Identified` from reviewer output. For each new gotcha (not already in `~/BASECAMP/docs/logs/gotcha-log.md`), append using format: `## [{tag}] {Title}` / `**Date:** {YYYY-MM-DD}` / `{2-4 sentence description}` / `---`. Tags: `workflow`, `implementation`, `config`, `convention`, `tooling`, `process`.

## Related Skills

- `forge` — Upstream: produces the project artifacts this skill transforms into plans
- `forge-execute` — Downstream (pending creation): runs the implementation plans
