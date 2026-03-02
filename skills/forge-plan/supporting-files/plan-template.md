## Calibration Example

Well-specified builder/validator pair:

```
### 3. Implement login endpoint
- **Task ID**: auth-core-impl-login
- **Depends On**: auth-core-setup-database
- **Assigned To**: builder-api
- **Agent Type**: builder
- **Skills**: [tdd, verification-before-completion]
- **Parallel**: false
- **Build Unit**: src/
- **Validate**: true
- **Context Specs**: specs/auth/user-authentication.md, product-brief.md, decisions.md
- Create `src/routes/login.py` with POST /login accepting email+password
- Add `LoginRequest` and `LoginResponse` models to `src/models/auth.py`
- **Verify**: `pytest tests/test_login.py -v` -- all 5 tests pass
- **Expected**: "5 passed" including auth success, invalid creds, missing fields, token format, and rate limit
- **Behavioral**: Valid credentials return token + 200; invalid return error message + 401; missing fields return validation errors + 422

### 4. Validate login endpoint
- **Task ID**: validate-auth-core-impl-login
- **Depends On**: auth-core-impl-login
- **Assigned To**: validator-api
- **Agent Type**: validator
- **Skills**: [verification-before-completion]
- **Parallel**: false
- Verify `src/routes/login.py` exists and handles POST /login
- Verify `src/models/auth.py` contains LoginRequest and LoginResponse
- Run `pytest tests/test_login.py -v` and confirm all tests pass
- **Functional completeness**: Verify handler authenticates against real store (not hardcoded), returns signed token (not placeholder string), validates all input fields, returns proper HTTP status codes for each error case
- Compare implementation against task 3 requirements line by line
```

Well-specified UI builder/validator pair with spec routing:

```
### 5. Implement home screen
- **Task ID**: app-shell-impl-home
- **Depends On**: app-shell-setup-navigation
- **Assigned To**: builder-ui
- **Agent Type**: builder
- **Skills**: [tdd, verification-before-completion]
- **Parallel**: false
- **Build Unit**: src/
- **Validate**: true
- **Context Specs**: specs/features/home-screen.md, specs/ux/brand-os.md, specs/ux/design-system.md, product-brief.md, decisions.md
- Create `src/screens/HomeScreen.tsx` with hero section, feature cards, and CTA button
- Apply typography, color, and spacing tokens from design system spec
- Implement responsive layout per brand-os breakpoint definitions
- **Verify**: `npx playwright test tests/e2e/home-screen.spec.ts`
- **Expected**: All visual assertions pass, layout matches design tokens, CTA navigates to onboarding
- **Behavioral**: User opens app → home screen renders with brand-consistent hero, feature cards, and CTA → tapping CTA navigates to onboarding flow

### 6. Validate home screen
- **Task ID**: validate-app-shell-impl-home
- **Depends On**: app-shell-impl-home
- **Assigned To**: validator-ui
- **Agent Type**: validator
- **Skills**: [verification-before-completion]
- **Parallel**: false
- Verify `src/screens/HomeScreen.tsx` exists and renders all required sections
- Verify typography, color, and spacing tokens match `specs/ux/design-system.md` definitions
- Verify responsive behavior at all brand-os breakpoint sizes
- Run `npx playwright test tests/e2e/home-screen.spec.ts` and confirm all tests pass
- **Functional completeness**: Verify hero section uses real content from product-brief (not Lorem Ipsum), feature cards render dynamic data (not hardcoded), CTA button triggers real navigation (not console.log), layout applies actual design tokens (not approximations)
- Compare implementation against specs/features/home-screen.md requirements line by line
```

---

# Plan: <plan-name> — <project-name>

## Task Description
<scope of this plan — which components, which specs it implements>

## Objective
<what will be accomplished when this plan completes>

## Project Directory
`projects/<project-name>`

## Worktree
- **Branch:** feat/<plan-slug>
- **Base:** main
- **Components:** <list of components this plan covers>
- **Blocks Plans:** <plans that depend on this one completing>
- **Depends On Plans:** <plans that must complete before this one starts, or "none">

## Problem Statement
<from the forge specs — what problem this component set solves>

## Solution Approach
<from the forge specs — how the components address it>

## Relevant Files
Use these files to complete the task:
<forge spec files, product-brief, decisions relevant to this scope>

## Interface Contract
### Exports (created by this plan, consumed by other plans)
- <TypeName>: <definition summary> — file: <path>
### Imports (created by other plans, consumed by this plan)
- <TypeName> from <plan-name>: <expected path>

## Project Topology

<project topology detected in workflow step 3 — use the format below>

- **Project Type**: <detected type>
- **Language**: <primary language>
- **Compilation Model**: <compiled | interpreted | hybrid>
- **Build Units**:
  - <unit-path> (<unit-type>) — <dependency notes>
- **Parallelism Profile**: <serial | moderate | high>
  - <brief justification>
- **Parallelism mode**: <serial | topology | aggressive> (recommended based on profile)

## Implementation Phases
<if plan exceeds 200 units — split into phases per workflow step 5f>

## Team Orchestration

- You operate as the team lead and orchestrate the team to execute the plan.
- You're responsible for deploying the right team members with the right context.
- IMPORTANT: You NEVER operate directly on the codebase. You use `Task` and `Task*` tools to deploy team members for building, validating, testing, and other tasks.
- You validate all work is going well and make sure the team is on track.
- Take note of the session id of each team member for reference.

### Team Members
<list the team members needed — use the agent types appropriate to the plan>

Available agent types: builder, validator, researcher, critic, designer, auditor

- Builder
  - Name: <unique name for this builder>
  - Role: <the single role and focus>
  - Agent Type: builder
  - Skills: [tdd, verification-before-completion]
  - Resume: true
- Validator
  - Name: <unique name for this validator — paired with the builder above>
  - Role: <validation focus>
  - Agent Type: validator
  - Skills: [verification-before-completion]
  - Resume: true

## Step by Step Tasks

- IMPORTANT: Execute every step in order, top to bottom. Each task maps to a `TaskCreate` call.
- Before starting, run `TaskCreate` for all tasks so all team members can see the full plan.
- For every builder task with `Validate: true`, a corresponding validator task MUST follow immediately.

### 1. <First Task Name>
- **Task ID**: <plan-slug>-<task-slug>
- **Depends On**: <Task ID(s) or "none">
- **Assigned To**: <team member name>
- **Agent Type**: <subagent type>
- **Skills**: [<skill-1>, <skill-2>]
- **Parallel**: <true/false — consult Project Topology and Build Unit constraint>
- **Build Unit**: <build unit path from Project Topology, or "none" for non-code tasks>
- **Validate**: <true/false — false for scaffolding/config tasks>
- **Context Specs**: <forge spec file paths this task implements>
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
- **Skills**: [verification-before-completion]
- **Parallel**: false
- Verify <file path> exists and matches requirements
- Run <verification command> and confirm output
- **Functional completeness**: <2-4 domain-specific assertions verifying REAL behavior, not stubs. Name specific functions/handlers and what they must actually do.>
- Compare implementation against task 1 requirements line by line

### 3. <Continue Pattern — builder/validator pairs for implementation tasks>

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

<Non-builder/validator tasks run once without validation cycling:>

### N-2. <Research/Design/Critique/Specialist Task> (single-run, no validator pair)
- **Task ID**: <unique kebab-case identifier>
- **Depends On**: <Task ID(s) or "none">
- **Assigned To**: <team member name>
- **Agent Type**: <researcher | critic | designer | auditor>
- **Parallel**: <true/false>
- **Validate**: false
- **Skills**: <skill names to load for this task, if applicable>
- <specific action>
- **Expected Output**: <document, verdict, design artifact, or findings>

### N-1. Integration Wiring (required for plans with 5+ builder tasks)
- **Task ID**: integration-wiring
- **Depends On**: <all builder/validator Task IDs>
- **Assigned To**: <builder name>
- **Agent Type**: builder
- **Skills**: [tdd, verification-before-completion]
- **Parallel**: false
- **Validate**: true
- Wire all components to the app entry point, navigation, and dependency injection
- Verify real implementations are injected — not stubs, facades, or placeholder types
- Ensure navigation reaches real views with real data
- Trace at least one end-to-end user flow across component boundaries
- **Verify**: <full build + run command>
- **Expected**: Build passes, app launches, end-to-end flow works
- **Behavioral**: <1-2 sentences describing a concrete user flow that crosses task boundaries>
- **Functional completeness**: Verify dependency injection uses real implementations (not mocks/stubs), navigation targets are real views (not placeholders), data flows end-to-end through all layers without dead ends

### N. Final Acceptance
- **Task ID**: final-acceptance
- **Depends On**: <all previous validator Task IDs>
- **Assigned To**: <validator>
- **Agent Type**: validator
- **Parallel**: false
- Run all validation commands from every task
- Verify all acceptance criteria are met
- **Functional completeness sweep**: For every UI element or endpoint created across ALL tasks, verify: event handlers fire and produce effects, state updates are reflected visually, data persists where specified, navigation reaches real destinations, error states produce user feedback. Check for stub patterns: `// TODO`, `throw new Error("not implemented")`, `pass`, `() => {}`, `console.log` as handler logic.
- **Cross-task integration check**: Trace at least one end-to-end user flow that spans multiple tasks. Verify the seams between tasks are connected.

## Acceptance Criteria
<from forge specs and roadmap>

## Validation Commands
<consolidated>

## Merge Readiness Checklist
Before merging this plan's branch:
- [ ] All tasks PASS
- [ ] Final acceptance PASS
- [ ] Interface Contract exports exist at declared paths
- [ ] No unresolved CRITICAL items
- [ ] Build passes in isolation

## Notes
<cost estimate, open questions, deferred items>
