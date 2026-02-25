# Claude Code Skills

A collection of skill definitions for [Claude Code](https://claude.ai/code). Skills provide structured methodologies, workflows, and review lenses that Claude Code agents can load on demand via the `/skill` command or the `Skill()` tool.

## Skills

| Skill | Description | Category |
|-------|-------------|----------|
| [ci-fix](skills/ci-fix/SKILL.md) | CI failure diagnosis and fix | workflow |
| [create-hook](skills/create-hook/SKILL.md) | Create Claude Code hooks for workflow automation | meta |
| [create-mcp](skills/create-mcp/SKILL.md) | Configure MCP servers to connect Claude Code to external tools | meta |
| [create-skill](skills/create-skill/SKILL.md) | Create Claude Code skills with proper structure and invocation control | meta |
| [create-subagent](skills/create-subagent/SKILL.md) | Create custom Claude Code sub-agents with full ecosystem setup | meta |
| [deslop](skills/deslop/SKILL.md) | AI slop detection and cleanup | discipline |
| [drift-detection](skills/drift-detection/SKILL.md) | Drift analysis between plans, docs, and code | discipline |
| [enhance](skills/enhance/SKILL.md) | Audit and enhance Claude Code configuration files | meta |
| [execute](skills/execute/SKILL.md) | Agent-driven plan execution | workflow |
| [execute-beta](skills/execute-beta/SKILL.md) | Agent-teams-driven plan execution | workflow |
| [github](skills/github/SKILL.md) | GitHub workflow operations (PRs, issues, releases, CI) | workflow |
| [iterate](skills/iterate/SKILL.md) | Post-build project refinement and iteration | workflow |
| [learn-topic](skills/learn-topic/SKILL.md) | Progressive deep-research methodology | workflow |
| [pitch](skills/pitch/SKILL.md) | Idea shaping and viability assessment | workflow |
| [plan](skills/plan/SKILL.md) | Implementation task decomposition from technical documents | workflow |
| [plan-beta](skills/plan-beta/SKILL.md) | Task decomposition with agent teams execution model | workflow |
| [refine](skills/refine/SKILL.md) | Pitch-to-specification refinement | workflow |
| [review-architecture](skills/review-architecture/SKILL.md) | Architecture review lens for large codebases | discipline |
| [review-code-quality](skills/review-code-quality/SKILL.md) | Code quality review lens | discipline |
| [review-performance](skills/review-performance/SKILL.md) | Performance review lens | discipline |
| [review-security](skills/review-security/SKILL.md) | Security review lens | discipline |
| [sync-docs](skills/sync-docs/SKILL.md) | Documentation-code sync check | discipline |
| [systematic-debugging](skills/systematic-debugging/SKILL.md) | Root cause investigation before proposing fixes | discipline |
| [tdd](skills/tdd/SKILL.md) | Test-driven development methodology | discipline |
| [test-coverage-review](skills/test-coverage-review/SKILL.md) | Test quality and coverage review | discipline |
| [verification-before-completion](skills/verification-before-completion/SKILL.md) | Verification evidence before claiming completion | discipline |

### Categories

- **workflow** -- Skills that drive end-to-end workflows (planning, execution, CI, GitHub operations)
- **meta** -- Skills for creating and managing Claude Code configuration (skills, hooks, MCP servers, agents)
- **discipline** -- Skills that enforce quality practices (TDD, debugging, code review, security)

## Installation

### Symlink (recommended)

Link the entire skills directory so updates are reflected automatically:

```bash
ln -sf ~/repos/claude-skills/skills ~/.claude/skills
```

### Copy individual skills

Copy specific skills you need:

```bash
cp -r ~/repos/claude-skills/skills/tdd ~/.claude/skills/
cp -r ~/repos/claude-skills/skills/systematic-debugging ~/.claude/skills/
```

## Usage

Once installed, invoke skills in Claude Code:

```
/skill tdd
/skill systematic-debugging
/skill review-security
```

Or programmatically via the `Skill()` tool in agent prompts:

```
Skill('tdd')
Skill('systematic-debugging')
```

## Structure

```
skills/
  {skill-name}/
    SKILL.md              # Skill definition with YAML frontmatter
    supporting-files/     # Optional additional resources
```

Each `SKILL.md` includes YAML frontmatter with `name`, `description`, and `version` fields.

## License

Private repository. All rights reserved.
