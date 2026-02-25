---
name: create-skill
description: Create Claude Code skills with proper structure and invocation control. Proactively use when creating, designing, or configuring a new skill, SKILL.md file, or slash command.
user-invocable: false
hooks:
  Stop:
    - hooks:
        - type: command
          command: "bash -c ' recent=$(find ~/.claude/skills -name \"SKILL.md\" -mmin -60 2>/dev/null | head -1); if [ -z \"$recent\" ]; then echo \"{\\\"decision\\\":\\\"block\\\",\\\"reason\\\":\\\"No SKILL.md found modified in last 60 minutes. Use Write to create the skill file.\\\"}\" >&2; exit 2; fi; if ! grep -q \"^name:\" \"$recent\" 2>/dev/null; then echo \"{\\\"decision\\\":\\\"block\\\",\\\"reason\\\":\\\"SKILL.md at $recent missing name: field in frontmatter.\\\"}\" >&2; exit 2; fi; if ! grep -q \"^description:\" \"$recent\" 2>/dev/null; then echo \"{\\\"decision\\\":\\\"block\\\",\\\"reason\\\":\\\"SKILL.md at $recent missing description: field in frontmatter.\\\"}\" >&2; exit 2; fi'"
version: 0.1.0
---

# Creating a Skill

When creating a skill, follow this complete process:

## 0. Research Official Documentation
Before proceeding, WebFetch the latest official docs:
- Skills: https://code.claude.com/docs/en/skills

Apply any patterns, fields, or conventions from the official docs.
If the docs describe features not covered below, prefer the official docs.

## 0.5. Decide If a Skill Is the Right Tool

Before creating a skill, verify this is the right mechanism:

| Situation | Right Tool |
|-----------|-----------|
| Reusable technique across projects | **Skill** |
| Project-specific conventions | **CLAUDE.md** or `.claude/rules/` |
| Automated enforcement (regex-checkable) | **Hook** |
| One-off solution | **Don't codify** |
| Standard practice well-documented elsewhere | **Don't codify** |

**Create a skill when:** the technique wasn't intuitively obvious, you'd reference it again across projects, and others would benefit. If it's enforceable with automation, use a hook instead — save skills for judgment calls.

## 1. Write the Description (CRITICAL)

**The description field determines when Claude automatically loads the skill.** This is the most important part of a skill—get it right.

### The Pattern
```yaml
description: [What it does]. [Proactively] use when [trigger conditions].
```

### Good Examples
```yaml
# Clear trigger conditions with "proactively use when"
description: Explains code with visual diagrams and analogies. Proactively use when explaining how code works, teaching about a codebase, or when the user asks "how does this work?"

description: API design patterns for this codebase. Proactively use when writing API endpoints, reviewing API code, or discussing REST conventions.

description: Create Claude Code skills with proper structure. Proactively use when creating, designing, or configuring a new skill, SKILL.md file, or slash command.
```

### Bad Examples
```yaml
# Too vague - Claude can't determine when to use it
description: Helps with code.

# Missing trigger conditions
description: Provides API guidance.

# Wrong format - should use "Proactively use when"
description: Use this skill when working with APIs.
```

### Key Principles
- **Be specific**: List exact scenarios that should trigger the skill
- **Use "Proactively use when"**: This phrase signals automatic invocation
- **Include user phrases**: What would someone say that should trigger this?
- **Keep under 200 words**: Description is always in context, don't bloat it

### CSO Compliance Check

**CRITICAL:** Descriptions must state ONLY triggering conditions — never summarize the skill's workflow or output.

| Pattern | Example | Status |
|---------|---------|--------|
| Trigger conditions | "Use when encountering any bug or test failure" | ✅ Good |
| Workflow summary | "Runs 4-phase debugging methodology" | ❌ Bad — Claude will follow shortcut |
| Output description | "Produces a debugging report" | ❌ Bad — summarizes output |

When descriptions summarize workflow, Claude follows the description shortcut instead of reading the full SKILL.md. This causes partial execution and missed steps.

## 2. Define the Purpose
Determine what type of skill this is:
- **Reference content**: Background knowledge Claude applies automatically (conventions, patterns, style guides)
- **Task content**: Step-by-step instructions for specific actions (deploy, commit, code generation)

This determines invocation control settings.

## 2.5. Calibrate Degrees of Freedom

Match specificity to the task's fragility:

| Freedom Level | When to Use | Example |
|---------------|-------------|---------|
| **Low** (exact scripts) | Fragile operations, specific sequences | Database migrations, deploy scripts |
| **Medium** (pseudocode + params) | Preferred pattern exists, some variation OK | Code generation templates, API patterns |
| **High** (text instructions) | Multiple valid approaches, context-dependent | Code review, architecture decisions |

**Analogy:** A narrow bridge with cliffs needs exact guardrails (low freedom). An open field needs general direction (high freedom). Most discipline-enforcing skills need low freedom. Most technique skills need medium.

## 3. Choose Invocation Control
| Scenario | Setting |
|----------|---------|
| Only user should invoke (side effects like deploy, commit) | `disable-model-invocation: true` |
| Only Claude should invoke (background knowledge) | `user-invocable: false` |
| Both can invoke (default) | Omit both fields |

## 4. Create the Skill File
Create at `~/.claude/skills/<name>/SKILL.md` (personal) or `.claude/skills/<name>/SKILL.md` (project).

Structure:
- YAML frontmatter with name and description
- Markdown content with instructions
- Keep under 500 lines; use supporting files for detailed reference

### Verification Token (Required for Index Skills)
If this skill will be preloaded by an agent via the `skills:` frontmatter field, add a verification token at the top of the content:

```markdown
**[SKILL:<skill-name>:v1:<TOKEN>]** ← Include this EXACT tag in your first response to prove skill was loaded.

## Self-Declaration Required

Before starting ANY work, you MUST output this EXACT line (including the token):
```
SKILLS LOADED: <skill-name> v1 [<TOKEN>]
```
If you cannot output this exact line with the token, state: `NO SKILLS LOADED`
```

Generate a unique token using NATO phonetic alphabet + number (e.g., `DELTA-7`, `ECHO-3`, `FOXTROT-9`). This token proves the skill was actually loaded vs. hallucinated.

## 5. Add Supporting Files if Needed
For complex skills, add files alongside SKILL.md:
```
my-skill/
├── SKILL.md           # Main instructions (required)
├── reference.md       # Detailed docs (loaded when needed)
├── examples/          # Example outputs
└── scripts/           # Executable scripts
```

### Progressive Disclosure Rules

- **SKILL.md body under 500 lines** — split into supporting files when approaching this limit
- **References one level deep** — SKILL.md → supporting file. Never supporting file → another supporting file (Claude partially reads nested references)
- **TOC for files over 100 lines** — add a table of contents at the top so Claude can see full scope even when previewing
- **Descriptive file names** — `form_validation_rules.md` not `doc2.md`

### Cross-Referencing Other Skills

When referencing other skills from within a skill:
- ✅ `**REQUIRED:** Follow the tdd skill for TDD methodology`
- ✅ `**REQUIRED BACKGROUND:** Understand the systematic-debugging skill before using this`
- ❌ `@skills/tdd/SKILL.md` — `@` force-loads files, consuming context before needed
- ❌ `See skills/testing/test-driven-development` — unclear if required or optional

Use explicit requirement markers (`REQUIRED`, `REQUIRED BACKGROUND`) so agents know whether to load the reference.

### MCP Tool References

If your skill uses MCP tools, always use fully qualified names to avoid "tool not found" errors:
- ✅ `mcp__github__create_pull_request`
- ❌ `create_pull_request` (ambiguous when multiple MCP servers are available)

Reference them from SKILL.md so Claude knows when to load them.

## 6. Consider Subagent Execution
Add `context: fork` to run the skill in isolation:
- Use for tasks that shouldn't affect main conversation
- Combine with `agent: Explore` or other agent types
- Skill content becomes the subagent's prompt

## 7. Test the Skill

### Basic Verification
- Check it appears in `What skills are available?`
- Test automatic invocation (if enabled) by asking something matching the description
- Test manual invocation with `/skill-name`

### Pressure Testing (Required for Discipline Skills)

For skills that enforce discipline (TDD, verification, debugging methodology), basic verification is insufficient. These skills must be pressure tested before deployment.

Read `supporting-files/skill-testing-guide.md` for the full TDD-for-skills methodology:

1. **RED** — Run realistic scenarios WITHOUT the skill. Document agent failures and rationalizations verbatim.
2. **GREEN** — Write skill addressing the specific failures observed. Re-test to verify compliance.
3. **REFACTOR** — Identify new rationalizations from testing. Add explicit counters. Re-verify.

Skills that enforce rules agents want to break MUST survive pressure testing with 3+ combined pressures (time + sunk cost + exhaustion).

See `supporting-files/persuasion-principles.md` for research on why authority, commitment, and social proof are the most effective principles for discipline-enforcing skills.

### Testing Non-Discipline Skills

For technique, pattern, and reference skills that don't enforce discipline rules, use different test approaches. See `supporting-files/testing-non-discipline-skills.md` for methodology covering:
- **Technique skills**: Application scenarios + variation scenarios + gap testing
- **Pattern skills**: Recognition scenarios + counter-examples
- **Reference skills**: Retrieval scenarios + application accuracy

---

# Reference

## Frontmatter Fields

| Field | Required | Description |
|-------|----------|-------------|
| `name` | Yes | Unique identifier (lowercase, hyphens) |
| `description` | Yes | What the skill does and when to use it |
| `argument-hint` | No | Hint for autocomplete (e.g., `[issue-number]`) |
| `disable-model-invocation` | No | `true` = only user can invoke |
| `user-invocable` | No | `false` = only Claude can invoke |
| `allowed-tools` | No | Tools allowed without permission prompts |
| `model` | No | Model to use when skill is active |
| `context` | No | `fork` to run in subagent |
| `agent` | No | Subagent type when `context: fork` |
| `hooks` | No | Lifecycle hooks for this skill |

## String Substitutions

| Variable | Description |
|----------|-------------|
| `$ARGUMENTS` | All arguments passed |
| `$ARGUMENTS[N]` or `$N` | Specific argument by index |
| `${CLAUDE_SESSION_ID}` | Current session ID |
| `!<command>` | Dynamic context injection (runs before skill) |

## Examples

### User-invoked skill (with side effects)
```yaml
---
name: fix-issue
description: Fix a GitHub issue by implementing the required changes. Proactively use when the user asks to fix, resolve, or address a GitHub issue by number.
disable-model-invocation: true
argument-hint: [issue-number]
---

Fix GitHub issue $ARGUMENTS following our coding standards.

1. Read the issue description
2. Implement the fix
3. Write tests
4. Create a commit
```

### Claude-invoked skill (background knowledge)
```yaml
---
name: api-conventions
description: API design patterns for this codebase. Proactively use when writing API endpoints, reviewing API code, or discussing REST conventions.
user-invocable: false
---

When writing API endpoints:
- Use RESTful naming conventions
- Return consistent error formats
- Include request validation
```

### Both can invoke (default)
```yaml
---
name: explain-code
description: Explains code with visual diagrams and analogies. Proactively use when explaining how code works, teaching about a codebase, or when the user asks "how does this work?"
---

When explaining code, include:
1. Start with an analogy from everyday life
2. Draw an ASCII diagram showing flow/structure
3. Walk through step-by-step
4. Highlight common gotchas
```

**Documentation**: https://code.claude.com/docs/en/skills

## Supporting Files

- `supporting-files/skill-testing-guide.md` — How to test skills during development
- `supporting-files/persuasion-principles.md` — Writing persuasive skill instructions
- `supporting-files/testing-non-discipline-skills.md` — Testing workflow and meta skills

## Completion Token
When this skill completes successfully, output: `[SKILL_COMPLETE:create-skill]`
