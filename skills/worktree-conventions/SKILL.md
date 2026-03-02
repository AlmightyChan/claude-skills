---
version: 1.0.0
description: "Worktree creation, naming, isolation, and cleanup conventions for parallel git work"
user-invocable: false
---

# Worktree Conventions

Shared worktree rules for all skills and agents that create or work within git worktrees.

---

## Base Path

All worktrees are created under `.claude/worktrees/` relative to the repository root.

```bash
git worktree add .claude/worktrees/<branch-name> -b <branch-name>
```

**Gitignore requirement:** `.claude/worktrees/` must be in `.gitignore`. Before creating a worktree, verify with `git check-ignore -q .claude/worktrees/`; add if missing.

---

## Branch Derivation and Sanitization

Derive the branch name from context (plan filename, issue ID, feature description):
- Plan file: `docs/plan/add-auth.md` -> `feat/add-auth`
- Issue: `BUG-042` -> `fix/bug-042`
- Feature: "dark mode support" -> `feat/dark-mode-support`

**Sanitization rules:**
- Pattern: `^[a-z0-9][a-z0-9/_-]*$`
- Max length: 50 characters (truncate if needed)
- Branch naming convention: `{type}/{short-description}` (per `docs/conventions/`)

**On failure:** If `git worktree add` fails (branch exists, dirty state, non-git project, shallow clone), ask the user whether to proceed in the current directory or abort. Do not force-create.

---

## Dispatch Path Rule

All agent dispatch prompts MUST include the worktree absolute path as a prefix instruction:

> "Your working directory is {worktree-path}. All file paths in this task are relative to this directory. Use absolute paths prefixed with {worktree-path}/ when reading or writing files."

This applies to both `Task` tool subagents and `SendMessage` teammate prompts. If not in a worktree, use the current working directory.

---

## Xcode Worktree Builds

For Xcode projects in worktrees, use `xcodebuild` CLI with per-worktree `-derivedDataPath`. See `.claude/rules/xcode-mcp.md` for the full hybrid strategy and command examples.

---

## Concurrent Worktree Cap

Do not maintain more than 4 active worktrees simultaneously. This balances parallelism against disk usage and context budget. Check active count before creating: `git worktree list | wc -l`.

---

## Merge-Back and Cleanup

1. **Pre-merge:** Run a full-project build in the worktree to confirm it compiles independently.
2. **Merge:** `git merge <branch>` from the main working tree. If conflicts exist, present the diff for manual resolution.
3. **Post-merge integration check:** Build again from the main tree after merge. Non-overlapping file sets can still cause integration failures when combined.
4. **Cleanup:** After successful merge and verification:
   ```bash
   git worktree remove .claude/worktrees/<branch-name>
   git branch -d <branch-name>
   ```
5. **Stale worktree audit:** Periodically check for abandoned worktrees with `git worktree list`. Remove any that are no longer associated with active work.

---

## Task Tool Integration

When using the `Task` tool with `isolation: "worktree"`, Claude Code handles worktree creation and cleanup automatically. The conventions above still apply to:
- Branch naming (the Task tool generates its own branch name, but manual worktrees follow these rules)
- Dispatch path rules (always include worktree path in prompts)
- Merge-back procedure (orchestrator manages merge after Task completes)
- Concurrent cap (count both manual and Task-managed worktrees)
