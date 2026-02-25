# Skills Lens

Detection patterns for SKILL.md skill definition files. Apply every check in this document to each discovered skill file.

## File Discovery

Find all skill files using: `Glob("**/SKILL.md")` under the target path. Default paths:
- `~/.claude/skills/` (personal skills)
- `.claude/skills/` (project skills)

For each SKILL.md, also check for supporting files in the same directory.

## Detection Categories

### 1. Frontmatter Validation (HIGH Certainty)

**Required Fields**
- File must have YAML frontmatter delimited by `---`
- `name` field: lowercase, hyphens only, max 64 characters
- `description` field: max 1024 characters, must include trigger conditions

**Flag:**
- Missing `---` frontmatter delimiters
- Missing `name` field
- Missing `description` field
- `name` with uppercase characters, spaces, or underscores
- `description` exceeding 1024 characters

**Recommended Fields (MEDIUM if missing)**
- `argument-hint` for skills that accept input
- `allowed-tools` for skills that use Bash or other powerful tools

| Field | Required | Validation |
|-------|----------|------------|
| `name` | Yes | Lowercase, hyphens, max 64 chars |
| `description` | Yes | Max 1024 chars, includes trigger |
| `argument-hint` | Recommended | Under 30 chars, describes expected input |
| `disable-model-invocation` | Conditional | Required if skill has side effects (deploy, commit, push) |
| `user-invocable` | Optional | Set `false` for background-only skills |
| `allowed-tools` | Recommended | Restrict powerful tools like Bash |
| `model` | Optional | `opus`, `sonnet`, or `haiku` |
| `context` | Optional | `fork` for isolated execution |
| `agent` | Conditional | Required when `context: fork` |

### 2. Trigger Quality (HIGH Certainty)

The description field determines when Claude automatically loads the skill. Poor triggers mean the skill never fires or fires incorrectly.

**Required Pattern**
Description must include trigger context using one of:
- "Proactively use when..."
- "Use when..."
- "Invoke when..."

**Good triggers** (specific, actionable):
- `"Proactively use when creating, designing, or configuring a new skill, SKILL.md file, or slash command."`
- `"Use when encountering any bug, test failure, or unexpected behavior — before proposing fixes"`

**Bad triggers** (vague, missing context):
- `"Reviews code"` — no trigger conditions
- `"Useful tool"` — no specificity
- `"Helps with development"` — too broad

**CSO Compliance**
Description must state ONLY triggering conditions. Flag descriptions that:
- Summarize the skill's workflow ("Runs 4-phase methodology") — causes Claude to shortcut
- Describe outputs ("Produces a debugging report") — summarizes instead of triggering
- Mix triggers with implementation details

**Flag:**
- Description without any trigger phrase
- Description that summarizes workflow or output instead of triggers
- Description with only one vague trigger ("Use when needed")

### 3. Invocation Control (HIGH Certainty)

Skills with side effects must be protected from automatic invocation.

**Side-Effect Detection**
Flag skills that match these patterns but lack `disable-model-invocation: true`:
- Name contains: `deploy`, `ship`, `publish`, `push`, `commit`, `release`, `delete`, `remove`
- Body contains: `git push`, `git commit`, `npm publish`, `deploy`, `rm -rf`
- Uses `Bash` without restrictions

**Background-Only Detection**
Flag skills that appear to be reference/background knowledge but are user-invocable:
- Body is purely informational (no workflow steps, no numbered instructions)
- No `$ARGUMENTS` usage
- Could suggest `user-invocable: false`

### 4. Tool Restriction Validation (HIGH Certainty)

**Unrestricted Bash**
Flag skills with `Bash` in `allowed-tools` without scope restriction:
- Bad: `allowed-tools: Bash`
- Good: `allowed-tools: Bash(git:*)`, `Bash(npm:*)`, `Bash(gh:*)`

**Tool-Role Mismatch**
- Read-only / analysis skills should not have `Write`, `Edit`, or unrestricted `Bash`
- Research skills should not have `Task` tool (spawning subagents from a skill is unusual)
- Flag: `allowed-tools` includes tools inconsistent with the skill's stated purpose

**Missing Tool Declarations**
- Scan the skill body for tool usage patterns (`Read`, `Write`, `Grep`, `Glob`, `Bash`, `Task`)
- Compare against `allowed-tools` if declared
- Flag: Tools used in body but not in `allowed-tools` (may cause permission prompts)

### 5. Content Scope (MEDIUM Certainty)

**Size Limits**
- SKILL.md body should be under 500 lines
- Flag: Files exceeding 500 lines — suggest splitting into supporting files

**Supporting File References**
- References must be one level deep (SKILL.md → supporting file)
- Flag: Supporting files that reference other supporting files (Claude partially reads nested references)
- Flag: `@` force-load syntax (`@skills/name/file.md`) — consumes context before needed

**Dynamic Context Injection**
- Skills can use `` !`command` `` for dynamic content
- Maximum 3 injections per skill (each adds to context budget)
- Flag: More than 3 dynamic injections

**Cross-Skill References**
- Should use explicit requirement markers: `**REQUIRED:**` or `**REQUIRED BACKGROUND:**`
- Flag: Vague references like "See the testing skill" without requirement level

### 6. Structure Quality (MEDIUM Certainty)

**Recommended Sections**
Well-structured skills typically have:
- Purpose/overview section
- "When to Use" section
- Workflow or methodology steps
- Constraints / Iron Constraints section
- Output format (if applicable)
- Supporting Files listing (if applicable)
- Completion Token

**Flag:**
- No workflow/methodology section (skill is purely declarative with no guidance)
- No constraints section for skills that modify files or make decisions
- Missing Completion Token (`[SKILL_COMPLETE:name]`)

### 7. Context Configuration (MEDIUM Certainty)

**Fork Configuration**
- `context: fork` requires an `agent` field
- `agent` field without `context: fork` is ignored (likely a mistake)
- Flag: Mismatched fork/agent configuration

**Agent-Tool Alignment**
When `context: fork` is set:
| Agent Type | Expected Tools |
|------------|---------------|
| `Explore` | Read, Grep, Glob only |
| `Plan` | Read, analysis tools |
| `general-purpose` | All tools |

Flag: `agent: Explore` with Write/Edit in `allowed-tools`

### 8. Anti-Patterns (LOW Certainty)

- **Multi-responsibility**: Skill tries to do too many unrelated things (>3 distinct workflow branches). Should be split.
- **Missing argument-hint**: Skill uses `$ARGUMENTS` but has no `argument-hint` field (users won't know what to pass).
- **Redundant CoT**: Phrases like "think step by step", "let's reason through this" — modern models don't need these.
- **Hardcoded paths**: Absolute paths that won't work across machines (e.g., `/Users/specific-user/...`).
- **Version field**: Not standard in Claude Code skills — flag if present (it's an agentsys convention, not a Claude Code one).

## Pattern Statistics

| Category | Patterns | Certainty | Auto-Fixable |
|----------|----------|-----------|--------------|
| Frontmatter | 5 | HIGH | 2 (add missing delimiters, lowercase name) |
| Trigger Quality | 4 | HIGH | 1 (add trigger prefix) |
| Invocation Control | 3 | HIGH | 0 |
| Tool Restrictions | 3 | HIGH | 0 |
| Content Scope | 4 | MEDIUM | 0 |
| Structure Quality | 3 | MEDIUM | 0 |
| Context Config | 2 | MEDIUM | 0 |
| Anti-Patterns | 5 | LOW | 0 |
| **Total** | **29** | — | **3** |
