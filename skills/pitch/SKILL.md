---
name: pitch
description: "Use when shaping a new idea, assessing viability, or scoping an enhancement for an existing project"
argument-hint: [idea description or ideas-log reference]
hooks:
  Stop:
    - hooks:
        - type: command
          command: >-
            uv run
            $HOME/.claude/hooks/validators/validate_file_contains.py
            --directory docs/pitch --extension .md --prefix pitch-
            --contains '## The Idea'
            --contains '## Problem'
            --contains '## Proposed Approach'
            --contains '## Scope Assessment'
            --contains '## Viability Assessment'
            --contains '## Open Questions'
version: 0.1.0
---

# Pitch

Shape a new idea into a lightweight viability assessment. Follow the phases below in order.

## Variables

```
ARGUMENTS: $ARGUMENTS
OUTPUT_DIR: docs/pitch/
IDEAS_LOG: docs/logs/ideas-log.md
INTERVIEW_GUIDE: ~/.claude/skills/pitch/supporting-files/interview-guide.md
```

## No-Go Directives

These rules are absolute. Do not violate them under any circumstances.

- Do NOT automatically apply output from any subagent (critic, researcher, or other) to the document. Always present findings to the user and get explicit approval before writing changes.
- Do NOT produce PRD-like output. The pitch is a lightweight assessment, not a specification.
- Do NOT impose hard round limits on the interview. Let thoroughness drive the conversation, not a counter.
- Do NOT recommend whether to proceed or kill the idea. Provide the viability score and rationale; the user decides.
- Do NOT try to split ideas. Flag broad scope in the Scope Assessment section, but leave splitting to the user.
- Do NOT include Refine-stage content: success criteria, technical specifications, implementation details, or formal scope boundaries.
- Do NOT conduct design vision gathering during the interview.
- Do NOT hardcode interview questions or ask the same questions for every project — derive questions from the Input Analysis Brief, shaped by the idea's domain, audience, and content gaps, via the interview guide.
- Do NOT write code, deploy agents beyond the specified subagent dispatches, or produce any output beyond the pitch document and chat report.

---

## Phase 1 — Input Parsing

Parse `$ARGUMENTS` to determine the input mode.

**If `$ARGUMENTS` is empty:** Display the following usage help and stop.

```
Usage: /pitch [idea description or ideas-log reference]

Examples:
  /pitch a mobile app for tracking reading habits
  /pitch reading tracker           (matches ideas-log entry by keyword)
  /pitch Auto-Archive Pipeline     (matches ideas-log entry by title)
```

**If `$ARGUMENTS` is not empty:** Determine whether the input is a free-text idea or an ideas-log reference.

1. Read `docs/logs/ideas-log.md`.
2. Search for entries whose title contains any of the argument words (case-insensitive). A match requires at least two words overlapping with the entry title, or an exact substring match.
3. Based on results:
   - **No matches found and input is short (fewer than 5 words):** Display: "No matching idea found in ideas-log.md. Please provide the idea description directly or verify the reference." Stop and wait for user input.
   - **No matches found and input is descriptive (5+ words):** Treat the full argument as a free-text idea description. Proceed to Phase 2.
   - **Exactly one match:** Extract the entry's description as the idea description. Confirm with the user: "Found idea from ideas-log: '{title}'. Using this as the basis for the pitch." Proceed to Phase 2.
   - **Multiple matches:** Use AskUserQuestion to present the matching entries as options (up to 4), each with the entry title as the label and the date as the description. Wait for the user's response before proceeding.
4. Store the resolved idea description for use in subsequent phases.

---

## Phase 2 — Interview

Read the interview guide at `$INTERVIEW_GUIDE` before starting the interview. The guide defines the adaptive methodology — this section defines what to do at each step.

### Anti-Hallucination Rule

All document content MUST trace back to user-provided answers, researcher findings, or critic assessment. Do not infer, assume, or fabricate information the user did not provide. If a question area was not adequately covered, capture it in Open Questions — do not fill the gap with plausible-sounding content. "I don't know" is better than a confident guess.

### Idea Type Determination

After input parsing, determine the idea type:
- If the idea references an existing system, codebase, or feature (keywords: "improve", "add to", "extend", "fix", "enhance", "modify", "update", or references a known project) → **Enhancement Track**
- Otherwise → **New Idea Track**
- If ambiguous, ask via AskUserQuestion: "Is this a new standalone idea or an enhancement to an existing system?"

### Input Analysis Brief

After determining idea type, generate an internal Input Analysis Brief per interview guide Section 1. Analyze the user's raw input against the 5 Core Question Areas to produce:

- **Coverage Map**: Classify each Core Question Area as Strong / Partial / Absent / Contradictory
- **Domain Profile**: 1-2 sentences — product type, target audience, domain considerations
- **Risk/Ambiguity Signals**: Vague language, scope creep, contradictions, unvalidated assumptions
- **Priority Ranking**: Rank Core Question Areas as Deep Dive / Targeted Probe / Confirm and Move

The brief is internal — do not show it to the user.

### Core Question Areas

These 5 areas define what must be covered during the interview. They are the coverage checklist, not the question source — questions are derived from the Input Analysis Brief.

| Core Question Area | What must be covered | Maps to Output Sections |
|---|---|---|
| Problem | What problem does this solve? Who experiences it? How significant? | Problem, The Idea |
| Solution | What is the proposed solution at a high level? What makes it distinct? | Proposed Approach, The Idea |
| Alternatives | How does this compare to what exists? Why is a new approach needed? | Proposed Approach (competitive context) |
| Effort vs. Impact | Expected effort? Expected payoff? Is the ratio favorable? | Scope Assessment, Viability Assessment |
| Scope | Is this one idea or several? What are the natural boundaries? | Scope Assessment |

### Adaptive Interview Process

1. **First batch**: Generate questions from the Deep Dive and high-priority Targeted Probe areas, shaped by the domain profile. Use the interview guide's batching rules (Section 5) for grouping.

2. **Thin-input fallback**: If the user's input is too thin for meaningful analysis (fewer than 2 Core Question Areas touched), use the track-appropriate warm-up questions from the interview guide (Section 2 fallback questions) as round 1. After round 1 answers are collected, regenerate the Input Analysis Brief with enriched context and proceed with brief-derived questions for all subsequent rounds.

3. **Subsequent batches**: After each batch, update coverage tracking against the 5 Core Question Areas. Generate the next batch from the highest-priority uncovered areas, adapting question framing based on answers received so far.

4. **Decision-making under uncertainty**: When the user is uncertain, follow the interview guide's confidence-level protocol (Section 3) — propose a default, assign confidence, define revisit conditions.

5. **Escape hatch**: If the user signals they want to move on (interview guide Section 4), stop new batches and proceed to the coverage guarantee.

6. **Coverage guarantee**: Before proceeding to Phase 3, run the coverage guarantee (interview guide Section 6). All 5 Core Question Areas must have substantive answers or explicit routing to Open Questions.

### Interview Behavior

- Use AskUserQuestion for ALL interview questions. Batch up to 4 related questions per AskUserQuestion call. Each question should have 2-4 concrete options based on common answers for the idea type, plus the automatic "Other" option for free-text responses.
- Be conversational and adaptive. Follow threads that matter for this specific idea.
- Simple ideas with clear scope get brief interviews (1-2 exchanges). Complex or gap-filled ideas get deeper exploration.
- If the user provides only minimal responses (single words, vague answers) for a core question area, note the gap rather than inferring an answer. Capture it in Open Questions during Phase 4.

### Scope Boundary

"Whether to build it" is in scope. "How to build it" is out of scope. If a thread drifts toward implementation details, redirect to the refinement stage.

---

## Phase 3 — Research Enrichment

After the interview concludes, dispatch a researcher subagent for competitive and market context.

### Researcher Dispatch

Use the Task tool to dispatch a researcher subagent:

```
Task({
  description: "Research competitive context for pitch: {idea name}",
  prompt: "Pipeline context gathering for pitch.\n\nIdea: {idea description from interview}\n\nFollow your Deep Context Gathering (Pipeline Mode) protocol with emphasis on categories 1 (competing products), 3 (technical landscape), and 4 (community sentiment). Categories 2 (design precedents) and 5 (relevant repos) are optional — include only if directly relevant to viability. Keep output concise — this is a lightweight pitch, not exhaustive research.",
  subagent_type: "researcher"
})
```

### After Researcher Returns

- **If direct competition found:** Ask the user how their idea differs (use AskUserQuestion with 2-3 differentiation options). Hold the answer for use in Phase 4's Proposed Approach section.
- **If no direct competition:** Proceed to Phase 4.

Hold detailed findings in context to enrich the pitch document. Do not dump raw research on the user.

### Failure Handling

If the researcher fails or times out, proceed to Phase 4. Add to Open Questions: "**Research gap**: Competitive/market context could not be gathered. Consider manual research before Refine."

---

## Phase 4 — Document Generation

Generate the pitch document.

### Filename

Create a kebab-case filename from the idea name:
- Lowercase, hyphens only, no special characters
- Must match `^[a-z0-9-]+$`
- Truncate to 50 characters if needed
- Result: `docs/pitch/pitch-{name}.md`

Create the `docs/pitch/` directory if it does not exist.

### Document Template

Write the pitch document using exactly this structure — no more sections, no fewer.

```markdown
# Pitch: {Name}

## The Idea
{Clear, concise description of what this is.}

## Problem
{Who has this problem and why it matters.}

## Proposed Approach
{How the idea seeks to solve the problem. High-level, not technical.}

## Scope Assessment
{Is this one idea or multiple? Is the scope appropriate?
 Flags if the idea is broad enough to justify splitting, with reasoning.}

## Viability Assessment
**Score: [PENDING CRITIC REVIEW]**

## Open Questions
{Anything unresolved. What needs to be answered before refinement.}
```

### Voice

Write in a neutral, analytical tone. State what the idea does, what problem it addresses, and what gaps remain. Do not advocate for or against the idea. Avoid promotional language ("revolutionary", "game-changing") and dismissive language ("merely", "just another").

### Source Mapping

| Section | Primary Source | Secondary Source |
|---|---|---|
| The Idea | Interview: all areas (synthesize) | Research: category/niche positioning |
| Problem | Interview: Problem | Research: community sentiment validating the problem |
| Proposed Approach | Interview: Solution | Research: competing products, technical landscape; differentiation answer |
| Scope Assessment | Interview: Scope, Effort vs. Impact | Research: landscape breadth signals |
| Open Questions | Interview: unexplored areas, user uncertainty | Research: unknowns surfaced by researcher |

### Content Requirements

- **The Idea**: Synthesize the interview into one clear paragraph. If research revealed the idea occupies a known category or niche, reflect that positioning.
- **Problem**: Ground in real needs from the interview. If research found community sentiment validating the problem, cite it. Do not speculate beyond what the user described or research confirmed.
- **Proposed Approach**: High-level solution direction. Incorporate relevant competitive context. If the user provided a differentiation answer during competition handling, state it here.
- **Scope Assessment**: If the idea spans distinct standalone concerns, flag it. If research revealed the landscape is broader than the interview suggested, note the scope implications. Do not split.
- **Viability Assessment**: Write the placeholder exactly as shown. Phase 5 will replace it.
- **Open Questions**: List unresolved items from the interview. Add unknowns surfaced by research. Include any research gap, assessment gap, or thin-section notes from early exit. Include any decisions made with Medium or Low confidence during the interview. Format: "{Decision} (Confidence: {Level}. Revisit: {condition})."

Every section except Viability Assessment must contain substantive content. No placeholders or boilerplate.

### Pre-Write Quality Gate

Before writing the file, verify ALL of the following. If any check fails, fix it before writing.

- [ ] No TBDs, placeholders, or template variables remain
- [ ] Every claim in Problem traces to a user answer or research finding — no speculative problems
- [ ] Proposed Approach is specific enough that `/refine` can derive requirements from it
- [ ] Scope Assessment addresses whether this is one idea or many
- [ ] Open Questions captures every unresolved area — nothing was silently filled in
- [ ] No section contains content the user did not provide or approve

---

## Phase 5 — Viability Assessment (Critic Review)

Dispatch a critic subagent to evaluate the idea's viability.

### Critic Dispatch

Use the Task tool to dispatch a critic subagent:

```
Task({
  description: "Viability assessment for pitch: {idea name}",
  prompt: "Read docs/pitch/pitch-{name}.md. Evaluate on three dimensions only (not your full review workflow):\n\n1. **Effort vs. impact** — Payoff vs. investment, given scope and problem significance\n2. **Differentiation strength** — Does the idea offer something meaningfully distinct from existing approaches? Is the differentiation defensible?\n3. **Problem clarity** — Problem well-defined, solution well-targeted, ambiguities?\n\nScore anchors (use the full range):\n- 8-10: Strong case to proceed. Problem is clear, approach is sound, gaps are minor.\n- 5-7: Proceed with caution. Material gaps or risks that need attention during refinement.\n- 1-4: Weak case. Fundamental gaps in problem definition, approach, or justification.\n\nRespond in this exact format:\n\n**Score: {N}/10**\n\n**Effort vs. Impact**: {2-3 sentences}\n**Differentiation Strength**: {2-3 sentences}\n**Problem Clarity**: {2-3 sentences}\n\n**Key Gaps**: {Bullet list of gaps and risks}\n\nProvide score and rationale only — no proceed/kill recommendation.",
  subagent_type: "critic"
})
```

### User Approval Gate

Present the critic's full assessment (score, dimensional rationale, key gaps) in chat.

If the critic's Key Gaps list contains 2 or more items, present gaps to the user in rounds of up to 4 per AskUserQuestion call. Each gap is a separate question within the call. For each question:
   - **Question text**: Include: (a) the gap title, (b) a detailed explanation of what the gap is and why the critic flagged it, (c) the potential impact if the gap is not addressed.
   - **Options** (same for every question):
     1. "Approve" — include this gap in the viability assessment
     2. "Deny" — omit it from the assessment
     3. "Revise" — let the user modify the gap description before including
   - If more than 4 gaps exist, continue with additional rounds until all gaps have been reviewed.
   - After each round, for any "Revise" selections, ask the user for their modified version before proceeding to the next round.

If the critic's Key Gaps list contains 0-1 items, use AskUserQuestion: "Would you like to apply this viability assessment to the pitch document?" Options: "Apply as-is (Recommended)" / "Adjust before applying".

**After gap review:** Use the Edit tool to find `**Score: [PENDING CRITIC REVIEW]**` and replace with the critic's full assessment. Include only the user-approved gaps in the Key Gaps section. Omit excluded gaps.

**If the user wants changes:** Incorporate their feedback, then write the modified version.

**If the Edit fails:** Read the file, locate `## Viability Assessment`, and use the existing section text as old_string.

### Critic Failure Handling

If the critic fails or times out, inform the user and update Viability Assessment to:

**Score: Not assessed** — Critic review could not be completed. Consider manual assessment before Refine.

Add to Open Questions: "**Assessment gap**: Automated viability scoring was not completed."

---

## Phase 6 — Report

Output a chat summary. This is conversational output only — no file is produced.

```
Pitch Complete

File: docs/pitch/pitch-{name}.md
Idea: {idea name}
Viability Score: {N}/10
Key Gaps (up to 5, most critical first):
- {gap}
- ...
```

Merge gaps from both the Viability Assessment's Key Gaps and the Open Questions section. Deduplicate items describing the same concern. Prioritize gaps that would block refinement over informational gaps.

If the viability score is "Not assessed" due to critic failure, display that instead of a number.

---

## Phase 7 — Smart Routing

Determine the recommended next step using signals from the completed pitch.

### Routing Logic

Read the viability score from the Viability Assessment section. Apply the first matching rule:

| # | Condition | Recommend | Rationale |
|---|-----------|-----------|-----------|
| 1 | Viability score < 5 or "Not assessed" | Review the pitch | Low viability or missing assessment — reconsider before investing further |
| 2 | Otherwise | `/refine` | Default pipeline path — refine adds structure before planning |

### Routing Context

When presenting the recommendation, include relevant context from the pitch:

- If the Scope Assessment flags the idea as broad or multi-part, append: "Note: Scope was flagged as potentially broad. Consider whether to split before refining."
- If the Open Questions section has 3+ substantive items (excluding gap notes), append: "Note: {N} open questions remain — these will need resolution during refinement."

### Presentation

Use AskUserQuestion with the question: "What would you like to do next?" Present options based on the routing recommendation:

**If recommendation is "Review the pitch":**
1. "Review and revise the pitch (Recommended)" — description: "The viability score suggests revisiting the idea before investing further"
2. "Run /refine anyway" — description: "Proceed to formal requirements despite low viability"
3. "Done for now" — description: "End here"

**If recommendation is `/refine`:**
1. "Run /refine now (Recommended)" — description: "Generate formal requirements from this pitch"
2. "Review the pitch first" — description: "Read through the document before continuing"
3. "Done for now" — description: "End here"

### Execution

- **If the user selects any /refine option:** Execute: `Skill({ skill: "refine", args: "docs/pitch/pitch-{name}.md" })`
- **If the user selects "Review..." or "Done for now":** Output the file path and end.

## Stage Handoff

When this skill completes, produce a structured handoff for the next stage. This handoff is part of the Report phase output — include it in the chat summary.

**Schema:**
- **Output file:** `docs/pitch/pitch-{name}.md`
- **Viability score:** {N}/10 (or "Not assessed")
- **Routing recommendation:** {/refine | review the pitch}
- **Key decisions:** 3-5 bullet points of decisions made during the interview and why
- **Constraints established:** Any constraints the next stage must respect (scope boundaries, no-gos, effort budget)
- **Open questions:** Unresolved items for the next stage
- **Scope:** Problem domain and solution direction established

## Related Skills

- `refine` — Next stage: generates formal requirements from a pitch

## Completion Token

Output `[SKILL_COMPLETE:pitch]` after:
- The user selects "Done for now" or "Review the pitch first" and the file path is displayed.
- The user selects a /refine option — output the token BEFORE invoking the Skill tool for refine.

The token confirms the pitch artifact was produced. It fires regardless of the routing choice.
