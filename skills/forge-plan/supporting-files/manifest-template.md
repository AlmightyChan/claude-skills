# Forge Plan Manifest: <Project Name>

## Source Project
- **Project:** projects/<name>/
- **Product Brief:** projects/<name>/product-brief.md
- **Components:** <N> components in <M> plan groups
- **Specs:** <N> specs across <M> domains
- **Target Version:** v1 (MVP)

## Project Topology
<same format as plan template>

## Dependency DAG
<component dependency graph — which plans depend on which>

### Foundation Plan
<If no component qualifies as foundation (none depended on by 2+), omit this subsection and start tiers at Tier 1.>
- **Plan:** plans/<slug>.md
- **Components:** <list>
- **Rationale:** Depended on by <N> other plans

### Parallelism Tiers
- **Tier 0 (foundation):** <foundation plan> — must complete first
- **Tier 1:** <plans> — can run in parallel after foundation merges
- **Tier 2:** <plans> — after tier 1 merges
- **Integration:** plans/integration.md — after all tiers merge

### Critical Path
<longest sequential chain with hop count>

## Component Plans

| Plan | Components | Tasks | Est. Units | Tier | Branch |
|------|-----------|-------|-----------|------|--------|
| plans/<slug>.md | <list> | <N>B + <N>V | ~<N> | 0 | feat/<slug> |
| plans/<slug>.md | <list> | <N>B + <N>V | ~<N> | 1 | feat/<slug> |

## Integration Plan

| Plan | Tasks | Est. Units | Depends On |
|------|-------|-----------|------------|
| plans/integration.md | <N> | ~<N> | All component plans |

## Execution Strategy
- **Parallelism Mode:** <topology | aggressive>
- **Worktree Cap:** WORKTREE_CAP concurrent (system cap is WORKTREE_CAP + 1 including main; tiers exceeding cap are batched)
- **Merge Order:** Foundation first, then tiers in order, then integration
- **Merge Protocol:** Build verification after each merge. Conflicts escalate to user.
- **Rollback:** If merge fails, revert merge, re-execute plan against current main, retry.

## High-Conflict Files
<files touched by multiple plans — for merge awareness>

| File | Plans Touching It | Risk |
|------|------------------|------|
| <path> | <plan-a>, <plan-b> | <merge/overwrite> |

## Foundation Exports
<shared types/APIs from foundation plan — other plans import these>

## Estimated Cost
- Component plans: ~<N> units total across <M> plans
- Integration plan: ~<N> units
- Grand total: ~<N> units across <M> parallel streams

## Open Questions
<from decisions.md — MODERATE items that may affect execution>

Next step: Run `/forge-execute <project-name>` to begin execution.
