# Template Design Guide

Best practices for designing technical document templates, with considerations for AI-assisted workflows.

---

## Core Principles

1. **Optimize for context economy.** Every token counts. Keep templates concise and use progressive disclosure -- load metadata first, detailed content only when needed.

2. **Prioritize machine-readability without sacrificing human readability.** Use standard markdown with consistent structure. Both humans and AI should parse it easily.

3. **Use shallow, tiered hierarchy.** Target section counts by template weight class:
   - **Lightweight** (findings, enhancement): 4-8 H2s
   - **Standard** (feature spec, defect report): 8-14 H2s
   - **Heavyweight** (operating model, architecture, service def): 14-20 H2s
   Avoid deep nesting (H4+). Within each tier, keep H3s proportional to the template's scope.

4. **Enable verification, not just instruction.** Templates should include success criteria, test commands, expected outputs, or comparison references so work can be self-verified.

   > **Risk table standard format:** All risk assessment tables use base columns `| Risk | Impact | Likelihood | Mitigation |` in this exact order. Template-specific extensions (Contingency Plan, Owner, Score, Risk ID) follow after the base columns.

5. **Separate what AI reads from what tools enforce.** Never include linting rules, formatting conventions, or other deterministic checks in AI-consumed templates. Use hooks, linters, or formatters instead.

6. **Include non-goals explicitly.** AI cannot infer from omission. State what NOT to do as clearly as what TO do.

7. **Make placeholders unambiguous.** Use consistent `{placeholder}` syntax with clear variable names.

8. **Provide examples inline.** Examples within the template help convey format and intent more effectively than external references.

9. **Treat templates as code.** Version control them, review changes, prune regularly, test by observing behavior.

10. **Design for the instruction budget.** LLMs reliably follow 150-200 instructions. Be ruthless in pruning -- each additional instruction degrades performance uniformly.

11. **Section headings drive interview questions.** Adding an H2/H3 heading adds coverage depth; removing one removes a question. Use `**Bold:**` prefix only for inline labels, never as structural subsections — the refine skill's extraction reads H2/H3 only.

---

## Recommended Placeholder Syntax

Use curly braces for all dynamic content:

```
{Product Name}
{YYYY-MM-DD}
{Description in 1-3 sentences}
```

Rules:

- One syntax per template (don't mix `{var}`, `[VAR]`, `$VAR`)
- Clear, descriptive names
- Include format hints where helpful: `{YYYY-MM-DD}`, `{High/Med/Low}`

---

## Structural Patterns

### Pattern A: Document Template (PRD, Spec, Proposal)

```markdown
# {Document Type}: {Title}

**Status:** {Draft | In Review | Approved}
**Owner:** {Name}
**Date:** {YYYY-MM-DD}

---

## Section 1

{Guidance text explaining what goes here.}

---

## Section 2

{Guidance text explaining what goes here.}
```

- Metadata block at top
- Horizontal rules between sections for visual clarity
- Guidance text in each section describes what to fill in

### Pattern B: YAML Frontmatter + Body (Skills, Agents)

```markdown
---
name: template-name
description: What this does and when to use it.
version: "1.0"
---

# Instructions

Step-by-step guidance.

## Examples

Concrete examples with inputs and outputs.
```

- Metadata loads first (~100 tokens) for discovery
- Full instructions load only when activated

### Pattern C: Progressive Disclosure with Imports

```markdown
# Main Document

See @docs/architecture.md for system context.
See @docs/api-reference.md for endpoint details.

## Core Content

{Main content here, kept concise.}
```

- Main file stays under 100 lines
- Detailed content loaded only when referenced

---

## Section Description Guidelines

Each section in a template should have guidance text that:

- **States what goes here** in 1-3 sentences
- **Provides format hints** (e.g., "List 3-5 objectives")
- **Gives examples of content types** (e.g., "Include metrics like response time, throughput, error rate")
- **Avoids being prescriptive about length** unless necessary

Good:
> Describe the user problem or business need this feature addresses. Include who experiences it, the impact, and why solving it matters now.

Bad:
> Write the problem statement here.

---

## Anti-Patterns to Avoid

### 1. The Kitchen Sink

Including every possible section in a single template. Leads to AI ignoring parts and context window exhaustion.

**Fix:** Keep templates focused. Use appendices for optional content.

### 2. Vague Placeholders

Guidance text like "Fill in details" or "Add relevant information."

**Fix:** Be specific about what type of content, how much, and in what format.

### 3. Deep Nesting

Sections nested 4+ levels deep (H4, H5, H6).

**Fix:** Flatten to H2/H3 maximum. Use bullet lists for sub-items instead of deeper headings.

### 4. Inconsistent Structure

Sections that vary wildly in format, depth, or style within the same template.

**Fix:** Establish a consistent pattern and apply it uniformly.

### 5. Missing Non-Goals / Out of Scope

Relying on omission to communicate what's excluded.

**Fix:** Always include an explicit "Out of Scope" or "Non-Goals" section.

### 6. Duplicating Tool Enforcement

Embedding code style rules, commit message formats, or linting rules in document templates.

**Fix:** Let linters, formatters, and hooks enforce deterministic rules. Keep templates focused on reasoning and content.

### 7. No Verification Path

Templates that describe what to produce but not how to verify it's correct.

**Fix:** Include acceptance criteria, test scenarios, or success metrics in every template.

---

## Template Inventory

| Template                      | Use Case               | File                                  |
| ----------------------------- | ---------------------- | ------------------------------------- |
| Product Requirements Document | New app                | `prd-template.md`                     |
| Feature Specification         | New feature            | `feature-spec-template.md`            |
| Enhancement Specification     | Feature improvement    | `enhancement-spec-template.md`        |
| Defect Report                 | Bug fix                | `defect-report-template.md`           |
| Service Definition            | New service            | `service-definition-template.md`      |
| System Requirements Document  | Internal system        | `system-requirements-doc-template.md` |
| Architecture Proposal         | System re-architecture | `architecture-proposal-template.md`   |
| Operating Model Design        | Workflow/org change    | `operating-model-design-template.md`  |
| Program Charter               | Cross-team effort      | `program-charter-template.md`         |
| Experiment Proposal           | Pilot/test             | `experiment-proposal-template.md`     |
| Brand Operating System        | Product brand identity | `brand-os-template.md`                |
| Findings Report               | Research output        | `findings-template.md`                |

---

## Sources

- [Claude Code Best Practices](https://code.claude.com/docs/en/best-practices)
- [AGENTS.md Specification](https://agents.md/)
- [Agent Skills Specification](https://agentskills.io/specification)
- [/llms.txt Specification](https://llmstxt.org/)
- [Writing a Good CLAUDE.md - HumanLayer](https://www.humanlayer.dev/blog/writing-a-good-claude-md)
- [Google Developer Style Guide - Placeholders](https://developers.google.com/style/placeholders)
- [How to Write PRDs for AI Coding Agents](https://medium.com/@haberlah/how-to-write-prds-for-ai-coding-agents-d60d72efb797)
