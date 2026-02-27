---
name: review-architecture
description: "Architecture review lens for codebases with 50+ source files. Proactively use when reviewing module boundaries, dependency direction, cross-layer coupling, or pattern consistency."
version: 0.1.0
---

# Architecture Review

- **Agents:** auditor, critic

## Overview

Architecture-level review focused on structural health: boundaries, layering, coupling, and pattern consistency. This review examines relationships between modules, not individual code quality.

**Core principle:** Architecture degrades through accumulation of small boundary violations. Each violation is minor; their sum is a monolith.

## When to Use

- Reviewing codebases with 50+ source files
- Checking module boundaries and ownership
- Analyzing dependency direction and layering violations
- Detecting cross-layer coupling
- Verifying pattern consistency across the codebase
- Post-feature structural health checks

**Conditional activation:** For codebases under 50 source files, this review provides limited value. Focus on `review-code-quality` instead. If invoked on a small codebase, note the size and adjust depth accordingly.

## Review Methodology

### Four Review Dimensions

#### 1. Module Boundaries

- Each module/directory has a clear single responsibility
- Public APIs are intentional (not everything is exported)
- Internal types are not leaked across module boundaries
- Module names reflect their responsibility (not implementation details)
- No "utils" or "helpers" modules that accumulate unrelated functions

**Detection method:** List all public exports per module. If a module exports types used by more than 3 other modules, examine whether it has become a "god module."

#### 2. Dependency Direction

- Dependencies flow in one direction (top-down, outer-to-inner)
- Domain/business logic does not depend on infrastructure (DB, HTTP, UI)
- No circular dependencies between modules
- Shared types live in a common layer, not arbitrarily chosen modules
- Dependency inversion used at architectural boundaries (protocols/interfaces)

**Detection method:** For each import statement, verify it points "downward" in the layer stack. Any import pointing "upward" or "sideways" across a boundary is a potential violation.

**Layer stack (typical):**
```
UI / Presentation
     |
Application / Use Cases
     |
Domain / Business Logic
     |
Infrastructure / Data Access
```

#### 3. Cross-Layer Coupling

- UI components do not directly access data layer
- Business logic does not know about HTTP request/response shapes
- Data models are not passed through all layers (use DTOs at boundaries)
- Configuration does not leak implementation details upward
- Error types are translated at layer boundaries

**Detection method:** Search for imports that skip a layer (e.g., UI importing data access types directly). Each skip increases coupling surface.

#### 4. Pattern Consistency

- Same problem solved the same way throughout the codebase
- Error handling follows a single pattern (not mixed strategies)
- State management uses consistent approach
- API patterns are uniform (naming, parameter ordering, return types)
- Test organization mirrors source organization

**Detection method:** Identify the dominant pattern for each concern (error handling, state, API shape). Flag deviations. Not every deviation is wrong -- some may be intentional exceptions that should be documented.

## Checklist

For the codebase under review:

- [ ] Each module has a clear, singular responsibility
- [ ] No circular dependencies between modules
- [ ] Dependencies flow downward in the layer stack
- [ ] Domain logic has zero infrastructure imports
- [ ] No "god modules" (exported to 4+ consumers without clear purpose)
- [ ] Public APIs are intentionally minimal
- [ ] Same concerns handled with consistent patterns
- [ ] Error handling strategy is uniform
- [ ] Layer boundaries enforced (no layer-skipping imports)
- [ ] Shared types live in appropriate common layer

## Finding Format

```
### [SEVERITY] {boundary/module description}

**Category:** {boundaries|dependency-direction|coupling|pattern-consistency}
**Confidence:** {high|medium|low}
**Description:** {What the architectural issue is}
**Evidence:** {Import chains, module relationships, or pattern deviations with file paths}
**Impact:** {How this affects maintainability, testability, or change velocity}
**Remediation:** {Structural change suggested -- module extraction, dependency inversion, etc.}
```

Severity levels: HIGH, MEDIUM, LOW
- HIGH: Circular dependency, domain-infrastructure coupling, or pattern that blocks independent testing
- MEDIUM: Layer violation, growing god module, or inconsistent pattern
- LOW: Minor boundary smell, naming inconsistency, or missing abstraction

## Constraints

- This is a structural review -- do not duplicate `review-code-quality` findings
- For small codebases (<50 files), note that architectural patterns may not yet be established
- Do not propose wholesale rewrites -- suggest incremental boundary improvements
- Acknowledge when a pattern deviation is intentional and documented
- Flag items needing dependency analysis tooling as "requires static analysis" where appropriate
- Architecture review is opinion-heavy -- distinguish "this violates a principle" from "I would do it differently"

## Related Skills

- `review-code-quality` -- Per-file code quality (this skill covers cross-file structure)
- `review-performance` -- Performance patterns that may have architectural roots
- `drift-detection` -- Detecting when architecture has drifted from documented design

## Completion Token

When this skill completes successfully, output: `[SKILL_COMPLETE:review-architecture]`
