# Project Memory Lens

Detection patterns for CLAUDE.md and project memory files. Apply every check in this document to each discovered file.

## File Detection

Search for project memory files in priority order:
1. `CLAUDE.md` (Claude Code — root, `.claude/`, `.github/`)
2. `AGENTS.md` (OpenCode, Codex — root, `.opencode/`, `.github/`)

Also load `README.md` if it exists (used for duplication comparison in Efficiency checks).

## Detection Categories

### 1. Structure Validation (HIGH Certainty)

**Critical Rules Section**
- File should have a `## Critical Rules` or equivalent top-level section
- Rules should be prioritized (numbered or ordered by importance)
- Each rule should include a WHY explanation (`*WHY: ...*` or indented rationale)
- Flag: Rules without explanations, unprioritized rule lists

**Architecture Section**
- File should document directory structure or key file locations
- Module relationships and boundaries should be described
- Flag: No architecture/structure section in files >500 characters

**Key Commands Section**
- Common development commands (build, test, lint, deploy) should be documented
- References to `package.json` scripts, `Makefile` targets, or equivalent
- Flag: No commands section, or commands that reference non-existent scripts

### 2. Instruction Effectiveness (HIGH Certainty)

Based on prompt engineering research — Claude follows instructions better with specific patterns:

**Strong Constraint Language**
- Critical rules should use "must", "always", "required"
- Weak language ("should", "try to", "consider") reduces compliance for important rules
- Flag: Critical rules using weak language — look for rules marked as important/critical that use "should" instead of "must"

**Instruction Hierarchy**
- File should define priority order when rules conflict
- Pattern: "In case of conflict: X takes precedence over Y"
- Flag: No conflict resolution guidance in files with >10 rules

### 3. Content Positioning (HIGH Certainty)

LLMs have a "lost in the middle" problem — they recall the START and END of context better than the MIDDLE.

**Critical Content Placement**
- Most important rules should appear in the first 25% of the file
- Second-most important at the END (final 15%)
- Supporting context and examples in the MIDDLE
- Flag: Critical rules (containing "must", "always", "never", "critical") appearing only in the middle 60% of the file

**Recommended Structure Order**
1. Critical Rules / Iron Laws (START — highest attention)
2. Architecture / Structure
3. Commands / Workflows
4. Examples / References
5. Reminders / Constraints (END — high attention)

### 4. Reference Validation (HIGH Certainty)

**File Path References**
- Extract paths from `[text](path)`, `` `path/to/file.ext` ``, and inline references
- Validate each exists on the filesystem using Glob or Read
- Flag: Broken file references (file does not exist)

**Command References**
- Extract `npm run <script>`, `make <target>`, and similar command references
- Validate against `package.json` scripts or `Makefile` targets
- Flag: References to non-existent scripts or targets

**Skill and Agent References**
- Extract references to skill names (`/skill-name`, `skill: "name"`)
- Validate skill exists at `~/.claude/skills/{name}/SKILL.md` or `.claude/skills/{name}/SKILL.md`
- Flag: References to non-existent skills

### 5. Efficiency Analysis (MEDIUM Certainty)

**Token Estimation**
- Estimate: `character_count / 4` (rough token count)
- Recommended maximum: 1500 tokens (~6000 characters) for project CLAUDE.md
- Global CLAUDE.md (`~/.claude/CLAUDE.md`) should be even leaner (<1000 tokens)
- Flag: Files exceeding the threshold with specific sections that could be moved to `.claude/rules/`

**README Duplication**
- Compare content with README.md (if it exists)
- Look for >40% content overlap (copy-pasted sections, duplicated architecture descriptions)
- Flag: High duplication — CLAUDE.md should complement README, not duplicate it

**Verbosity**
- Prose paragraphs >5 sentences should be bulleted lists instead
- Constraints as lists are easier for LLMs to parse than embedded in paragraphs
- Flag: Long prose blocks that could be reformatted as lists

### 6. Quality Checks (MEDIUM Certainty)

**WHY Explanations**
- Rules should explain their rationale
- Patterns: `*WHY: ...*`, `<!-- WHY: ... -->`, indented explanation after rule
- Flag: >50% of rules lack WHY explanations

**Structure Depth**
- Avoid nesting deeper than 3 heading levels (`###`)
- Keep hierarchy scannable — flat structures parse better for LLMs
- Flag: Sections nested 4+ levels deep

**Section Length Balance**
- No single section should be >40% of the total file
- Flag: Oversized sections that should be split or moved to `.claude/rules/`

### 7. File Hierarchy Awareness (MEDIUM Certainty)

Claude Code supports scoped CLAUDE.md files:

| Location | Scope |
|----------|-------|
| `~/.claude/CLAUDE.md` | Global (all projects) |
| `./CLAUDE.md` or `.claude/CLAUDE.md` | Project root |
| `src/.claude/CLAUDE.md` | Directory-specific |

**Check:**
- Global CLAUDE.md should contain only cross-project preferences (not project-specific rules)
- Project CLAUDE.md should not duplicate global rules
- Flag: Project-specific rules in global CLAUDE.md, or global preferences duplicated in project file

## Pattern Statistics

| Category | Patterns | Certainty |
|----------|----------|-----------|
| Structure | 3 | HIGH |
| Instruction Effectiveness | 2 | HIGH |
| Content Positioning | 2 | HIGH |
| Reference Validation | 3 | HIGH |
| Efficiency | 3 | MEDIUM |
| Quality | 3 | MEDIUM |
| File Hierarchy | 2 | MEDIUM |
| **Total** | **18** | — |
