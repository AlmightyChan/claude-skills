# Interview Guide

Methodology for conducting structured interviews that convert template sections into filled PRD content. The refine skill uses this guide to systematically gather information from the user, leveraging existing pitch content where available.

---

## 1. Pitch Analysis Brief and Question Generation

Before generating any interview questions, analyze the pitch content against the loaded template to produce an internal Pitch Analysis Brief. This brief is NOT shown to the user — it guides question generation.

### Part A: Pitch Analysis Brief

For each template section (H2 and H3), classify coverage from the pitch:

| Classification | Meaning | Interview Priority |
|---|---|---|
| Strong | Pitch clearly addresses this section with specific, actionable content | Confirm and Move |
| Partial | Pitch touches on this topic but lacks specifics, quantification, or decisions | Targeted Probe |
| Absent | Pitch does not address this section at all | Deep Dive |
| Contradictory | Pitch contains conflicting information about this topic | Deep Dive |

**Domain Profile:** Write 2-3 sentences characterizing the project: product type (app, service, tool, platform), target audience (consumer, enterprise, internal, developer), and domain-specific considerations (regulatory, real-time, data-intensive, etc.).

**Risk and Ambiguity Signals:** Flag vague language ("maybe", "possibly", "TBD"), scope creep signals (unbounded feature lists, "and more"), unvalidated dependencies, and contradictions between sections.

**Interview Priority Ranking:** Rank all template sections by interview priority:
1. **Deep Dive** — Absent or Contradictory sections, plus any high-risk signals
2. **Targeted Probe** — Partial sections with moderate risk
3. **Confirm and Move** — Strong sections with low risk

**Coverage Granularity Rule:** For templates with 30+ extractable sections (e.g., service-definition post-conversion), classify coverage at the H2 level only. Generate interview questions for H2 topics. Probe H3-level detail only for Deep Dive sections where the subsection distinction materially affects the document quality. Confirm and Move sections use H2-level confirmation only.

### Part B: Question Generation Rules

1. **Never ask for information the pitch already provides clearly.** If the pitch says it, confirm — don't re-ask.
2. **Frame questions from the domain's perspective.** Use domain-aware language: "For a consumer mobile app..." or "Given this is an internal CLI tool..."
3. **For contradictions, surface both sides and ask for resolution.** "The pitch says X in one place and Y in another — which is correct?"
4. **For absent critical sections, explain why it matters before asking.** "This template includes a Risk Assessment section because [reason]. What are the biggest risks you see?"
5. **Vary question style** across the interview: open-ended exploration, scenario-based ("What happens when..."), constraint-probing ("What are the limits on..."), and priority-forcing ("If you had to pick the top 3...").

**Table-format sections:** For sections requiring tabular output (requirements tables, risk registers, stakeholder matrices, RACI matrices, budget tables), derive questions that elicit individual rows. Ask for 3-5 representative items with their column values, then offer to draft additional rows based on context. Example: For a Risk Assessment table, ask "What are the 3 biggest risks? For each, estimate the likelihood (High/Medium/Low) and impact."

**Process:**

1. Read the template section heading and its guidance text.
2. Consult the Pitch Analysis Brief for this section's classification and priority.
3. For Strong sections: formulate a confirmation question referencing the pitch content.
4. For Partial sections: formulate 1-2 targeted questions addressing the specific gaps.
5. For Absent sections: formulate 1-2 questions with domain context explaining why the section matters.
6. For Contradictory sections: surface the contradiction and ask for resolution.
7. For tabular sections: formulate questions that elicit representative rows with column values.
8. Verify each question is answerable in 2-4 sentences (not unbounded).

---

## 1.5 Behavioral Requirements

When gathering functional requirements, prompt users for Given-When-Then format.

**Scripted question:** "For each key requirement, can you describe the expected behavior as: Given [some context], When [the user does something], Then [what should happen]?"

**Example:**
> Given a logged-in user on the dashboard, When they click "Create New", Then a modal appears with a blank form and the cursor in the first field.

If the user provides natural language instead of G-W-T format, derive the Given-When-Then structure from their description and present it for confirmation: "Based on what you described, I'd capture this as: Given [X], When [Y], Then [Z] -- does that match your intent?"

---

## 2. Pitch Reference Pattern

Use existing pitch content to accelerate the interview. Avoid re-asking what the pitch already answers.

**Rules:**

- For each template section, check if the pitch document contains relevant content.
- If pitch content exists: present it and ask "The pitch says [X] about this -- do you want to expand, revise, or keep it as-is?"
- If no pitch content exists: ask the derived question directly.

**Pitch-to-Template Section Mapping:**

These pitch sections are produced by the `/pitch` skill. Match them to template sections when scanning for existing content.

| Pitch Section | Template Sections |
|---|---|
| The Idea | Executive Summary, Overview, Purpose |
| Problem | Problem Statement, Background, Motivation |
| Proposed Approach | Technical Approach, Implementation, Design |
| Scope Assessment | Out of Scope, Constraints, Scope |
| Viability Assessment | Risk Assessment, Feasibility, Assumptions |
| Open Questions | Open Questions, Assumptions, Known Unknowns |

**Sections with no pitch coverage:** Success Metrics, Acceptance Criteria, Dependencies, Technical Design, Resource Requirements, Milestones, User Personas, and Non-Functional Requirements have no corresponding pitch section. These will always require fresh interview questions.

**Process:**

1. Before asking a question, scan the pitch for content matching the current template section.
2. If a match is found, quote the relevant pitch content and offer the expand/revise/keep prompt.
3. If no match is found, fall through to the derived question from Section 1.
4. Record the user's response regardless of path taken.

**Domain-Aware Follow-Ups:**

When pitch content exists for a section, follow-ups should be domain-informed rather than generic:

> "The pitch says {X}. For a {domain type} project, I would also want to understand {Y} — do you want to expand to cover that, or is {X} sufficient?"

This pattern applies the Domain Profile from the Pitch Analysis Brief to generate follow-ups that surface domain-specific gaps the user may not have considered.

---

## 3. Decision-Making Rules

Every section gets a real decision. No TBDs, no placeholders, no "to be determined."

**Rules:**

- For uncertain areas: propose a reasonable initial decision with explicit rationale.
- Flag uncertain decisions with confidence level: "Decision confidence: High/Medium/Low."
- Add uncertain decisions to the template's assumptions, risk assessment, or open questions section -- whichever is present and most appropriate for the confidence level.
- Revisit conditions must be specific and testable, not vague ("revisit later" is not acceptable).

**Examples:**

| Decision | Confidence | Revisit Condition |
|---|---|---|
| "Use Supabase Auth for authentication" | Medium | "Revisit if auth requirements exceed basic magic link flow" |
| "Target iOS 17+ only" | High | N/A |
| "Use REST over GraphQL for API layer" | Low | "Revisit after API surface area is fully scoped in design phase" |

**Process:**

1. When the user is uncertain, propose a default decision based on available context.
2. State the rationale for the proposed decision.
3. Assign a confidence level.
4. If confidence is Medium or Low, define a concrete revisit condition.
5. Record the decision and its metadata in the appropriate template section.

---

## 4. Escape Hatches

Allow the user to end the interview early without leaving sections empty.

**Trigger phrases:** "proceed", "enough", "good enough", "let's move on", "skip the rest", or similar intent.

**Rules:**

- When triggered, stop asking questions immediately.
- For remaining unfilled sections: propose reasonable defaults based on pitch content and answers collected so far.
- Present proposed defaults to the user for bulk approval: "I'll fill the remaining N sections with proposed defaults. Review and approve?"
- No section is left empty -- escape hatches produce defaults, not blanks.

**Process:**

1. Detect the escape trigger.
2. Identify all remaining unfilled template sections.
3. For each unfilled section, generate a proposed default using:
   - Pitch content (primary source)
   - Answers from completed interview questions (secondary source)
   - Reasonable domain conventions (fallback)
4. Present all proposed defaults in a single review block.
5. Apply approved defaults. Revise any the user flags.

---

## 5. Batching Rules

Group questions to reduce round-trips while maintaining coherence. Use the Pitch Analysis Brief's priority ranking to determine batch composition.

**Rules:**

- Batch up to 4 questions per interaction.
- Group related questions together (e.g., all questions for the same template section area).
- Do not batch unrelated questions -- "User Personas" should not share a batch with "Deployment Plan."

**Priority-Based Batching:**

| Batch Position | Content |
|---|---|
| First | Deep Dive questions (highest-priority gaps from absent/contradictory sections) |
| Middle | Mix of remaining Deep Dive + Targeted Probe questions, following conversational threads |
| Final | Coverage sweep — Confirm and Move sections + any remaining probes |

Use the template's numeric prefix as a clustering signal — sections with the same integer prefix (e.g., 2.1, 2.2, 2.3) are natural batch candidates.

**Process:**

1. Consult the Pitch Analysis Brief's priority ranking.
2. Select the highest-priority unfilled sections.
3. Group sections that share a thematic area or numeric prefix.
4. Generate questions for each section in the group (per Section 1 rules).
5. Present the batch (max 4 questions) in a single interaction.
6. Process all answers, then update coverage tracking before generating the next batch.
7. Repeat until all sections are covered or an escape hatch is triggered.

---

## 6. Success Criteria Derivation

After all functional requirements are gathered, derive testable success criteria from the Given-When-Then behavioral specifications.

**Process:**

1. For each MUST-priority requirement with a G-W-T behavior, create a success criterion that verifies the behavior.
2. For each success criterion, identify a verify command and expected output.
3. Present the derived criteria table to the user for confirmation before including in the document.

**Example table:**

| ID | Criterion | Verify Command | Expected Output |
|----|-----------|----------------|-----------------|
| SC-1 | User can log in with valid credentials | `pytest tests/test_auth.py::test_login_success -v` | "1 passed" |
| SC-2 | Invalid credentials show error message | `pytest tests/test_auth.py::test_login_invalid -v` | "1 passed", error message contains "Invalid" |
| SC-3 | Session persists across page reload | `npx playwright test tests/e2e/session.spec.ts` | "1 passed", cookie present after reload |

Criteria that cannot be automated should note "Manual verification required" in the Verify Command column with a description of the manual test procedure.

---

## 7. Coverage Guarantee

After all interview batches are complete (or an escape hatch is triggered), perform a final coverage check before proceeding to document generation.

**Process:**

1. Walk the complete template section list (all H2 and H3 headings extracted in Phase 1).
2. For each section, verify it has one of:
   - A concrete user-provided answer from the interview
   - Confirmed pitch content (user approved via Section 2 pattern)
   - An escape-hatch default with confidence level (from Section 4)
3. If gaps remain — sections with no user answer, no pitch content, and no default — ask one targeted fill-in question per uncovered section.
4. If the user declines to answer a fill-in question, route the section to the document's Open Questions with a note: "Not addressed during interview — requires follow-up."

**Validation rule:** No section should reach document generation without content or an explicit routing decision. The coverage guarantee is the last checkpoint before Phase 4.
