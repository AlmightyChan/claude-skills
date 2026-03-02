# Phasing Guide

Methodology for splitting large implementation plans into context-safe phases. The plan skill uses this guide to estimate operational cost, determine when phasing is needed, and produce checkpoints that the execute skill can resume from after context clearing.

---

## 1. Context Budget Estimation Heuristics

Estimate the operational cost of each task using these unit values. One unit approximates one atomic tool invocation and its associated context consumption.

**Operation Costs:**

| Operation Type | Cost (units) | Notes |
|---|---|---|
| Read operation | ~1 | Single file read |
| Write/Edit operation | ~2-3 | Includes re-reading for verification |
| Subagent dispatch | ~5-10 | Task tool call + context handoff + result processing |
| Builder task (typical) | ~15-25 | Exploration reads + implementation writes + verification runs |
| Validator task (typical) | ~8-12 | File reads + command execution + report generation |

These estimates are derived from observed sessions. A single PRD generation session produced 128+ Edit operations and 17 Read operations, triggering 3 compactions over 7.25 hours.

**Budget Threshold:**

- Safe budget per phase: **~200 operational units**
- This is conservative, leaving headroom for unexpected operations (retries, debugging cycles, additional reads).

These estimates are initial heuristics derived from limited sessions. After executing 3-5 plans, review actual operation counts and adjust. If phases consistently exhaust context, reduce by 25%. If phases complete with significant headroom, increase by 25%.

**Application:**

1. Sum the estimated units across all tasks in the plan.
2. If the total is within 200 units, no phasing is needed -- produce a single-phase plan.
3. If the total exceeds 200 units, split the plan into phases where each phase stays within budget.

---

## 2. Phase Boundary Rules

When splitting into phases, follow these rules to determine where boundaries fall.

**Hard Rules (never violate):**

- Keep builder/validator pairs together -- never split a task from its validation step.
- Keep directly dependent tasks within the same phase where possible.
- Each phase must be independently resumable: tasks depend only on files written to disk, not on in-memory state from a prior phase.

**Soft Rules (prefer but may bend for budget):**

- Place research and design tasks in early phases -- they produce artifacts that inform later implementation.
- Prefer splitting at natural seams: after a major component is complete and validated, before starting a new component.
- Balance phase sizes -- avoid one phase with 180 units and another with 20.

**Ordering Heuristic:**

1. Research and design tasks first (they inform everything else).
2. Core data model and shared utilities second (other components depend on them).
3. Feature implementation grouped by component (each component with its validation).
4. Integration and end-to-end validation last.

---

## 3. Checkpoint Format

At the end of each phase, produce a checkpoint block that enables the execute skill to resume after context clearing. Write the checkpoint to the plan file itself, appended after the completed phase's task list.

```markdown
## Checkpoint: Phase {N} Complete

**Plan file**: {absolute path to plan document}
**Completed phases**: {list of completed phase numbers with one-line summary each}
**Next phase**: {phase number and name}
**Files produced so far**: {list of files created or modified in prior phases}
**Resume instruction**: Reload the plan at {path}, skip to Phase {N+1}, and begin with task {first task ID in next phase}. All files listed above are on disk and should not be recreated.
```

**Rules:**

- The checkpoint must contain enough information for a fresh context to resume without reading prior conversation history.
- File paths in the checkpoint must be absolute.
- The resume instruction must reference the exact task ID to start with.

---

## 4. Re-entry Protocol

When the execute skill resumes from a checkpoint, follow this sequence.

**Steps:**

1. Read the plan file at the path specified in the checkpoint.
2. Read the checkpoint block to determine the current state.
3. Verify that every file listed in "Files produced so far" exists on disk.
4. Begin the next phase's first task as specified in the resume instruction.

**Missing File Recovery:**

- If a file from a prior phase is missing, identify which task produced it.
- Re-execute only that specific task -- not the entire prior phase.
- After recovery, continue with the next phase as normal.

**Verification Before Proceeding:**

- Do not begin a new phase until all prior-phase files are confirmed on disk.
- If multiple files are missing, recover them in the order they were originally produced.
