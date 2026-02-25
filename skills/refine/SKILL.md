---
name: refine
description: "Use when you have a pitch file that needs formal requirements, specification, or technical documentation"
argument-hint: <path-to-pitch-file>
hooks:
  Stop:
    - hooks:
        - type: command
          command: >-
            bash -c '
            uv run $HOME/.claude/hooks/validators/validate_file_contains.py
            -d docs/prd -e .md --prefix prd- --max-age 60
            --contains "## 1." --contains "Status:" 2>/dev/null
            || uv run $HOME/.claude/hooks/validators/validate_file_contains.py
            -d docs/specs -e .md --max-age 60
            --contains "## 1." --contains "Status:" 2>/dev/null
            || uv run $HOME/.claude/hooks/validators/validate_file_contains.py
            -d docs/defects -e .md --prefix defect- --max-age 60
            --contains "## 1." 2>/dev/null
            || uv run $HOME/.claude/hooks/validators/validate_file_contains.py
            -d docs/findings -e .md --max-age 60
            --contains "## 1." 2>/dev/null'
version: 0.1.0
---

# Refine

Transform a pitch document into a complete technical document. Execute phases sequentially.

## Variables

```
ARGUMENTS: $ARGUMENTS
TEMPLATES_DIR: ~/.claude/skills/refine/templates/
INTERVIEW_GUIDE: ~/.claude/skills/refine/supporting-files/interview-guide.md
```

## No-Go Directives

- Do NOT embed templates inline -- always read from `~/.claude/skills/refine/templates/` at runtime.
- Do NOT hardcode interview questions or ask the same questions for every project -- derive questions from the Pitch Analysis Brief, shaped by the project's domain, audience, and content gaps, via the interview guide.
- Do NOT allow empty sections, TBDs, TODOs, or placeholder text in the output. N/A is acceptable only with explanation.
- Do NOT fill sections without showing the user the proposed content and getting approval.
- Do NOT fabricate, assume, or hallucinate content. Every requirement, constraint, and decision MUST trace back to a user-provided answer, pitch file content, or researcher findings. If information is missing, ask for it or mark it as an Open Question — never guess.
- Do NOT write code or deploy agents beyond those specified in Phases 3 and 5.

---

## Template Index

Use this routing table to classify pitch content and determine output parameters (section headings extracted from template files at runtime).

| Template | File | Output | Filename | Signals | Research Sections | Pri | Est. Duration |
|---|---|---|---|---|---|---|---|
| PRD | `prd-template.md` | `docs/prd/` | `prd-{name}.md` | new app, new product, greenfield, from scratch, broad market/business scope | S3 Market Analysis (all), S17 Legal & Regulatory | P1 | ~40 min |
| Brand OS | `brand-os-template.md` | `docs/specs/` | `brand-os-{name}.md` | brand identity, visual identity, design language, brand system, look and feel, brand expression | Competitive visual positioning, design trends | P1 | ~30 min |
| Feature/Enhancement | `feature-spec-template.md` | `docs/specs/` | `feature-{name}.md` | new feature, add feature, improve, enhance, upgrade, refs existing system | Competitive approaches for feature area | P1 | ~35 min |
| Defect Report | `defect-report-template.md` | `docs/defects/` | `defect-{name}.md` | bug, defect, broken, fix, regression, error, refs specific failure | None | P1 | ~20 min |
| Service Definition | `service-definition-template.md` | `docs/specs/` | `service-{name}.md` | new service, API, microservice, backend service, endpoint | Framework/API availability | P2 | ~45 min |
| System Requirements | `system-requirements-doc-template.md` | `docs/specs/` | `srd-{name}.md` | internal system, platform, infrastructure system, system requirements | Regulatory, compliance, standards | P2 | ~30 min |
| Architecture Proposal | `architecture-proposal-template.md` | `docs/specs/` | `architecture-{name}.md` | re-architecture, migration, platform change, system design, architecture | Prior art, tech evaluations | P2 | ~40 min |
| Operating Model | `operating-model-design-template.md` | `docs/specs/` | `operating-model-{name}.md` | workflow change, process design, org change, operating model, team structure | Industry best practices, org design | P2 | ~35 min |
| Program Charter | `program-charter-template.md` | `docs/specs/` | `charter-{name}.md` | cross-team, program, initiative, multi-project, portfolio | Strategic context, market benchmarks | P2 | ~35 min |
| Experiment Proposal | `experiment-proposal-template.md` | `docs/specs/` | `experiment-{name}.md` | experiment, pilot, A/B test, hypothesis, test, validate assumption | Competitive benchmarks, stat methodology | P2 | ~30 min |
| Findings Report | `findings-template.md` | `docs/findings/` | `{name}.md` | research output, findings, analysis, investigation, assessment | Context for research domain | P2 | ~15 min |

### Feature/Enhancement Combination

When the selected template is Feature/Enhancement Specification, include a metadata field:

```
Change Type: New Feature | Enhancement
```

When Change Type = Enhancement, read `~/.claude/skills/refine/templates/enhancement-spec-template.md` and include these 3 conditional sections in the interview and document at their appropriate positions:

- `2. Current State` (with `2.1 Current Behavior`, `2.2 Limitations & Issues`)
- `7. Impact Analysis` (with `7.1 Affected Components`, `7.2 Dependencies`, `7.3 Stakeholders`)
- `8. Backward Compatibility` (with `8.1 Breaking Changes`, `8.2 Deprecation Plan`, `8.3 Migration Path`)

---

## Phase 1 -- Input & Classification

Read the pitch file at the path provided in `$ARGUMENTS`.

**If `$ARGUMENTS` is empty:** Display usage help (`/refine <path-to-pitch-file>` with one example) and stop.

**If the file does not exist:** Display an error: "File not found: {path}. Verify the pitch file exists and try again." Stop.

**If the file exists:**

1. Read the pitch file content.
2. **Content quality gate:** If the pitch has fewer than 2 sections with substantive content, inform the user: "This pitch has minimal content -- the interview will cover all sections from scratch rather than building on pitch decisions. Consider running /pitch first." Proceed regardless.
3. Match pitch content against the Template Index's Signals to determine the best template.
4. Present the recommended template via AskUserQuestion with "(Recommended)" suffix, plus 2-3 alternatives.
5. Load the selected template from `~/.claude/skills/refine/templates/{template-file}`.
5.5. **Dual-document awareness:** If the selected template is PRD and the pitch content contains UI/UX signals (app, screen, page, button, navigation, dashboard, layout, modal, animation, onboarding, user-facing), inform the user: "This project has user-facing UI. After the PRD (technical specs and requirements), I'll offer to produce a Brand OS (visual identity, voice, and interaction design). The PRD defines WHAT to build; the Brand OS defines HOW it should look, sound, and feel. Builders receive whichever document is relevant to their task." This sets expectations before the interview begins.
6. **Runtime section extraction:** Read the loaded template file and extract all H2 (`##`) and H3 (`###`) section headings. Preserve the exact heading text including numeric prefixes (e.g., "2. Purpose & Background", "2.1 Problem Statement"). This section list serves as the coverage checklist for the interview -- the template file is the source of truth.
7. If the selected template is Feature/Enhancement Specification: ask a follow-up question via AskUserQuestion for Change Type (New Feature or Enhancement). If Enhancement: read `~/.claude/skills/refine/templates/enhancement-spec-template.md` and extract the 3 conditional sections from the Feature/Enhancement Combination section. Insert them into the section list at their appropriate positions.

---

## Phase 2 -- Full Interview

Read the interview guide at `~/.claude/skills/refine/supporting-files/interview-guide.md`.

### UI/UX Awareness

Determine if the project involves visual/UI work using these signals:

**UI/UX signals**: app, screen, page, button, form, navigation, dashboard, widget, layout, modal, toast, animation, responsive, mobile, dark mode, theme, onboarding, user-facing, TUI, terminal UI, CLI with visual output

If present, note this in the Domain Profile (Pitch Analysis Brief). The adaptive interview will naturally generate UI/UX questions for relevant template sections. If the template lacks dedicated design sections, create a `## UI/UX Design` section in the output document with subsections for: Screens/Views, Visual Style, User Workflow, and Experience Goals.

### Adaptive Interview Process

1. Generate the Pitch Analysis Brief (internal — not shown to user).
2. Evaluate conditional sections (if template contains `<!-- conditional: ... -->` markers — see Conditional Section Handling below).
3. Generate the first question batch from Deep Dive / high-priority Targeted Probe sections.
4. After each batch, update coverage tracking and adapt remaining questions based on answers received.
5. Run the coverage guarantee check (interview guide Section 7) before proceeding to Phase 3.

- **Stop condition:** All sections have concrete, user-provided answers or pitch-sourced content. No TBDs, no guesses, no hallucinated content. If a section cannot be populated from user input or pitch content, it goes to Open Questions.
- If the user triggers an escape hatch: apply the interview guide's escape hatch process (Section 4). Tag each default with a confidence level (High/Medium/Low). For sections requiring highly specific information (API contracts, RACI matrices, statistical methods, SLOs), mark defaults as Low confidence and add revisit conditions.
- For uncertain decisions: propose a reasonable default, flag the confidence level (High/Medium/Low), and add a revisit condition to the template's assumptions, risk, or open questions section (whichever is present).
- **Requirement cap:** If more than 12 MUST-priority functional requirements emerge during the interview, flag this to the user and suggest splitting into multiple documents or demoting lower-priority items to Should/Could. Bloated requirements produce bloated plans that fail during execution.

### Conditional Section Handling

Some template sections have `<!-- conditional: include by default. Exclude when: {criteria}. Trigger question: "{question}" -->` HTML comments.

**When:** During Pitch Analysis Brief generation, after classifying coverage but before generating questions.

**Logic:** For each conditional section, evaluate the exclusion criteria against the pitch content and Domain Profile. If exclusion criteria are met, add a trigger question to the first interview batch.

**Recording:** If user confirms skip, remove the section from the coverage checklist. All child H3s under a skipped H2 are also removed.

**Output:** Skipped sections appear in the generated document as: `## {N}. {Title}` followed by "Not applicable for this project. {One sentence explaining why.}"

Default is INCLUDE — only excluded when the user explicitly confirms.

For templates with `<!-- lifecycle-phase: resolution -->` markers (e.g., defect report), sections marked for a later lifecycle phase are automatically classified as "Absent (deferred to resolution phase)" in the Pitch Analysis Brief and excluded from the coverage checklist. These sections appear in the generated document with: "{To be completed during resolution phase.}"

---

## Phase 3 -- Research Enrichment (conditional)

Check the Template Index's Research Sections column for the selected template.

**Dispatch the researcher if ALL of the following are true:**
1. The template's Research Sections column is non-empty.
2. At least one research-eligible section has content with confidence level Medium or Low, OR the section content came from escape-hatch defaults.
3. The user has not explicitly stated that research is unnecessary for those sections.

Dispatch a researcher subagent (Task tool, subagent_type: "researcher") to research the eligible sections. Prompt must:
- List the research-eligible sections by template section heading.
- Include the interview content gathered so far for each section.
- Request output **keyed by template section heading**: for each section, provide a 2-3 sentence summary, source URLs, and confidence level (High/Medium/Low).

After the researcher returns:

- For each finding, present to the user with its confidence level: "Research found [X] for section [Y] (confidence: Z) -- incorporate, modify, or skip?"
- Integrate approved findings into the corresponding sections.

**Failure handling:** If the researcher fails or times out, log the failure to `docs/logs/gotcha-log.md` using the agent entry format (see `.claude/rules/gotcha-capture.md`). Add a note to the document's Open Questions: "Research enrichment unavailable -- consider manual research for [sections]." Proceed without research enrichment.

---

## Phase 4 -- Document Generation

1. Generate a filename using the Template Index's Filename Pattern. The `{name}` portion: kebab-case, matching `^[a-z0-9-]+$`, max 50 chars.
2. Create the Output Dir from the Template Index if it does not exist.
3. Write the document following the selected template's structure exactly:
   - All section headings preserved from the template (using the exact headings extracted in Phase 1, including numeric prefixes)
   - All content populated with real decisions from the interview
   - Metadata block at top (Status, Owner, Date, etc. per the template's metadata format)

**Voice:** Use active voice for requirements. Adopt consistent modal verbs: "must" for mandatory, "should" for recommended, "may" for optional. Use third person for system behavior, second person for user-facing scenarios. Avoid hedging ("approximately", "generally") in requirement statements.

**Given-When-Then rule:** If the document contains a Functional Requirements section (or equivalent behavioral requirements table), every MUST-priority requirement must include a Behavior column with at least one Given-When-Then scenario: `Given {context}, When {action}, Then {outcome}`. Should/Could requirements should include behavioral specs where the behavior is non-obvious.

Write the document to `{Output Dir}/{Filename Pattern}`.

**Executable Success Criteria rule:** Every success criterion in the document needs an ID, testable condition, verify command, and expected output. Format as a table:

| ID | Criterion | Verify Command | Expected Output |
|----|-----------|----------------|-----------------|
| SC-1 | {testable condition} | {command} | {expected output} |

---

## Phase 5 -- Quality Gate

### Stage 1: Completeness Check (inline)

Read the generated document. Verify ALL of the following:

1. Every H2/H3 section has substantive content (not just a heading followed by whitespace)
2. No placeholder patterns remain: `{...}`, `TBD`, `TODO`, `to be determined`, `N/A` without explanation
3. Every functional requirement has a MoSCoW priority (Must/Should/Could/Won't)
4. Every functional requirement rationale explains WHY, not just restates the requirement
5. Every MUST-priority functional requirement has a Given-When-Then behavior specification
6. Success criteria are measurable and include verify commands where applicable
7. Scope boundaries are explicit -- what is in scope AND what is out of scope

If any check fails: propose content to the user and fix before proceeding.

### Stage 2: Critic Review (5-cycle cap)

Dispatch a critic subagent (Task tool, subagent_type: "critic") to review the generated document. The prompt must specify:
- **Artifact type:** Pass the actual template type (e.g., "architecture proposal", "feature specification"), not a hardcoded type. If no matching critic checklist exists, instruct the critic to use the PRD checklist as the nearest approximation.
- **Additional focus:** Verify all decisions have confidence levels and revisit conditions where uncertain.
- **Output format:** Verdict (PASS/REVISE/DROP), Readiness Score (minimum of the four dimension scores), Findings (numbered), Required Changes (if REVISE).

**Routing:**

- **PASS (score >= 8):** Present the score to the user and proceed to Phase 6.
- **REVISE (score < 8):** Parse ALL Required Changes from critic (and designer, if Stage 3 ran) into individual change items. Present proposed changes to the user in rounds of up to 4 per AskUserQuestion call. Each change is a separate question within the call. For each question:
     - **Question text**: Include: (a) which reviewer proposed this change — Critic or Designer (annotate designer changes as "advisory"), (b) a detailed description of what the change is and which section of the document it affects, (c) the reviewer's reasoning for why it's needed, (d) whether the change conflicts with a user's explicit interview decision (if so, note: "This conflicts with your decision on {section}").
     - **Options** (same for every question):
       1. "Approve" — apply this change to the document
       2. "Deny" — defer it, note in Open Questions: "Deferred reviewer suggestion: {change summary}"
       3. "Revise" — let the user modify the change before applying
     - If more than 4 changes exist, continue with additional rounds until all changes have been reviewed.
     - After each round, for any "Revise" selections, ask the user for their modified version before proceeding to the next round.
     After applying all user-approved changes, re-run the critic.
- **DROP:** Surface immediately to the user: "The critic assessment indicates fundamental issues: [drop reasoning]. Options: address the root issues and re-run, or proceed with the document as-is with a warning in Open Questions."
- **Convergence:** If the readiness score does not improve between consecutive cycles, stop cycling. Present stalled findings: "Score plateaued at N/10 after M cycles. Remaining findings noted in Open Questions."
- **Cap:** After cycle 5, proceed regardless of score, noting unresolved issues in Open Questions.

**Failure handling:** If the critic fails or times out, log the failure to `docs/logs/gotcha-log.md` using the agent entry format (see `.claude/rules/gotcha-capture.md`). Proceed to Phase 6 with the Stage 1 completeness check as the sole quality gate. Note in Open Questions: "Critic review could not be completed."

### Stage 3: Designer Review (conditional)

**Trigger:** The selected template has UI/UX-related sections (e.g., Design Resources, UI/UX Considerations, Wireframes) OR interview content explicitly describes visual/interaction design.

Dispatch a designer subagent (Task tool, subagent_type: "designer") in Quick Mode to review the document's design-related sections. Request evaluation of: information architecture, interaction patterns, and accessibility considerations. Request output format: Design Assessment (scored), Findings (numbered), Required Changes.

Present designer findings alongside the critic's. Designer findings are advisory -- they do not independently block progression.

**Failure handling:** If the designer fails or times out, log the failure to `docs/logs/gotcha-log.md` using the agent entry format (see `.claude/rules/gotcha-capture.md`). Note in Open Questions: "Design review could not be completed -- consider manual design review." Proceed without design enrichment.

---

## Phase 6 -- Report & Routing

### Pitch Archival

After the document passes the quality gate, archive the source pitch if it's in a staging directory:
- If `$ARGUMENTS` points to a file in `docs/pitch/`: move it to `docs/archive/pitch/` using `mv`. Create `docs/archive/pitch/` if it doesn't exist.
- If `$ARGUMENTS` points elsewhere (project directory, external path): leave it in place.
- Report the move in the chat summary: "Archived source pitch: {old-path} → {new-path}".
- If `$ARGUMENTS` was not a file path (raw prompt), skip this step.

This keeps `docs/pitch/` as a staging area — pitches leave once they've been refined into technical documents.

### Chat Summary

Output: File path, Template type, Section count, Readiness score, Verdict, top 3-5 key decisions, and any archival performed.

### Companion Document Suggestion (conditional)

After producing a PRD, check if UI/UX signals were detected during the interview (Phase 2). If yes, and no Brand OS document exists at `docs/specs/brand-os-{name}.md`:

Present via AskUserQuestion: "This product has user-facing UI. A Brand Operating System document would define visual identity, voice, and interaction patterns — loaded by builders during UI tasks. Produce one?"

- "Yes, produce Brand OS (Recommended)" — Re-run Phases 1-5 with the Brand OS template (`brand-os-template.md`), reusing relevant PRD interview answers (positioning, personas, and experience goals map to Brand OS Core Identity sections). The Brand OS interview focuses on decisions not covered by the PRD: visual direction, voice attributes, motion philosophy, and design token intent.
- "No, skip for now" — Note in the PRD's Open Questions: "Brand OS not yet created — consider producing one before UI implementation begins."

After the companion suggestion (or if not triggered), proceed with routing.

### Routing

Then recommend the next step via AskUserQuestion:

- "Run /plan {output-path}" (Recommended) — If both PRD and Brand OS were produced, recommend: "Run /plan {prd-path}" and note the Brand OS path for the plan skill to detect.
- "Done for now"

If the user selects plan: output the invocation command. Do not auto-chain.

## Stage Handoff

When this skill completes, produce a structured handoff for the next stage. This handoff is part of the Report phase output — include it in the chat summary.

**Schema:**
- **Output file:** `{output-dir}/{filename}` (e.g., `docs/prd/prd-{name}.md`)
- **Document type:** {PRD | Brand OS | Feature Spec | Defect Report | etc.} — template used from skill templates
- **Companion documents:** List any companion documents produced in this session (e.g., "Brand OS at docs/specs/brand-os-{name}.md"). If none, omit this field.
- **Archived:** Source pitch path and archive destination (if archival was performed). Omit if no archival.
- **Key decisions:** 3-5 bullet points of requirements decisions made and why
- **Constraints established:** Scope boundaries, non-functional requirements, MoSCoW priorities
- **Open questions:** Unresolved items for the next stage
- **Scope:** Modules, features, and boundaries defined in the document

## Completion Token

Output `[SKILL_COMPLETE:refine]` after the user responds to the routing question:
- If user selects "/plan": emit the token, then output the invocation command.
- If user selects "Done for now": emit the token after displaying the file path.

## Metadata

- **Upstream:** `pitch` (produces the pitch file this skill refines)
- **Downstream:** `plan` (generates implementation tasks from the refined document)
- **Supporting files:** `supporting-files/interview-guide.md`
- **Templates:** `templates/` (12 document templates)
- **Reference:** `template-design-guide.md` (template design principles)
