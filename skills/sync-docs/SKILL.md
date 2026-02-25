---
name: sync-docs
description: "Documentation-code sync check. Proactively use when verifying doc accuracy after code changes, checking for stale references, outdated examples, or missing CHANGELOG entries."
version: 0.1.0
---

# Sync Docs

## Overview

Verify that documentation accurately reflects the current state of the codebase. Detect stale references, outdated examples, missing CHANGELOG entries, and broken cross-references. Adapted from the sync-docs-agent's five-phase methodology.

**Core principle:** Documentation that contradicts code is worse than no documentation. It actively misleads.

## When to Use

- After code changes that may affect documentation
- Checking for stale references in docs after refactoring
- Verifying outdated examples still work
- Checking CHANGELOG against recent commits
- Pre-release documentation accuracy sweep
- Periodic documentation health checks

## Methodology

### Phase 1: Validate Documentation Structure

Scan the documentation directory for structural issues:
- Files referenced in indexes/TOCs that don't exist
- Broken relative links between documentation files
- Files with no inbound links (orphaned docs)

Use Glob to find all `.md` files in `docs/`, `README.md`, `CHANGELOG.md`.
Use Grep to extract cross-references and validate targets exist.

### Phase 2: Discover Related Documentation

For each recently changed source file, find documentation that references it:

1. Use Grep to search all `.md` files for references to changed file paths
2. Search for references to exported functions/classes/types from changed files
3. Check README for feature descriptions that may be affected
4. Check API documentation for endpoint descriptions

Build a mapping: `changed-source-file -> [related-doc-files]`

### Phase 3: Analyze Issues by Category

For each related doc file, check for specific issue types:

| Category | What to Check | Detection Method |
|----------|--------------|-----------------|
| **Outdated versions** | Package versions in install instructions | Grep for version patterns, compare to actual |
| **Removed exports** | Docs reference functions/types no longer exported | Grep doc for identifiers, verify in source |
| **Wrong import paths** | Docs show import paths that don't resolve | Extract import examples, verify with Glob |
| **Stale code examples** | Code snippets that no longer compile or run | Extract code blocks, check for referenced identifiers |
| **Missing features** | Implemented features with no documentation | Compare source exports to doc coverage |

### Phase 4: Check CHANGELOG

Compare CHANGELOG against recent commits:

1. Read CHANGELOG.md (or equivalent)
2. Get recent commits: `git log --oneline -20`
3. Identify commits that represent user-facing changes (features, fixes, breaking changes)
4. Flag commits with no corresponding CHANGELOG entry as `undocumented`
5. Check CHANGELOG entries reference correct versions and dates

### Phase 5: Produce Report

Structure results by severity:

```
## Documentation Sync Report

### Critical Issues (docs actively mislead)
- {issues where docs directly contradict code}

### High Issues (docs reference non-existent things)
- {removed exports, broken paths, wrong versions}

### Medium Issues (docs incomplete or stale)
- {missing features, stale examples}

### CHANGELOG Status
- Documented: N commits
- Undocumented: N commits (list them)

### Files Analyzed
- Source files changed: N
- Related docs found: N
- Issues detected: N
```

## Issue Format

```
### [SEVERITY] [CATEGORY] doc-file:line

**Source File:** {the code file that changed}
**Issue:** {what's wrong with the documentation}
**Evidence:** {the specific text in the doc that's incorrect}
**Current Reality:** {what the code actually does}
**Fix:** {exact text change needed}
```

## Constraints

- Only report issues you can verify against actual code -- no speculation
- Distinguish between "outdated" (was once correct) and "never correct" (documentation error)
- Do not auto-fix documentation -- report issues for human review
- Focus on accuracy over completeness -- missing docs are less harmful than wrong docs
- Skip generated documentation (API docs from code comments) -- those are downstream of source changes
- When checking CHANGELOG, only flag commits that represent user-facing changes

## Related Skills

- `drift-detection` -- Broader drift analysis (sync-docs focuses on documentation accuracy)
- `review-code-quality` -- Code quality (sync-docs focuses on doc quality)
- `deslop` -- Catches stale comments and doc references as slop patterns

## Completion Token

When this skill completes successfully, output: `[SKILL_COMPLETE:sync-docs]`
