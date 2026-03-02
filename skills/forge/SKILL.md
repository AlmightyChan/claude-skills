---
name: forge
version: 1.0.0
description: "Spec-driven project scaffolding. Takes an idea from discovery through spec generation, review, version planning, and task tracking."
disable-model-invocation: true
argument-hint: "[project-name]"
allowed-tools: Task, AskUserQuestion, Read, Glob, Grep
---

# Forge

Spec-driven development from idea to implementation-ready project. Eight phases: discovery, decomposition, documentation planning, scaffolding, spec generation, spec review, version planning, and task generation.

- **Agents:** writer, researcher, critic, designer, auditor

## Invocation

If `$ARGUMENTS` is empty, display this message and stop:

> Usage: `/forge <project-name>` — Creates or resumes a spec-driven project under projects/.

Validate `$ARGUMENTS`: reject values containing `../`, beginning with `/`, or containing special characters beyond letters, numbers, hyphens, and underscores. The project directory is `projects/$ARGUMENTS/` relative to the repository root.

## Session Resumability

On invocation, check for existing project state at `projects/$ARGUMENTS/` to determine where to resume.

**State detection (check in order, first match wins):**

1. No `projects/$ARGUMENTS/` directory exists → Start at Phase 1
2. `discovery-notes.md` exists, no `components.md` → Resume at Phase 2
3. `components.md` exists, no `spec-manifest.md` → Resume at Phase 3
4. `spec-manifest.md` exists, no `product-brief.md` → Resume at Phase 4
5. `product-brief.md` exists, specs/ missing or fewer files than spec-manifest.md specifies → Resume Phase 5 for remaining specs
6. All specs in spec-manifest.md have corresponding files, no `review-findings.md` → Resume at Phase 6
7. `review-findings.md` exists, no `roadmap.md` → Resume at Phase 7
8. `roadmap.md` exists, no `status.md` → Resume at Phase 8
9. `status.md` exists → Forge complete. Display completion summary and stop.

**Resume briefing:** When resuming at any phase after Phase 1, present to the user before continuing:
- Project name and one-line description (from CLAUDE.md)
- Current phase number and name
- Progress summary (e.g., "7 of 18 specs complete, 11 remaining")
- Open items count from decisions.md (MODERATE and CRITICAL counts)
- Confirm "Ready to continue?" via AskUserQuestion with options: Continue / Start Over

Reconstruct context by reading: CLAUDE.md, decisions.md, issues.md, and the relevant checkpoint files.

## Confidence Gates

All writer dispatches across all phases MUST follow this protocol for decisions.

**HIGH — Write and log.** Confident based on product brief, existing specs, and domain knowledge.
- Write the content as decided
- Log in decisions.md tagged `[HIGH]`
- No flags in response

**MODERATE — Write best-judgment, flag for review.** Reasonable basis but could go either way.
- Write best-judgment content (do NOT leave blank)
- Log in decisions.md tagged `[MODERATE]` with `OPEN QUESTION` marker
- Include `OPEN:` line in response

**LOW — Placeholder, flag as critical.** Cannot make a defensible decision; getting it wrong would cascade.
- Write placeholder `[PENDING: {question}]` in the spec
- Log in decisions.md tagged `[LOW]` with `CRITICAL` marker
- Include `CRITICAL:` line in response

**Decision log format** (each entry in decisions.md):

```
### [Phase] — [Decision subject]
**Confidence:** [HIGH/MODERATE/LOW]
**Decision:** [What was decided]
**Rationale:** [Why — one sentence]
**Status:** [Decided | OPEN QUESTION | CRITICAL]
```

**Calibration examples:**

| Signal | Confidence | Example |
|--------|-----------|---------|
| Product brief explicitly states it | HIGH | "Target audience is enterprise developers" |
| Standard industry practice | HIGH | REST API for a public-facing web service |
| Reasonable inference from brief | MODERATE | Choosing between OAuth and API keys |
| Multiple valid approaches | MODERATE | Monorepo vs polyrepo structure |
| No basis in brief, high impact | LOW | Pricing model for a SaaS product |
| Legal/regulatory implications | LOW | GDPR compliance scope |
| Contradicts another spec | LOW | Two specs assume different auth methods |

## Error Protocol

1. **Agent dispatch failure:** Retry once with the same input. If the second attempt fails, inform the user which phase and agent failed. Ask via AskUserQuestion whether to retry, skip, or abort.
2. **Malformed response:** If an agent response does not contain expected format markers (CREATED:, UPDATED:, OPEN:, CRITICAL:), re-dispatch with an explicit reminder of the required response format. If still malformed, extract what you can and inform the user.
3. **Permission denied:** Inform the user which tool permission was denied and why it is needed. Ask to approve and retry.
4. **Context compaction:** Re-read CLAUDE.md, decisions.md, issues.md, and all checkpoint files. Present a resume briefing before continuing.
5. **Never silently swallow a failure.** Every error MUST be reported to the user with the phase, agent, and nature of the failure.

## User Experience Protocol

**Orientation:** On first invocation (new project, Phase 1), before asking discovery questions, present:
1. A one-paragraph summary of what forge produces
2. The 8 phases listed with one-line descriptions each
3. Confirm "Ready to begin?" via AskUserQuestion before starting Phase 1

**Phase transitions:** Before starting each phase, display:

> --- Phase [N] of 8: [Phase Name] ---

**Progress during parallel dispatches:** Before dispatching parallel agents, announce what is happening and which agents are working. After each agent returns, report results. In Phase 5, show running progress: `[5/18] Created specs/ux/user-personas.md (45 lines)`

**User summaries:** After processing each agent response, present a plain-language summary to the user. Do NOT display raw response contract output. Example: "Created the user personas spec (45 lines). Defines 3 primary personas. Made 2 HIGH-confidence decisions. No open questions."

**Completion summary:** After Phase 8's gate passes, present:
1. File inventory grouped by type with line counts
2. Total decision count and open-item count
3. Any remaining MODERATE open questions from issues.md
4. Next steps: which spec to review first, how to begin implementation using status.md

**Backtracking:** If the user requests changes to a previous phase's output at any point:
1. Pause current work
2. Identify all downstream artifacts affected by the requested change
3. Present the scope of changes needed to the user
4. Get explicit confirmation via AskUserQuestion before making changes
5. Dispatch writer to update all affected files
6. Resume from the current phase after updates complete

## Gate Protocol

Every phase gate MUST use AskUserQuestion with at minimum these options:
- **Confirm** — Approve and proceed to the next phase
- **Request Changes** — Provide specific feedback for revision

If the user selects "Request Changes": incorporate the feedback, re-present the updated artifact, and re-confirm via AskUserQuestion. Loop until the user selects "Confirm." Do NOT proceed to the next phase without explicit "Confirm."

For compound gates (e.g., Phase 5), verify all conditions are met before presenting for confirmation.

---

## Phase 1: Discovery

**Goal:** Mutual clarity on the idea.
**Agent:** None — orchestrator converses directly with the user.

You MUST cover all four threads before proceeding to Phase 2. Each thread requires at least 2 substantive questions answered. If the user gives a vague answer, probe once more before moving on.

- **Why** — What problem does this solve? Why does it need to exist?
- **Who** — Who is this for? What are their pain points?
- **What** — What does it do at a high level? What are the key capabilities?
- **Vision** — Where does this go long-term? What does success look like?

Use AskUserQuestion for all questions. After covering all threads, summarize your complete understanding back to the user.

**Gate:** Present summary via AskUserQuestion. User confirms the picture is clear.

**Checkpoint:** After gate passes, dispatch writer to create `projects/$ARGUMENTS/discovery-notes.md` containing the full discovery summary with all four threads and the user's answers.

## Phase 2: Decomposition

**Goal:** Break the idea into product components (domains of work, not features).
**Agents:** researcher (initial breakdown) + researcher, critic, designer (parallel review)

1. Dispatch researcher with the full discovery output from discovery-notes.md. Instruct: "Produce an initial component breakdown for this product. List each component with a one-sentence description. Components are domains of work (e.g., 'Authentication', 'Analytics', 'Billing'), not individual features."
2. Dispatch all three reviewers in parallel with:
   - Full product description from discovery (thorough, not summarized)
   - The researcher's complete component list with descriptions
   - Task: "Analyze if this component breakdown is sufficient for a market-ready product"
   - **For the critic agent, include this format override:** "For this review, do NOT use your standard PASS/REVISE/DROP format. Respond using ONLY the ADD/MERGE/SPLIT/CONCERN format below."
3. All three reviewers MUST respond using this format:
```
ADD: [component] — [one sentence why]
MERGE: [A] + [B] — [one sentence why]
SPLIT: [component] into [A] and [B] — [one sentence why]
CONCERN: [one sentence]
```
4. Synthesize all reviewer feedback and adjust the component breakdown
5. Present the revised breakdown to the user via AskUserQuestion, highlighting what changed from the initial breakdown and why

**Gate:** User confirms component list is complete.

**Checkpoint:** After gate passes, dispatch writer to create `projects/$ARGUMENTS/components.md` containing the confirmed component list with descriptions.

## Phase 3: Documentation Planning

**Goal:** Map components to spec documents with dependency ordering.
**Agent:** writer

Dispatch writer with:
- Full discovery output (from discovery-notes.md)
- Confirmed component list (from components.md)
- Confidence Gates protocol (defined above)
- Instruction: "Map each component to required spec documents. For each spec, provide: name, file path, component, classification (required or contextual), and dependencies (list of other spec names that must be written first). Identify cross-component documents."

Writer MUST create `projects/$ARGUMENTS/spec-manifest.md` using this entry format:

```
### [Spec Name]
- **Path:** specs/[domain]/[filename].md
- **Component:** [component name]
- **Classification:** Required | Contextual
- **Dependencies:** [comma-separated spec names, or None]
- **Status:** Pending
```

**Response format:**
```
CREATED: spec-manifest.md ([line count] lines, [N] required docs, [M] contextual docs)
DECISIONS: [count] ([HIGH/MODERATE/LOW] breakdown)
OPEN: [one-line moderate-confidence questions, if any]
CRITICAL: [one-line low-confidence questions, if any]
```

**Gate:** If CRITICAL items exist, present them to user via AskUserQuestion first. Then present manifest summary for confirmation.

## Phase 4: Scaffolding

**Goal:** Create project directory and reference documents.
**Agent:** writer (single dispatch)

Dispatch writer with:
- Discovery output (from discovery-notes.md)
- Component list (from components.md)
- spec-manifest.md content
- All decisions accumulated so far
- Any open questions or concerns
- The CLAUDE.md template below
- Instruction: "Create the project scaffolding files. Use the CLAUDE.md template. Group the index by domain."

**CLAUDE.md template:**

````markdown
# CLAUDE.md

{Project name} — {one-line description}

## Index

### Overview
| Document | Path | Purpose | Status |
|----------|------|---------|--------|
| Product Brief | product-brief.md | Refined idea and component breakdown | Current |
| Spec Manifest | spec-manifest.md | Documentation checklist | Current |
| Decisions Log | decisions.md | All decisions with confidence tags | Current |
| Issues | issues.md | Open questions, risks, blockers | Current |

### [Domain Name] Specs
| Document | Path | Purpose | Status |
|----------|------|---------|--------|

(Repeat for each domain)

## Rules

- All documents in this project are living documents.
- When a decision conflicts with an existing spec, update the spec and log the change in decisions.md.
- After creating or modifying any document, update this index.
- Before claiming any task complete, verify against the spec-manifest.
- Reference decisions.md before making choices already decided.
````

Writer MUST create:
- `projects/$ARGUMENTS/product-brief.md` — Refined idea, audience, vision, component breakdown
- `projects/$ARGUMENTS/spec-manifest.md` — Copy from Phase 3 output
- `projects/$ARGUMENTS/decisions.md` — All decisions from Phases 1-3
- `projects/$ARGUMENTS/issues.md` — Open questions or concerns
- `projects/$ARGUMENTS/CLAUDE.md` — Project index using the template, grouped by domain
- `projects/$ARGUMENTS/specs/` — Empty directory

**Response format:**
```
CREATED: [list of files with line counts]
UPDATED: decisions.md (+[N] entries)
ISSUES: [count or None]
```

**Gate:** None (mechanical step). Proceed to Phase 5.

## Phase 5: Spec Generation

**Goal:** Write all spec documents.
**Agent:** writer (one dispatch per spec)

For each spec in spec-manifest.md, dispatch writer with:
- product-brief.md content (always included)
- The specific spec-manifest entry being written
- File paths of 1-2 related completed specs (paths only — writer reads them via Read tool)
- Decisions from decisions.md tagged to the same component as this spec, plus all cross-cutting decisions
- Confidence Gates protocol (defined above)

**Parallelization:** Read the dependency field in spec-manifest.md. Dispatch specs with no unmet dependencies simultaneously. After each parallel batch completes, determine which specs are now unblocked and dispatch the next batch.

**MANDATORY post-write actions (include in every spec dispatch — failure to perform any is a defect):**
1. Create the spec file at the path specified in spec-manifest.md
2. Report all decisions made with confidence tags in the response
3. Report any issues or concerns discovered in the response
4. Report any conflicts with existing specs in the response

**After each parallel batch returns, the orchestrator MUST:**
1. Batch-apply all reported decisions to decisions.md
2. Update CLAUDE.md index with all new spec entries
3. Update issues.md with any reported concerns
4. Update spec-manifest.md status for completed specs
5. If conflicts were reported, dispatch writer to resolve them

**Response format (per spec):**
```
CREATED: specs/[domain]/[filename].md ([line count] lines)
DECISIONS: [list with confidence tags]
ISSUES: [concerns, or None]
CONFLICTS: [conflicting specs, or None]
OPEN: [moderate-confidence questions, if any]
CRITICAL: [low-confidence questions, if any]
```

**Orchestrator handling of flags:**
- No CRITICAL → move to next spec/batch
- CRITICAL → batch all from this parallel group, present to user via AskUserQuestion. Dispatch writer to update specs with answers before proceeding.
- OPEN → accumulate. Present to user in a batch at end of phase (non-blocking).

**Gate:** All specs in spec-manifest.md have corresponding files. No unresolved CRITICAL items. All OPEN items presented to user.

## Phase 6: Spec Review

**Goal:** Gap analysis and consistency check across all specs.
**Agents:** critic + auditor (parallel)

Dispatch both with:
- product-brief.md content
- spec-manifest.md content
- Full list of generated spec file paths (paths only — reviewers read files independently)
- Instruction: "Identify missing specs, inconsistencies between specs, and coverage gaps against the manifest."
- **For the critic, include format override:** "Do NOT use PASS/REVISE/DROP. Use ONLY the format below."

Critic focus: completeness, internal consistency, cross-spec coherence.
Auditor focus: coverage gaps against manifest, technical depth, regulatory/compliance gaps.

**Response format (both reviewers):**
```
MISSING SPEC: [name] — [one sentence why]
INCONSISTENCY: [file A] vs [file B] — [one sentence description]
GAP: [file] — [one sentence what's missing]
CONCERN: [one sentence]
```

After both respond: if findings exist, dispatch writer to create missing specs (add them to spec-manifest.md), resolve inconsistencies, and fill gaps. If findings are ambiguous, present to user via AskUserQuestion.

If both reviewers return zero findings, that is a passing review — proceed.

**Checkpoint:** Dispatch writer to create `projects/$ARGUMENTS/review-findings.md` documenting all findings and their resolutions (or "Clean review — no findings" if none).

**Gate:** All findings resolved.

## Phase 7: Version Planning

**Goal:** Decide what goes in v1, v2, v3+.
**Agent:** writer

Dispatch writer with:
- product-brief.md content
- All spec summaries from CLAUDE.md index (domain-grouped table rows — NOT full spec content)
- All HIGH and MODERATE decisions from decisions.md
- Confidence Gates protocol (defined above)
- Instruction: "Propose a version breakdown. v1 = minimum viable product. v2 = high-value extensions. v3+ = long-term vision."

Writer MUST create `projects/$ARGUMENTS/roadmap.md`.

**Response format:**
```
CREATED: roadmap.md ([line count] lines, [N] versions defined)
DECISIONS: [list with confidence tags]
CRITICAL: [any version-scoping questions that need user input]
```

After response: batch-apply decisions to decisions.md, update CLAUDE.md index.

**Gate:** Present version breakdown to user via AskUserQuestion. User confirms.

## Phase 8: Task Generation

**Goal:** Create the ordered implementation task list.
**Agent:** writer

Dispatch writer with:
- roadmap.md content (version breakdown)
- spec-manifest.md content (full doc list with dependencies)
- All spec file paths with line counts
- Instruction: "Generate an ordered task list grouped by version, ordered by dependency within each version. Use the status format below. Include a Prerequisites section for non-code blockers (legal, regulatory, business, design decisions) sourced from decisions.md (MODERATE/LOW unresolved), issues.md, and review-findings.md. Omit the section if none exist."

**Status format:**

```markdown
# Status

## Current Phase: [phase name]

### Prerequisites
Non-code items that must be resolved before implementation begins. Sourced from unresolved MODERATE/LOW decisions in decisions.md, open issues in issues.md, and operational concerns in review-findings.md. Group by category (e.g., Legal, Market Research, Design, Domain Decisions). Omit this section if no prerequisites exist.

- [ ] Description — source reference (blocked-phase: [phase name or "all"])

### In Progress
- [ ] `path/to/file` — Description

### Up Next
- [ ] `path/to/file` — Description (blocked by: [dependency])

### Completed
- [x] `path/to/file` — Description

### Added During Execution
- [ ] `path/to/file` — Description (discovered during: [source])
```

Writer MUST create `projects/$ARGUMENTS/status.md` and update CLAUDE.md index.

**Response format:**
```
CREATED: status.md ([line count] lines, [N] tasks)
UPDATED: CLAUDE.md index (+1 entry)
```

**Gate:** Present task list summary to user via AskUserQuestion. User confirms.

After confirmation, display the completion summary (see User Experience Protocol).
