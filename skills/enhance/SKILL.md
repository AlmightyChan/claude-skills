---
name: enhance
description: "Audit and enhance Claude Code configuration files. Proactively use when auditing, validating, or improving CLAUDE.md project memory files, SKILL.md skill definitions, or cross-file consistency across agents, skills, and rules."
argument-hint: "[target: project-memory | skills | cross-file] [path]"
version: 0.1.0
---

# Enhance

- **Agents:** builder

Analyze Claude Code configuration files for best-practice gaps and produce a structured report with certainty-based triage.

**Core principle:** Configuration quality compounds — a weak CLAUDE.md degrades every session, a bad skill trigger means the skill never fires, and cross-file inconsistencies create silent failures.

## When to Use

- Auditing CLAUDE.md or project memory files for structure, references, and efficiency
- Validating SKILL.md files for frontmatter, trigger quality, and tool restrictions
- Checking cross-file consistency across agents, skills, rules, and workflows
- Periodic maintenance sweep of the BASECAMP configuration ecosystem

## Variables

- `TARGET`: First argument — one of `project-memory`, `skills`, or `cross-file`
- `PATH`: Optional second argument — directory or file to analyze (defaults to current directory for project-memory/cross-file, `~/.claude/skills/` for skills)

## Step 1: Determine Target

Parse `$ARGUMENTS` to identify the enhancement target:

| Input | Target | Default Path |
|-------|--------|-------------|
| `project-memory` | CLAUDE.md / project memory files | Current directory |
| `skills` | SKILL.md files | `~/.claude/skills/` |
| `cross-file` | Cross-file consistency | Current directory |

If no target is specified, ask: "What would you like to enhance? (project-memory, skills, or cross-file)"

## Step 2: Load Target Lens

Read the appropriate supporting file for the target:

| Target | Supporting File |
|--------|----------------|
| `project-memory` | `supporting-files/project-memory-lens.md` |
| `skills` | `supporting-files/skills-lens.md` |
| `cross-file` | `supporting-files/cross-file-lens.md` |

The lens contains the complete checklist of detection patterns, validation rules, and quality heuristics for that target type. Follow ALL checks defined in the lens.

## Step 3: Discover Files

Scan the target path to find files to analyze:

- **project-memory**: Find CLAUDE.md files (root, `.claude/`, `.github/`), also check for AGENTS.md. Load README.md for duplication comparison.
- **skills**: Find all `SKILL.md` files under the target path using `Glob("**/SKILL.md")`.
- **cross-file**: Find all agent definitions (`.claude/agents/**/*.md`), skill definitions (`skills/**/SKILL.md`), rules (`.claude/rules/*.md`), and workflow files.

## Step 4: Analyze

For each discovered file, run every detection pattern from the loaded lens. For each finding:

1. **Identify the issue** — what specifically violates the pattern
2. **Classify certainty** — using the thresholds below
3. **Propose a fix** — concrete, actionable recommendation
4. **Cite evidence** — the exact line or content that triggered the finding

### Certainty Classification

| Level | Threshold | Meaning |
|-------|-----------|---------|
| HIGH | >95% confidence | Structural/syntactic violation, clear contradiction, missing required field. Unambiguous. |
| MEDIUM | 75-95% confidence | Best-practice gap, context-dependent issue, efficiency flag. Likely correct but may have valid exceptions. |
| LOW | <75% confidence | Anti-pattern, subjective quality preference, style issue. May be intentional. |

## Step 5: Report

Generate a structured report following this format:

```markdown
# Enhancement Analysis: {target}

**Target**: {path}
**Files Analyzed**: {count}

## Summary

| Certainty | Count |
|-----------|-------|
| HIGH | {n} |
| MEDIUM | {n} |
| LOW | {n} |

## HIGH Certainty Issues

### [{category}] {issue title}
**File**: {path}:{line}
**Issue**: {description}
**Evidence**: {code snippet or content}
**Fix**: {concrete recommendation}

## MEDIUM Certainty Issues

[same format]

## LOW Certainty Issues

[same format, only if --verbose or few total findings]
```

Group findings by category (as defined in each lens), then by certainty within each category.

## Iron Constraints

- **Report only.** Do not modify any files unless the user explicitly asks for fixes.
- **Evidence required.** Every finding must cite the specific content that triggered it. No "this file could be improved" without pointing to what.
- **Validate before flagging.** For reference validation (file paths, script names), check the filesystem before reporting broken references. False positives erode trust.
- **Skip generated content.** Do not analyze files in `dist/`, `build/`, `node_modules/`, or `.git/`.
- **Respect context.** A pattern that looks wrong in isolation may be intentional. When certainty is ambiguous, downgrade to MEDIUM or LOW.

## Related Skills

- `deslop` — AI slop detection in code (enhance focuses on configuration files, not source code)
- `sync-docs` — Documentation-code synchronization (enhance focuses on AI config files, sync-docs covers general docs)
- `review-code-quality` — Code quality review (enhance is for config/prompt quality, not code)

## Supporting Files

- `supporting-files/project-memory-lens.md` — Detection patterns for CLAUDE.md and project memory files
- `supporting-files/skills-lens.md` — Detection patterns for SKILL.md skill definitions
- `supporting-files/cross-file-lens.md` — Detection patterns for cross-file consistency

## Completion Token

When this skill completes successfully, output: `[SKILL_COMPLETE:enhance]`
