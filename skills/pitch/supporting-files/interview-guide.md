# Interview Guide

Methodology for conducting adaptive interviews that explore an idea's viability. The pitch skill uses this guide to systematically gather information from the user, shaping questions around the idea's domain and content gaps rather than following a fixed script.

---

## 1. Input Analysis Brief

Before generating any interview questions, analyze the user's raw input against the 5 Core Question Areas to produce an internal Input Analysis Brief. This brief is NOT shown to the user — it guides question generation.

### Part A: Coverage Map

For each Core Question Area, classify coverage from the user's input:

| Classification | Meaning | Interview Priority |
|---|---|---|
| Strong | Input clearly addresses this area with specific, actionable content | Confirm and Move |
| Partial | Input touches on this topic but lacks specifics or decisions | Targeted Probe |
| Absent | Input does not address this area at all | Deep Dive |
| Contradictory | Input contains conflicting information about this topic | Deep Dive |

### Part B: Domain Profile

Write 1-2 sentences characterizing the idea: product type, target audience, and domain-specific considerations.

**Domain categories:**
- Consumer App
- Internal Tool
- API/Service
- Convention/Process
- Infrastructure
- Creative/Content
- Skill/Agent
- Integration

### Part C: Risk and Ambiguity Signals

Flag: vague language ("maybe", "possibly", "TBD"), scope creep signals (unbounded feature lists, "and more"), unvalidated assumptions, and contradictions within the input.

### Part D: Priority Ranking

Rank the 5 Core Question Areas by interview priority:
1. **Deep Dive** — Absent or Contradictory areas, plus any high-risk signals
2. **Targeted Probe** — Partial areas with moderate risk
3. **Confirm and Move** — Strong areas with low risk

### Core Question Area → Output Section Mapping

Use this mapping for traceability between interview coverage and document output:

| Core Question Area | Maps to Output Sections |
|---|---|
| Problem | Problem, The Idea |
| Solution | Proposed Approach, The Idea |
| Alternatives | Proposed Approach (competitive context) |
| Effort vs. Impact | Scope Assessment, Viability Assessment |
| Scope | Scope Assessment |

---

## 2. Question Generation Rules

Generate questions from the Input Analysis Brief, not from a fixed script.

1. **Never ask for information the input already provides clearly.** If the input says it, confirm — don't re-ask.
2. **Frame questions from the domain's perspective.** Use domain-aware language: "For a consumer app..." or "Since this is an internal CLI tool..."
3. **For contradictions, surface both sides and ask for resolution.** "Your description says X in one place and Y in another — which is correct?"
4. **For absent critical areas, explain why it matters before asking.** "Understanding the competitive landscape matters because [reason]. What alternatives exist today?"
5. **Vary question style** across the interview: open-ended exploration, scenario-based ("What happens when..."), constraint-probing ("What are the limits on..."), and priority-forcing ("If you had to pick the top 3...").

### Fallback Questions for Thin Inputs

When the user's input is too thin for meaningful analysis (fewer than 2 Core Question Areas touched), use these track-appropriate questions as round 1, then switch to brief-derived questions after the answers enrich the analysis.

**New Idea Track — Warm-Up Questions:**
1. What problem does this solve? Who experiences it?
2. Who are the target users? (1-2 personas)
3. What is the core interaction loop or value proposition?
4. What is the proposed solution at a high level?

**Enhancement Track — Warm-Up Questions:**
1. What problem does this solve or what opportunity does it create?
2. What is the proposed solution at a high level?
3. How much effort is this worth? (S = 1-2 tasks, M = 3-6, L = 7-15, XL = 15+)
4. What should we explicitly NOT do?

After round 1 answers are collected, regenerate the Input Analysis Brief with the enriched context and proceed with brief-derived questions for all subsequent rounds.

---

## 3. Decision-Making with Confidence Levels

Every area gets a real answer. No TBDs, no placeholders.

**Rules:**

- When the user is uncertain, propose a reasonable default with explicit rationale.
- Assign a confidence level: High, Medium, or Low.
- For Medium or Low confidence, define a concrete revisit condition (not vague — "revisit later" is not acceptable).
- Record uncertain decisions in Open Questions during document generation.

**Examples:**

| Decision | Confidence | Revisit Condition |
|---|---|---|
| "Target iOS users primarily" | Medium | "Revisit if user research reveals Android-dominant audience" |
| "MVP scope is 2-week effort" | High | N/A |
| "Freemium model for monetization" | Low | "Revisit after competitive analysis in Refine stage" |

**Process:**

1. When the user is uncertain, propose a default decision based on available context.
2. State the rationale for the proposed decision.
3. Assign a confidence level.
4. If confidence is Medium or Low, define a concrete revisit condition.
5. Hold the decision and its metadata for inclusion in Open Questions during Phase 4.

---

## 4. Escape Hatch

Allow the user to end the interview early without leaving output sections empty.

**Trigger phrases:** "proceed", "enough", "good enough", "let's move on", "skip the rest", or similar intent.

**Rules:**

- When triggered, stop generating new question batches immediately.
- Run the Coverage Guarantee (Section 6) to identify gaps.
- For each uncovered Core Question Area, propose a reasonable default based on answers collected so far.
- Present proposed defaults to the user for bulk approval: "I'll fill the remaining areas with proposed defaults. Review and approve?"
- No output section is left empty — escape hatches produce defaults, not blanks.

**Process:**

1. Detect the escape trigger.
2. Stop new question batches.
3. Run Coverage Guarantee (Section 6).
4. For each gap identified, generate a proposed default using:
   - Answers from completed interview questions (primary source)
   - Reasonable domain conventions from the Domain Profile (fallback)
5. Present all proposed defaults in a single review block.
6. Apply approved defaults. Revise any the user flags.

---

## 5. Batching Rules

Group questions to reduce round-trips while maintaining coherence. Use the Input Analysis Brief's priority ranking to determine batch composition.

**Rules:**

- Batch up to 4 questions per interaction.
- Group related questions together by thematic proximity.
- Do not batch unrelated areas — Problem + Solution batch naturally; Scope + Effort vs. Impact batch naturally. Problem and Scope should not share a batch.

**Priority-Based Batching:**

| Batch Position | Content |
|---|---|
| First | Deep Dive questions (highest-priority gaps from absent/contradictory areas) |
| Middle | Mix of remaining Deep Dive + Targeted Probe questions, following conversational threads |
| Final | Coverage sweep — Confirm and Move areas + any remaining probes |

**Process:**

1. Consult the Input Analysis Brief's priority ranking.
2. Select the highest-priority uncovered areas.
3. Group areas that share a thematic connection.
4. Generate questions for each area in the group (per Section 2 rules).
5. Present the batch (max 4 questions) in a single AskUserQuestion call.
6. Process all answers, then update coverage tracking before generating the next batch.
7. Repeat until all areas are covered or an escape hatch is triggered.

---

## 6. Coverage Guarantee

After all interview batches are complete (or an escape hatch is triggered), perform a final coverage check before proceeding to document generation.

**Process:**

1. Walk the 5 Core Question Areas (Problem, Solution, Alternatives, Effort vs. Impact, Scope).
2. For each area, verify it has one of:
   - A concrete user-provided answer from the interview
   - A confirmed default with confidence level (from Section 3 or Section 4)
3. If gaps remain — areas with no user answer and no default — ask one targeted fill-in question per uncovered area.
4. If the user declines to answer a fill-in question, route the area to Open Questions with a note: "Not addressed during interview — requires follow-up."
5. **Cross-check:** Each output section (The Idea, Problem, Proposed Approach, Scope Assessment, Open Questions) should have at least one contributing Core Question Area with substantive content. Use the mapping from Section 1 to verify.

**Validation rule:** No Core Question Area should reach document generation without content or an explicit routing decision. The coverage guarantee is the last checkpoint before Phase 3 (Research Enrichment).
