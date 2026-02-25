---
name: create-subagent
description: Create custom Claude Code sub-agents with full ecosystem setup. Proactively use when creating, designing, or configuring a new subagent, agent definition, or .claude/agents/ file.
user-invocable: false
version: 0.1.0
---

# Creating a Sub-agent

When creating an agent, follow this complete process:

## 0. Research Official Documentation
Before proceeding, WebFetch the latest official docs:
- Sub-agents: https://code.claude.com/docs/en/sub-agents

Apply any patterns, fields, or conventions from the official docs.
If the docs describe features not covered below, prefer the official docs.

## 1. Design the Agent
Create the agent file at `~/.claude/agents/<name>.md` (personal) or `.claude/agents/<name>.md` (project) with:
- Clear name (lowercase, hyphens)
- Detailed description explaining when to delegate
- Minimal tool access (only what's needed) — do NOT include `Skill` in the tools list
- Appropriate model: `haiku` for fast/simple, `sonnet` for complex, `opus` for critical

## 2. Required Agent Structure

Every agent file MUST include these sections in this order:

### Frontmatter
```yaml
---
name: <name>
description: <detailed description of when to delegate to this agent>
tools: <comma-separated tool list — NO Skill tool>
model: <haiku|sonnet|opus>
---
```

### Opening
```markdown
**On start, declare:** `AGENT LOADED: <name> v1 [<NATO-PHONETIC-NUMBER>]`

When invoked:
1. <Context gathering step specific to domain>
2. Review relevant project context and existing patterns
3. <Domain-specific preparation step>
```

### Domain Checklists
Comprehensive evaluation items organized by sub-domain. 4-6 checklists with 6-8 items each. These ensure consistent, thorough output across invocations.

### Implementation Workflow
Three phases:
- **Phase 1: Analysis** — What to examine before acting
- **Phase 2: Execution** — How to perform the work
- **Phase 3: Delivery** — Excellence checklist for output quality

### Constraints
Negative guidance and authority boundaries. What the agent should NOT do. Who it accepts critique from.

### Integration
Explicit collaboration patterns with other agents (collaborate with, coordinate with, work with, consult, support).

### Closing
A single line summarizing the agent's core priorities.

## 3. Evaluate Supporting Skills
Consider what background knowledge would make this agent more effective:
- Domain patterns, conventions, or standards
- Reference docs it should always have access to

If beneficial, create skills with `user-invocable: false` in `~/.claude/skills/<name>/SKILL.md` and add them to the agent's `skills:` frontmatter field.

**Important:** Skills listed in the `skills:` frontmatter are auto-loaded into the agent's context at spawn time. The agent does NOT need to call the Skill tool to access them.

## 4. Evaluate Hook Needs
Consider if lifecycle automation would help:
- `SubagentStart` hooks when this agent spawns
- `PostToolUse` hooks for this agent's tool patterns

Only add hooks if they provide clear, specific value.

## 5. Check MCP Servers
Run `claude mcp list` to see available integrations.
Recommend which servers would benefit this agent and explain why.

## 6. Wire Everything
- Set `skills:` field in agent frontmatter to preload supporting skills
- Report: agent location, skills created (if any), hooks added (if any), MCP recommendations

---

# Reference

## Frontmatter Fields

| Field | Required | Description |
|-------|----------|-------------|
| `name` | Yes | Unique identifier (lowercase, hyphens) |
| `description` | Yes | When Claude should delegate to this agent |
| `tools` | No | Allowlist of tools (inherits all if omitted) |
| `disallowedTools` | No | Tools to deny |
| `model` | No | `sonnet`, `opus`, `haiku`, or `inherit` |
| `permissionMode` | No | `default`, `acceptEdits`, `dontAsk`, `bypassPermissions`, `plan` |
| `skills` | No | Skills to preload into agent context |
| `hooks` | No | Lifecycle hooks scoped to this agent |

## Example

```yaml
---
name: code-reviewer
description: Expert code review specialist. Proactively reviews code for quality, security, and maintainability.
tools: Read, Grep, Glob, Bash
model: sonnet
---

**On start, declare:** `AGENT LOADED: code-reviewer v1 [ALPHA-4]`

When invoked:
1. Query project context for coding standards and recent changes
2. Review relevant project context and existing patterns
3. Run git diff to identify changed files before reviewing

## Domain Checklists

### Code Quality
- Naming clarity and consistency
- Function length and single responsibility
- Error handling coverage
- ...

### Security
- Input validation
- Authentication checks
- ...

## Implementation Workflow

### Phase 1: Review Analysis
Analysis priorities:
- Identify scope of changes
- ...

### Phase 2: Review Execution
Review approach:
- ...

### Phase 3: Review Delivery
Excellence checklist:
- ...

## Constraints
- Review and recommend; do not modify code directly
- Accept critique from architect, security-engineer
- ...

## Integration
- Collaborate with developer on remediation guidance
- Coordinate with security-engineer on vulnerability findings
- ...

Always prioritize actionable feedback, security awareness, and code quality.
```

### Skill Declaration Pattern
When an agent has preloaded skills, add a declaration line:

```markdown
**On start, declare:** `SKILLS LOADED: <skill-name> v1 [<TOKEN>]`
```

The token must match the token in the skill file. This verifies the skill was actually loaded into context.

**Documentation**: https://code.claude.com/docs/en/sub-agents

## Completion Token
When this skill completes successfully, output: `[SKILL_COMPLETE:create-subagent]`
