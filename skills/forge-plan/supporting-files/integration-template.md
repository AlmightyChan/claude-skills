# Plan: Integration — <project-name>

## Task Description
Post-merge integration verification for <project-name>. Runs after all
component plan branches have merged to main.

## Objective
Verify all components are correctly wired, cross-component contracts hold,
and end-to-end user flows work.

## Project Directory
`projects/<project-name>`

## Worktree
N/A — executes on main after all component branches have merged.

## Relevant Files
<all spec files, product-brief.md, decisions.md, all component plan files>

## Interface Contract
N/A — this plan consumes all interface contracts; it does not export.

## Project Topology
<same as component plans>

## Team Orchestration

- You operate as the team lead and orchestrate the team to execute the plan.
- You're responsible for deploying the right team members with the right context.
- IMPORTANT: You NEVER operate directly on the codebase. You use `Task` and `Task*` tools to deploy team members for building, validating, testing, and other tasks.
- You validate all work is going well and make sure the team is on track.
- Take note of the session id of each team member for reference.

### Team Members
- Builder
  - Name: integration-builder
  - Role: Post-merge integration wiring and fixes
  - Agent Type: builder
  - Skills: [tdd, verification-before-completion]
  - Resume: true
- Validator
  - Name: integration-validator
  - Role: Cross-component verification and end-to-end validation
  - Agent Type: validator
  - Skills: [verification-before-completion]
  - Resume: true

## Step by Step Tasks

### 1. Post-Merge Build Verification
- **Task ID**: integration-build-verify
- **Depends On**: none
- **Assigned To**: integration-validator
- **Agent Type**: validator
- **Skills**: [verification-before-completion]
- **Parallel**: false
- **Validate**: false
- Build the full project from main branch
- Verify zero compilation errors and zero warnings (or only pre-existing warnings)
- **Verify**: <full build command>
- **Expected**: Build succeeds with zero errors

### 2. Interface Contract Verification
- **Task ID**: integration-contract-verify
- **Depends On**: integration-build-verify
- **Assigned To**: integration-validator
- **Agent Type**: validator
- **Skills**: [verification-before-completion]
- **Parallel**: false
- **Validate**: false
- For each entry in Foundation Exports (from manifest), verify type exists and signature matches
- For each Interface Contract Imports entry across all component plans, verify the imported type exists at the declared path
- **Verify**: <type check / build command>
- **Expected**: All exported types exist, all imports resolve, no signature mismatches

### 3. Integration Wiring
- **Task ID**: integration-wiring
- **Depends On**: integration-contract-verify
- **Assigned To**: integration-builder
- **Agent Type**: builder
- **Skills**: [tdd, verification-before-completion]
- **Parallel**: false
- **Validate**: true
- Wire components at integration points: app entry, DI container, routing, shared state
- Verify real implementations are injected — not stubs, facades, or placeholder types
- Ensure navigation reaches real views with real data
- **Verify**: <build + run command>
- **Expected**: App builds, launches, and basic navigation works
- **Behavioral**: <user action → observable outcome crossing component boundaries>

### 4. Validate Integration Wiring
- **Task ID**: validate-integration-wiring
- **Depends On**: integration-wiring
- **Assigned To**: integration-validator
- **Agent Type**: validator
- **Skills**: [verification-before-completion]
- **Parallel**: false
- Verify `integration-wiring` task completed correctly
- Verify real implementations, not stubs
- **Functional completeness**: Verify DI container registers real implementations (not mocks), navigation routes reach real views (not placeholders), shared state is initialized with real data sources, no `// TODO` or stub patterns remain in integration points

### 5. End-to-End Flow Verification
- **Task ID**: integration-e2e-verify
- **Depends On**: validate-integration-wiring
- **Assigned To**: integration-validator
- **Agent Type**: validator
- **Skills**: [verification-before-completion]
- **Parallel**: false
- **Validate**: false
- Trace critical user flows across component boundaries
- Verify data flows end-to-end: user input → processing → persistence → display
- **Verify**: <e2e test command or manual verification steps>
- **Expected**: All critical user flows complete successfully

### 6. Final Acceptance
- **Task ID**: integration-final-acceptance
- **Depends On**: integration-e2e-verify
- **Assigned To**: integration-validator
- **Agent Type**: validator
- **Skills**: [verification-before-completion]
- **Parallel**: false
- Run all validation commands from ALL component plans
- **Functional completeness sweep**: For every UI element or endpoint created across ALL plans, verify: event handlers fire and produce effects, state updates are reflected visually, data persists where specified, navigation reaches real destinations, error states produce user feedback
- Verify all acceptance criteria from product-brief MVP definition are met
- **Expected**: All validations pass, all acceptance criteria met

## Acceptance Criteria
<from product-brief MVP definition>

## Validation Commands
<consolidated from all component plans>

## Merge Readiness Checklist
N/A — this is the final plan. After this passes, the project is complete.

## Notes
<estimated cost, open questions>
