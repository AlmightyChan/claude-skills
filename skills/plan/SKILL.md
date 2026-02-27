---
name: plan
description: "Use when you have a technical document that needs implementation task decomposition"
argument-hint: [user prompt or path to PRD/pitch]
model: opus
disallowed-tools: [EnterPlanMode]
hooks:
  Stop:
    - hooks:
        - type: command
          command: >-
            /Users/ethanmuna/.local/bin/uv run
            $HOME/.claude/hooks/validators/validate_file_contains.py
            --directory docs/plan --extension .md --prefix plan-
            --contains '## Task Description'
            --contains '## Objective'
            --contains '## Project Directory'
            --contains '## Relevant Files'
            --contains '## Step by Step Tasks'
            --contains '## Acceptance Criteria'
            --contains '## Team Orchestration'
            --contains '### Team Members'
version: 0.1.0
---

# Plan

- **Agents:** orchestrator-invoked

Create a detailed implementation plan based on the user's requirements. Analyze the request, think through the implementation approach, and save a comprehensive specification document to `docs/plan/<name-of-plan>.md` that can be used as a blueprint for development work. Follow the `Instructions` and work through the `Workflow`.

## Variables

USER_PROMPT: $ARGUMENTS
PLAN_OUTPUT_DIRECTORY: `docs/plan/`
TEAM_MEMBERS: `/Users/ethanmuna/.claude/agents/*.md`
AGENT_ROSTER: Select the most appropriate agent from the 6 existing agents: builder, critic, validator, researcher, designer, auditor

## Instructions

- **PLANNING ONLY**: Do NOT build, write code, or deploy agents. Your only output is a plan document saved to `PLAN_OUTPUT_DIRECTORY`.
- Create `docs/plan/` directory if it does not exist.
- If no `USER_PROMPT` is provided, stop and ask the user to provide it.
- **PRD/Pitch Input Support**: If `USER_PROMPT` is a path to a `prd-*.md` or `pitch-*.md` file (check if the file exists), read it and use its content as the primary input for plan generation. For PRD files: functional requirements map to tasks, scope boundaries constrain the plan, and success criteria become acceptance criteria. For pitch files: the Proposed Solution maps to tasks, Scope Boundaries and No-Gos constrain the plan, Success Criteria become acceptance criteria, Rabbit Holes (if present) inform risk-first task sequencing, and Technical Context (if present) provides existing codebase understanding. The Appetite constrains overall plan size — do not exceed the effort budget. If `USER_PROMPT` is a raw prompt (not a file path), proceed with the prompt as input.
- **Companion Document Detection**: After reading the primary input, check for companion documents:
  - **Determine lookup scope**: If the input is inside a `projects/{project}/docs/` path, look for companions in the same directory. Otherwise, PRDs are in `docs/prd/` and Brand OS/specs are in `docs/specs/`.
  - If the input is a PRD (`prd-*.md`): look for `brand-os-*.md` in the lookup scope. If found, read it.
  - If the input is a Brand OS (`brand-os-*.md`): look for `prd-*.md` in the lookup scope. If found, read it.
  - Check the lookup scope for additional companion document types:
    - Architecture docs (`architecture-*.md`): system architecture, component diagrams, boundary definitions.
    - Feature specs (`feature-*.md`): detailed feature specifications and user stories.
    - Service definitions (`service-*.md`): service contracts, API boundaries, integration points.
    - System requirement docs (`srd-*.md`): system-level requirements, constraints, and compliance criteria.
  - **Prerequisite verification gate**: After all companion doc checks, if no PRD (`prd-*.md`) or pitch (`pitch-*.md`) was found in the lookup scope AND the input document is not itself a PRD or pitch, use AskUserQuestion: "No PRD or pitch document found for this project. Proceed without one?" with options:
    1. "Proceed without PRD" — continue planning and note the gap in the plan's `## Notes` section as a warning.
    2. "Create one first" — stop planning and suggest the user run `/refine` to create a PRD or pitch before planning.
    3. "Specify path" — let the user provide a custom file path to a PRD or pitch document, then read it and continue.
    For Brand OS, architecture docs, feature specs, service definitions, and SRD docs: if absent, log their absence in the plan's `## Notes` section as a warning but do not block.
  - For each companion document found, record its path and type in the plan's `## Provisioning` section (see plan format).
  - Read ALL companion documents found to understand the full project scope before designing tasks.
  - Note all companion documents in the plan's `## Relevant Files` section with loading guidance. Use the document's current path (whether staging or project directory).
  - For each task, populate `**Context Files**` based on what the task needs:
    - **Brand OS only**: tasks involving UI layout, visual styling, copy/voice, interaction patterns, motion, or brand-specific behavior (e.g., building a view, writing user-facing copy, implementing animations).
    - **PRD only**: tasks involving backend logic, data models, business rules, non-functional requirements, or technical infrastructure with no UI component (e.g., implementing an API, setting up data persistence, configuring auth).
    - **Both PRD and Brand OS**: tasks that implement user-facing features requiring both the functional requirement spec (what to build) and the brand design spec (how it should look, sound, and feel) (e.g., building a screen that fulfills a specific functional requirement with brand-consistent UI).
    - **Neither**: scaffolding, configuration, or setup tasks with no feature or design decisions. Omit the field.
- Carefully analyze the requirements
- Determine the task type (chore|feature|refactor|fix|enhancement) and complexity (simple|medium|complex)
- Think deeply about the best approach to implement the requested functionality
- Understand the codebase directly (without subagents) to understand existing patterns and architecture
- Follow the Plan Format below to create the implementation plan
- Include all required sections and conditional sections based on task type and complexity
- Generate a descriptive, kebab-case filename based on the main topic
- Save the complete plan to `PLAN_OUTPUT_DIRECTORY/<descriptive-name>.md`
- Ensure the plan is detailed enough that another developer could follow it
- Include code examples or pseudo-code where appropriate
- Consider edge cases, error handling, and scalability
- You are the team lead. Refer to the `Team Orchestration` section for details.

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

**Parallel reconciliation checkpoints**: After every batch of `Parallel: true` builder tasks that produce types, protocols, or components consumed by later tasks, insert a reconciliation checkpoint task. This is a validator task that reads all files from the parallel batch, builds a type registry, and verifies cross-task compatibility (no duplicate types, no conflicting signatures, imports resolve). See the reconciliation checkpoint template in the Plan Format section. Note: The orchestrator's pre-dispatch type registry (in the execute skill) is a preventive input that reduces the chance of conflicts; the reconciliation checkpoint is a post-completion verification that detects conflicts that occurred despite the registry. Both are needed — prevention alone is insufficient when builders generate new types.

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

Example of well-specified builder/validator pair:
```
### 3. Implement login endpoint
- **Task ID**: impl-login
- **Depends On**: setup-database
- **Assigned To**: builder-api
- **Agent Type**: builder
- **Parallel**: false
- **Validate**: true
- Create `src/routes/login.py` with POST /login accepting email+password
- Add `LoginRequest` and `LoginResponse` models to `src/models/auth.py`
- **Verify**: `pytest tests/test_login.py -v` -- all 5 tests pass
- **Expected**: "5 passed" including auth success, invalid creds, missing fields, token format, and rate limit
- **Behavioral**: Valid credentials return token + 200; invalid return error message + 401; missing fields return validation errors + 422

### 4. Validate login endpoint
- **Task ID**: validate-login
- **Depends On**: impl-login
- **Assigned To**: validator-api
- **Agent Type**: validator
- **Parallel**: false
- Verify `src/routes/login.py` exists and handles POST /login
- Verify `src/models/auth.py` contains LoginRequest and LoginResponse
- Run `pytest tests/test_login.py -v` and confirm all tests pass
- **Functional completeness**: Verify handler authenticates against real store (not hardcoded), returns signed token (not placeholder string), validates all input fields, returns proper HTTP status codes for each error case
- Compare implementation against task 3 requirements line by line
```

### Team Orchestration

As the team lead, you have access to tools for coordinating work across multiple agents. You NEVER write code directly - you orchestrate team members.

#### Task Management Tools

**TaskCreate** - Create tasks in the shared task list:
```typescript
TaskCreate({
  subject: "Implement user authentication",
  description: "Create login/logout endpoints with JWT tokens. See docs/plan/auth-plan.md for details.",
  activeForm: "Implementing authentication"
})
// Returns: taskId (e.g., "1")
```

**TaskUpdate** - Update task status, assignment, or dependencies:
```typescript
TaskUpdate({
  taskId: "1",
  status: "in_progress",
  owner: "builder-auth"
})
```

**TaskList** - View all tasks and their status:
```typescript
TaskList({})
```

**TaskGet** - Get full details of a specific task:
```typescript
TaskGet({ taskId: "1" })
```

#### Task Dependencies

Use `addBlockedBy` to create sequential dependencies:

```typescript
TaskUpdate({
  taskId: "2",
  addBlockedBy: ["1"]
})
```

Dependency chain example:
```
Task 1: Setup foundation       -> no dependencies
Task 2: Implement feature      -> blockedBy: ["1"]
Task 3: Validate feature       -> blockedBy: ["2"]
Task 4: Implement next feature -> blockedBy: ["3"]
Task 5: Validate next feature  -> blockedBy: ["4"]
Task 6: Final acceptance       -> blockedBy: ["3", "5"]
```

#### Agent Deployment with Task Tool

```typescript
Task({
  description: "Implement auth endpoints",
  prompt: "Implement the authentication endpoints as specified in Task 1...",
  subagent_type: "builder",
  model: "opus",
  run_in_background: false
})
```

#### Resume Pattern

```typescript
// First deployment
Task({
  description: "Build user service",
  prompt: "Create the user service with CRUD operations...",
  subagent_type: "builder"
})
// Returns: agentId: "abc123"

// Resume with preserved context
Task({
  description: "Continue user service",
  prompt: "Now add input validation...",
  subagent_type: "builder",
  resume: "abc123"
})
```

When to resume vs start fresh:
- **Resume**: Continuing related work, agent needs prior context
- **Fresh**: Unrelated task, clean slate preferred

#### Parallel Execution

```typescript
Task({
  description: "Build API endpoints",
  prompt: "...",
  subagent_type: "builder",
  run_in_background: true
})

Task({
  description: "Build frontend components",
  prompt: "...",
  subagent_type: "builder",
  run_in_background: true
})

// Check progress
TaskOutput({ task_id: "agentId", block: false, timeout: 5000 })

// Wait for completion
TaskOutput({ task_id: "agentId", block: true, timeout: 300000 })
```

#### Orchestration Workflow

1. **Create tasks** with `TaskCreate` for each step
2. **Set dependencies** with `TaskUpdate` + `addBlockedBy`
3. **Assign owners** with `TaskUpdate` + `owner`
4. **Deploy agents** with `Task` to execute assigned work
5. **Monitor progress** with `TaskList` and `TaskOutput`
6. **Resume agents** with `Task` + `resume` for follow-up work
7. **Mark complete** with `TaskUpdate` + `status: "completed"`

## Workflow

IMPORTANT: **PLANNING ONLY** - Do not execute, build, or deploy. Output is a plan document.

1. Analyze Requirements - Parse USER_PROMPT (or read PRD/pitch file) to understand the core problem and desired outcome
2. Understand Codebase - Directly understand existing patterns, architecture, and relevant files. **Detect the project directory**: If the input document is inside `projects/{project}/docs/`, use `projects/{project}` as the project directory. Otherwise, check if the document references a specific project (by name or path) and verify `projects/{name}/` exists. If no project context can be determined, use `basecamp`. Record this in the plan's `## Project Directory` section.

   **Project Characteristic Detection**: During codebase analysis, detect technology signals to inform review task generation:

   | Signal | Detection Method | Review Skill |
   |--------|-----------------|--------------|
   | Database layer | models/, migrations/, schema files, ORM config | review-database* |
   | API layer | routes/, controllers/, handlers/, OpenAPI specs | review-api* |
   | Frontend | components/, state management, React/Vue/Angular markers | review-frontend* |
   | Backend services | services/, domain logic, queue/worker patterns | review-backend* |
   | CI/CD | .github/workflows/, Dockerfile, CI config | review-devops* |
   | Large codebase | 50+ source files | review-architecture |
   | Any code changes | Always | review-code-quality |

   *Tier 2 skills -- generate tasks only when the skill exists on disk (check with `ls ~/.claude/skills/{name}/SKILL.md`).

   Record detected characteristics in the plan under `## Project Topology`.
   When generating review tasks, reference `.claude/rules/subagent-skill-dispatch.md` for the skill-to-agent mapping.

   **Provisioning**: After companion document detection, populate the `## Provisioning` section of the plan with: (a) the list of companion documents found (paths and types), and (b) the project CLAUDE.md and decision log templates from `supporting-files/provisioning-templates.md`. If no companion documents were found, the Provisioning section should still be included with an empty Reference Documents list.
3. Design Solution - Develop technical approach including architecture decisions and implementation strategy
4. Define Team Members - Identify from `/Users/ethanmuna/.claude/agents/*.md`, selecting the most appropriate agent from the roster (builder, critic, validator, researcher, designer, auditor). Document in plan.
5. Define Step by Step Tasks - Write tasks with IDs, dependencies, assignments. Interleave builder/validator pairs for every code-producing task. Document in plan.
5.5. Context Budget Check - After task decomposition, estimate the total context budget:
   a. Read `supporting-files/phasing-guide.md` for unit costs and threshold.
   b. Sum estimated units across all tasks (builder ~15-25, validator ~8-12, other agents ~5-10 each).
   c. If total ≤ 200 units: single-phase plan, no changes needed.
   d. If total > 200 units: apply phasing per the guide's Phase Boundary Rules. Split into phases, add checkpoint blocks per the Checkpoint Format, and include phase boundaries under `## Implementation Phases`. Never split builder/validator pairs across phase boundaries.
   e. Include cost estimate in the plan's Notes section: "Estimated cost: ~{N} operational units across {M} phase(s)."
6. Cross-Check Requirement Coverage - After task decomposition, verify traceability:
   a. List every requirement from the input document (PRD sections, pitch decisions, or user prompt items).
   b. For each requirement, identify which task(s) address it. If any requirement has no corresponding task, add a task for it.
   c. For each task, confirm it traces to at least one requirement. Flag untraceable tasks as potential scope creep and remove unless justified.
   d. Document the mapping briefly in the plan's Acceptance Criteria section (e.g., "Req 1 → Task impl-auth, validate-auth").
7. Generate Filename - Create a descriptive kebab-case filename
8. Approval Gate - Present a summary before writing the plan:
   - Topic, task count (N builder + N validator), team members, estimated cost (~{2N} subagent calls)
   Use AskUserQuestion: "Write this plan to disk?" with options:
   1. "Write plan (Recommended)"
   2. "Revise approach" — return to step 3 (Design Solution) with user feedback
   3. "Abort" — stop without writing any file
   Only proceed to step 9 on "Write plan" approval.
9. Save Plan - Write to `docs/plan/<filename>.md`
10. Report - Follow the Report section

## Plan Format

- IMPORTANT: Replace `<requested content>` with actual content. It's templated for you to replace.
- IMPORTANT: Anything NOT in `<requested content>` should be written EXACTLY as it appears.
- IMPORTANT: The section headings below MUST match exactly -- the Stop hook validates their presence.

```md
# Plan: <task name>

## Task Description
<describe the task in detail based on the prompt or PRD>

## Objective
<clearly state what will be accomplished when this plan is complete>

## Project Directory
<relative path to the project root, e.g., `projects/cadence`. Use `basecamp` if this plan targets BASECAMP infrastructure itself. The execute skill uses this to move technical documents to the project and archive the plan after completion.>

<if task_type is feature or complexity is medium/complex, include these sections:>
## Problem Statement
<clearly define the specific problem or opportunity>

## Solution Approach
<describe the proposed solution and how it addresses the objective>
</if>

## Relevant Files
Use these files to complete the task:

<list files relevant to the task with bullet points explaining why. Include new files under an h3 'New Files' section if needed>

<if complexity is medium/complex:>
## Implementation Phases
### Phase 1: Foundation
<foundational work needed>

### Phase 2: Core Implementation
<main implementation work>

### Phase 3: Integration & Polish
<integration, testing, final touches>
</if>

## Project Topology

Detect the project type during step 2 (Understand Codebase) by checking for project marker files:
1. `.xcodeproj`/`.xcworkspace` → parse `project.pbxproj` for target count → `xcode-single` or `xcode-multi`
2. `Package.swift` → parse for `.target()` entries → `spm-single` or `spm-multi`
3. `Cargo.toml` → check for `[workspace]` → `rust-single` or `rust-workspace`
4. `go.mod` → `go-module`
5. `package.json` → check for `workspaces` → `node-single` or `node-monorepo`
6. `pyproject.toml`/`setup.py`/`requirements.txt` → `python`
7. Multiple project roots (e.g., `backend/` + `frontend/`) → `full-stack`
8. Fallback → `unknown` (defaults to serial)

Record the topology in this format:

```markdown
- **Project Type**: <detected type>
- **Language**: <primary language>
- **Compilation Model**: <compiled | interpreted | hybrid>
- **Build Units**:
  - <unit-path> (<unit-type>) — <dependency notes>
- **Parallelism Profile**: <serial | moderate | high>
  - <brief justification>
- **Parallelism mode**: <serial | topology | aggressive> (recommended based on profile)
```

For single-build-unit compiled projects (e.g., xcode-single):
```markdown
- **Project Type**: xcode-single
- **Language**: Swift
- **Compilation Model**: compiled
- **Build Units**:
  - `Cadence` (single app target) — all source files
- **Parallelism Profile**: serial within target
- **Parallelism mode**: serial
```

Parallelism mode recommendations:
- `xcode-single`, `spm-single`, `rust-single`, `unknown` → `serial`
- `xcode-multi`, `spm-multi`, `rust-workspace`, `node-monorepo`, `full-stack` → `topology`
- `python`, `node-single` with independent packages → `topology` or `aggressive`

## Provisioning

### Project CLAUDE.md
<Generated by the plan skill. Contains doc references and living-file rules for this project. Populate from companion documents found during step 2 and templates in supporting-files/provisioning-templates.md.>

Content:
```markdown
# {Project Name}

## Reference Documents
<list of companion documents found during planning, with paths and descriptions>

## Living-File Rules
- When a design decision changes during execution, update the relevant reference document (PRD, Brand OS, spec) to reflect the new decision.
- When adding a new file or module not covered by existing docs, note it in this file under a `## New Components` section.
- Do not silently diverge from specs — either update the spec or flag the divergence.
```

### Decision Log
Location: `{project-dir}/docs/decision-log.md`
Format: append-only markdown

```markdown
# Decision Log

Tracks decisions that diverge from or extend the original plan and specs. Written by the execute orchestrator only.

---
```

## Team Orchestration

- You operate as the team lead and orchestrate the team to execute the plan.
- You're responsible for deploying the right team members with the right context.
- IMPORTANT: You NEVER operate directly on the codebase. You use `Task` and `Task*` tools to deploy team members for building, validating, testing, and other tasks.
- You validate all work is going well and make sure the team is on track.
- Take note of the session id of each team member for reference.

### Team Members
<list the team members needed -- use the agent types appropriate to the plan>

Available agent types: builder, validator, researcher, critic, designer, auditor

- Builder
  - Name: <unique name for this builder>
  - Role: <the single role and focus>
  - Agent Type: builder
  - Resume: <default true>
- Validator
  - Name: <unique name for this validator -- paired with the builder above>
  - Role: <validation focus>
  - Agent Type: validator
  - Resume: <default true>

<include any of these optional team members when the plan requires them:>

- Researcher (optional -- for tasks requiring information gathering before building)
  - Name: <unique name>
  - Role: <research focus>
  - Agent Type: researcher
  - Resume: false
- Critic (optional -- for pre-build review of plans/designs)
  - Name: <unique name>
  - Role: <review focus>
  - Agent Type: critic
  - Resume: false
- Designer (optional -- for architecture/design tasks before building)
  - Name: <unique name>
  - Role: <design focus>
  - Agent Type: designer
  - Resume: false
- <continue with additional team members as needed>

## Step by Step Tasks

- IMPORTANT: Execute every step in order, top to bottom. Each task maps to a `TaskCreate` call.
- Before starting, run `TaskCreate` for all tasks so all team members can see the full plan.
- For every builder task with `Validate: true`, a corresponding validator task MUST follow immediately.

<list step by step tasks as h3 headers>

### 1. <First Task Name>
- **Task ID**: <unique kebab-case identifier>
- **Depends On**: <Task ID(s) or "none">
- **Assigned To**: <team member name>
- **Agent Type**: <subagent type>
- **Parallel**: <true/false — consult Project Topology and Build Unit constraint>
- **Build Unit**: <build unit path from Project Topology, or "none" for non-code tasks>
- **Validate**: <true/false -- false for scaffolding/config tasks>
- **Context Files**: <path(s) to load before execution. Use Brand OS for UI/visual/copy tasks, PRD for requirement/logic tasks, both for UI tasks implementing specific requirements. Omit for scaffolding/config tasks.>
- <specific action with exact file path>
- <specific action with exact file path>
- **Verify**: <command to run>
- **Expected**: <expected output>
- **Behavioral**: <1-2 sentences: user action → observable outcome> (required when Validate is true and task creates UI/endpoints; omit for scaffolding/config tasks)

### 2. <Validation of First Task> (only if task 1 has Validate: true)
- **Task ID**: validate-<matching task id>
- **Depends On**: <task 1 ID>
- **Assigned To**: <validator name>
- **Agent Type**: validator
- **Parallel**: false
- Verify <file path> exists and matches requirements
- Run <verification command> and confirm output
- **Functional completeness**: <2-4 domain-specific assertions verifying REAL behavior, not stubs. Name specific functions/handlers and what they must actually do. Example: "Verify loginHandler authenticates against real credential store (not hardcoded), returns signed JWT (not placeholder string), validates all input fields, returns proper HTTP status codes for each error case." Anti-example: "Verify CadenceDesign.dotDiameter == 64" — constants-only checks verify configuration, not behavior.>
- Compare implementation against task 1 requirements line by line

### 3. <Continue Pattern -- builder/validator pairs for implementation tasks>

<After any batch of Parallel: true builder tasks that produce types, protocols, or components consumed by later tasks, insert a reconciliation checkpoint:>

### M. <Reconciliation Checkpoint: {component boundary description}>
- **Task ID**: reconcile-<batch-name>
- **Depends On**: <all Task IDs in the parallel batch AND their validators>
- **Assigned To**: <validator name>
- **Agent Type**: validator
- **Parallel**: false
- **Validate**: false
- Read all files produced by the parallel batch
- Build a type registry: list every public type, protocol, enum, and function signature created
- Verify no duplicate type names, no conflicting protocol conformances, no incompatible signatures
- Verify import statements across parallel outputs resolve correctly
- Verify shared types used by multiple parallel tasks have identical definitions
- If conflicts found: report each conflict with file:line pairs from both sides
- **Expected Output**: Type registry document (inline in report) plus PASS/FAIL verdict

<Non-builder/validator tasks run once without validation cycling. Use these patterns when the plan includes research, design, critique, or auditor phases:>

### N-2. <Research/Design/Critique/Specialist Task> (single-run, no validator pair)
- **Task ID**: <unique kebab-case identifier>
- **Depends On**: <Task ID(s) or "none">
- **Assigned To**: <team member name>
- **Agent Type**: <researcher | critic | designer | auditor>
- **Parallel**: <true/false>
- **Validate**: false
- **Skills**: <skill names to load for this task, if applicable>
- <specific action>
- <specific action>
- **Expected Output**: <document, verdict, design artifact, or findings>

### N-2. <Continue with more tasks as needed>

### N-1. Integration Wiring (required for plans with 5+ builder tasks)
- **Task ID**: integration-wiring
- **Depends On**: <all builder/validator Task IDs>
- **Assigned To**: <builder name>
- **Agent Type**: builder
- **Parallel**: false
- **Validate**: true
- Wire all components to the app entry point, navigation, and dependency injection
- Verify real implementations are injected — not stubs, facades, or placeholder types
- Ensure navigation reaches real views with real data
- Trace at least one end-to-end user flow across component boundaries (e.g., user action → business logic → data persistence → UI update)
- **Verify**: <full build + run command>
- **Expected**: Build passes, app launches, end-to-end flow works
- **Behavioral**: <1-2 sentences describing a concrete user flow that crosses task boundaries>
- **Functional completeness**: Verify dependency injection uses real implementations (not mocks/stubs), navigation targets are real views (not placeholders), data flows end-to-end through all layers without dead ends

<if plan has fewer than 5 builder tasks, this task may be omitted — the final acceptance cross-task integration check covers it>

### N. Final Acceptance
- **Task ID**: final-acceptance
- **Depends On**: <all previous validator Task IDs>
- **Assigned To**: <validator>
- **Agent Type**: validator
- **Parallel**: false
- Run all validation commands from every task
- Verify all acceptance criteria are met
- **Functional completeness sweep**: For every UI element or endpoint created across ALL tasks, verify: event handlers fire and produce effects, state updates are reflected visually, data persists where specified, navigation reaches real destinations, error states produce user feedback. Check for stub patterns: `// TODO`, `throw new Error("not implemented")`, `pass`, `() => {}`, `console.log` as handler logic.
- **Cross-task integration check**: Trace at least one end-to-end user flow that spans multiple tasks (e.g., form submission → API call → database write → success feedback). Verify the seams between tasks are connected.

## Acceptance Criteria
<list specific, measurable criteria>

## Validation Commands
Execute these commands to validate completion:

<list specific commands>

## Notes
<optional additional context>
```

## Report

After saving the plan:

```
Implementation Plan Created

File: docs/plan/<filename>.md
Topic: <brief description>
Key Components:
- <main component 1>
- <main component 2>
- <main component 3>

Task List:
- <list of tasks and owners (concise)>

Team Members:
- <list of team members and roles (concise)>

Cost Estimate: ~{N} builder tasks + {N} validator tasks = {2N} subagent calls minimum

Next step: Run `/execute docs/plan/<filename>.md` to execute the plan.
```

---

## Phase: Quality Gate (Agent Review)

This phase dispatches designer and critic subagents to review the plan document. This replaces the previous Team Review phase with a more structured quality gate. Pattern 2 (Quality Gate Cycling) governs this phase.

### User Opt-Out

Use AskUserQuestion:
- Question: "Would you like the plan to go through quality review before building?"
- Options:
  1. "Yes, proceed with quality review (Recommended)" — Continue with this phase.
  2. "No, skip to execute" — Output `Next step: /execute docs/plan/<filename>.md` and stop.

If the user selects skip, output `Next step: /execute docs/plan/<filename>.md` and stop.

### Review Cycles (5-cycle cap)

**Cycle 1: Initial Review**

Dispatch both agents in parallel using `run_in_background: true`:

**Designer** — Assess architectural feasibility:

```
Task({
  description: "Design review of plan",
  prompt: "Read the implementation plan at docs/plan/<filename>.md. Evaluate architectural feasibility:\n\n1. Does the approach align with existing codebase patterns? Are there pattern conflicts?\n2. Are there architectural risks not addressed in the plan?\n3. Are the task boundaries clean — does any task require knowledge from a parallel task that hasn't completed yet?\n4. Are the builder/validator pairs properly interleaved with correct dependencies?\n5. Is the level of specification appropriate — not over-engineered or under-specified?\n\nProduce your response in this format:\n\n## Verdict: {PASS|REVISE}\n\n## Findings\n{Numbered findings for each point}\n\n## Required Changes\n{If REVISE: specific, actionable changes needed. If PASS: 'None'}\n\n## Gotchas Identified\n{Any architectural or workflow mistakes spotted. If none: 'None'}\n\nDo NOT write any files. Return assessment in conversation only.",
  subagent_type: "designer",
  run_in_background: true
})
```

**Critic** — Stress-test plan quality:

```
Task({
  description: "Critic review of plan",
  prompt: "Read the implementation plan at docs/plan/<filename>.md. Evaluate using this plan-specific checklist:\n\n1. Task Specificity — Does every task have exact file paths, verification commands, and expected outcomes? Are any tasks vague?\n2. Dependency Correctness — Are task dependencies accurate? Could any task start before its true prerequisites complete?\n3. Verification Executability — Are verify commands actually executable? Would they produce the expected output on a clean run?\n4. Team Allocation — Are the right agent types assigned to the right tasks? Are builder/validator pairs correctly matched?\n5. Acceptance Criteria Coverage — Do the acceptance criteria cover all stated objectives? Are any objectives missing verification?\n6. Scope Control — Does the task count match the objective? Are there tasks that go beyond what was requested?\n\nProduce your response in this format:\n\n## Verdict: {PASS|REVISE}\n\n## Findings\n{Numbered findings for each checklist item}\n\n## Required Changes\n{If REVISE: specific, actionable changes needed. If PASS: 'None'}\n\n## Gotchas Identified\n{Any workflow, implementation, or convention mistakes spotted. If none: 'None'}\n\nDo NOT write any files. Return findings in conversation only.",
  subagent_type: "critic",
  run_in_background: true
})
```

**Subagent failure handling:** If a subagent fails or times out, log the failure, note "{agent} review unavailable" in the plan's Notes section, and continue with results from any agents that did complete. Do not block the pipeline on a failed agent.

**After Cycle 1:**

Collect verdicts from both agents. Present feedback to the user constructively — lead with strengths, then improvements.

- If all verdicts are PASS → Output `Next step: /execute docs/plan/<filename>.md` and stop.
- If any verdict is REVISE → Continue to next revision cycle.

**Revision Cycles (up to 4 additional cycles)**

For each revision cycle:

1. Re-read the critic's (and designer's, if applicable) Required Changes sections. Parse each distinct change into a separate item.
2. Present proposed changes to the user in rounds of up to 4 per AskUserQuestion call. Each change is a separate question within the call. For each question:
   - **Question text**: Include: (a) which reviewer proposed this change (Designer or Critic), (b) a detailed description of what the change is and what part of the plan it affects, (c) the reviewer's reasoning for why it's needed, and (d) what would happen if the change is not applied.
   - **Options** (same for every question):
     1. "Approve" — apply this change to the plan
     2. "Deny" — skip this change, note in plan's Notes section
     3. "Revise" — let the user modify the change before applying
   - If more than 4 changes exist, continue with additional rounds until all changes have been reviewed.
   - After each round, for any "Revise" selections, ask the user for their modified version before proceeding to the next round.
3. Apply all user-approved changes to the plan document at `docs/plan/<filename>.md`. For skipped changes, add to the plan's Notes section: "Reviewer suggested: {change summary} — deferred by user."
4. Re-dispatch the same agents that returned REVISE verdicts, using the same prompts.
5. If all verdicts are PASS → Proceed to finalization.
6. If any verdict is REVISE and cycles remain → Continue to the next revision cycle.

**After Cycle 5 (or earlier PASS):**

Regardless of the verdict, proceed to finalization. If any issues remain unresolved after the final cycle, add them to the plan's Notes section with a note: "Flagged during quality review — not resolved within 5-cycle cap."

Output `Next step: /execute docs/plan/<filename>.md` and stop.

### Gotcha Log Orchestration

After the quality gate completes (whether after Cycle 1 PASS or after a later cycle), extract any items from `## Gotchas Identified` sections in critic and designer output. For each identified gotcha:

1. Read `~/BASECAMP/docs/logs/gotcha-log.md`.
2. Check for duplicates: if a gotcha with substantially the same issue already exists, skip it (do not create duplicate entries).
3. For new gotchas: append an entry at the top of the log (below the header) using the standard format:
   ```markdown
   ## [{tag}] {Title}
   **Date:** {YYYY-MM-DD}

   {Clear description in 2-4 sentences.}

   ---
   ```
4. Use the appropriate category tag: `workflow`, `implementation`, `config`, `convention`, `tooling`, or `process`.

## Stage Handoff

When this skill completes, produce a structured handoff for the next stage. This handoff is part of the Report output — include it in the chat summary.

**Schema:**
- **Output file:** `docs/plan/{plan-name}.md`
- **Task count:** {N} builder tasks + {N} validator tasks
- **Estimated phases:** {N} phase(s), estimated ~{N} operational units
- **Parallelism mode:** {serial | topology | aggressive}
- **Key decisions:** 3-5 bullet points of architectural/design decisions and why
- **Constraints established:** Build unit constraints, dependency ordering, team member assignments
- **Open questions:** Unresolved items flagged during planning or quality review
- **Scope:** Files and modules in play (from Relevant Files section)

## Related Skills

- `refine` — Upstream: produces the technical document this skill decomposes
- `execute` — Downstream: runs the implementation plan

## Supporting Files

- `supporting-files/phasing-guide.md` — Context budget estimation and phasing methodology. Used in workflow step 5.5 (Context Budget Check).
- `supporting-files/provisioning-templates.md` — Canonical templates for project CLAUDE.md and decision log. Used in workflow step 2 (Provisioning) to populate the plan's `## Provisioning` section.

## Completion Token

When this skill completes successfully, output: `[SKILL_COMPLETE:plan]`
