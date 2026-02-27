# Cross-File Lens

Detection patterns for cross-file consistency across agents, skills, rules, and workflows. These are issues that single-file analysis misses — they only emerge when comparing content across multiple files.

## File Discovery

Scan the target directory for all configuration files:

- **Agent definitions**: `.claude/agents/**/*.md`, `~/.claude/agents/**/*.md`
- **Skill definitions**: `.claude/skills/**/SKILL.md`, `~/.claude/skills/**/SKILL.md`
- **Rules**: `.claude/rules/*.md`
- **Project memory**: `CLAUDE.md`, `.claude/CLAUDE.md`
- **Dispatch tables**: Files containing skill-to-agent mappings (e.g., `subagent-skill-dispatch.md`)

Build an inventory of all discovered files before running checks.

## Detection Categories

### 1. Reference Integrity (HIGH Certainty)

**Skill References**
- Extract all skill name references from agent definitions, rules, dispatch tables, and CLAUDE.md
- Patterns: `/skill-name`, `skill: "name"`, `Skill('name')`, backtick references to skill names
- Validate each referenced skill exists at `~/.claude/skills/{name}/SKILL.md` or `.claude/skills/{name}/SKILL.md`
- Flag: Reference to a skill that does not exist on disk

**Agent References**
- Extract agent references from dispatch prompts, CLAUDE.md, and workflow files
- Patterns: `subagent_type`, `agent:`, references to agent file names
- Validate each referenced agent exists at `~/.claude/agents/{name}.md` or `.claude/agents/{name}.md`
- Flag: Reference to an agent that does not exist on disk

**Supporting File References**
- Extract `supporting-files/` references from SKILL.md files
- Validate each referenced file exists in the skill's directory
- Flag: Broken supporting file references

### 2. Dispatch Table Consistency (HIGH Certainty)

**Skill-to-Agent Mapping**
If a dispatch table exists (e.g., `subagent-skill-dispatch.md`):
- Every skill listed in the dispatch table should exist on disk
- Every agent type listed should be a valid subagent type (`builder`, `researcher`, `critic`, `validator`, `auditor`, `designer`)
- Flag: Dispatch table references non-existent skill or invalid agent type

**Skill-Awareness Coverage**
- Compare skills on disk against skills listed in the dispatch table
- Flag: Skills that exist on disk but have no dispatch table entry (may never be loaded by subagents)
- Note: This is MEDIUM certainty — some skills are user-invoked only and don't need dispatch entries

### 3. Instruction Consistency (MEDIUM Certainty)

**Duplicate Instructions**
- Scan for identical or near-identical MUST/NEVER/ALWAYS instructions across files
- Flag: Same instruction appearing in 3+ files — suggests it should be extracted to a shared rule file or CLAUDE.md
- Evidence: List all files containing the duplicate

**Contradictory Rules**
- Scan for logical contradictions across files:
  - File A says "ALWAYS do X" while File B says "NEVER do X"
  - File A says "Use tool Y" while File B says "Do not use tool Y"
- Flag: Contradictory instructions with file paths and line numbers
- Note: Check that contradictions are real, not scoped (e.g., "ALWAYS in tests" vs "NEVER in production" is not a contradiction)

**Convention Drift**
- Compare naming conventions used across files:
  - Branch naming patterns
  - Commit message formats
  - File naming conventions
- Flag: Inconsistent conventions across files that should follow the same standard

### 4. Tool Consistency (MEDIUM Certainty)

**Tool Usage vs Declaration**
For agent definitions with `tools:` or `allowed-tools:` frontmatter:
- Scan the agent/skill body for tool usage patterns (Read, Write, Grep, Glob, Bash, Task, Edit)
- Compare against declared tools
- Flag: Tools used in body but not declared in frontmatter (may cause runtime permission prompts)
- Flag: Tools declared but never used in body (unnecessary permissions)

**MCP Tool References**
- Scan for MCP tool references: `mcp__servername__toolname`
- Validate the MCP server is configured (check `.claude/settings.json` or `.claude/settings.local.json`)
- Flag: References to MCP tools from unconfigured servers

### 5. Orphan Detection (MEDIUM Certainty)

**Orphaned Agents**
- List all agent definition files
- Scan all other config files for references to each agent
- Flag: Agent files that are never referenced by any workflow, dispatch table, or CLAUDE.md
- Exception: Entry point agents (those designed to be invoked directly by the user) are not orphans

**Orphaned Skills**
- List all skill directories
- Scan for references in dispatch tables, agent definitions, rules, and CLAUDE.md
- Flag: Skills that are neither user-invocable nor referenced by any dispatch or agent
- Exception: Skills with `user-invocable: true` (default) are accessible via `/` menu and are not orphans

**Orphaned Rules**
- List all `.claude/rules/*.md` files
- Check if each rule's content is relevant to the current project (does it reference technologies, tools, or patterns actually used?)
- Flag: Rules that appear to be for a different project or technology stack (LOW certainty — may be aspirational)

### 6. Naming Consistency (LOW Certainty)

**Skill Name vs Directory**
- Skill `name` in frontmatter should match the directory name
- Flag: `name: foo-bar` in `skills/baz/SKILL.md`

**Agent Name vs Filename**
- Agent definition filename should match the agent's role
- Flag: File named `auditor.md` but content describes a builder role

**Cross-Reference Name Alignment**
- When one file references another by name, the name should match exactly
- Flag: References using stale names (e.g., referencing `review-tests` when the skill was renamed to `test-coverage-review`)

## Analysis Procedure

1. **Inventory**: Build a complete list of all config files with their types
2. **Extract references**: For each file, extract all outward references (skill names, agent names, tool names, file paths)
3. **Build reference graph**: Map which files reference which other files
4. **Run checks**: Apply each detection category to the reference graph
5. **Deduplicate**: Same issue found from multiple angles should appear once
6. **Report**: Group findings by category, sorted by certainty

## Iron Constraints

- **Never auto-fix.** Cross-file changes affect multiple files and need human review.
- **Skip example/bad-example blocks.** Content inside `<bad-example>`, `<bad_example>`, code blocks with "bad" in the info string, and lines prefixed with "Bad:" are intentional negative examples.
- **Entry points are not orphans.** Agents and skills designed for direct user invocation are not orphaned even if no other file references them.
- **Scope-aware contradiction check.** "ALWAYS X in tests" and "NEVER X in production" are not contradictions — they are scoped rules. Only flag contradictions where the scope is the same or unspecified.

## Pattern Statistics

| Category | Patterns | Certainty |
|----------|----------|-----------|
| Reference Integrity | 3 | HIGH |
| Dispatch Consistency | 2 | HIGH/MEDIUM |
| Instruction Consistency | 3 | MEDIUM |
| Tool Consistency | 2 | MEDIUM |
| Orphan Detection | 3 | MEDIUM |
| Naming Consistency | 3 | LOW |
| **Total** | **16** | — |
