# Provisioning Templates

Canonical templates used by the plan skill to populate the `## Provisioning` section of plan documents. The plan skill reads these templates and fills them with project-specific data (companion document paths, project name, project directory).

---

## Project CLAUDE.md Template

```markdown
# {Project Name}

## Reference Documents
{For each companion document found during planning:}
- `{path}` — {doc type}: {brief description of what this document covers}

## Living-File Rules
- When a design decision changes during execution, update the relevant reference document (PRD, Brand OS, spec) to reflect the new decision.
- When adding a new file or module not covered by existing docs, note it in this file under a `## New Components` section.
- Do not silently diverge from specs — either update the spec or flag the divergence.
```

---

## Decision Log Template

```markdown
# Decision Log

Tracks decisions that diverge from or extend the original plan and specs. Written by the execute orchestrator only.

## Entry Format

## {YYYY-MM-DD} — {task-id}: {brief decision summary}
**Context:** {what prompted the decision — gap-flag text, user clarification, or implementation discovery}
**Decision:** {what was decided and by whom}
**Impact:** {which files, features, or specs are affected}

---
```

---

## Usage

The plan skill populates the `## Provisioning` section as follows:
1. Read these templates
2. Replace `{Project Name}` with the project name from `## Project Directory`
3. Replace `{project-dir}` with the project directory path
4. Fill `## Reference Documents` with companion documents found during planning
5. Include the Decision Log template verbatim (the execute skill creates the file)
