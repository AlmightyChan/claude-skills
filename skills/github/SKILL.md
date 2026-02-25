---
name: github
description: "GitHub workflow operations. Use when creating PRs, reviewing pull requests, managing issues, working with worktrees, creating releases, setting up CI/GitHub Actions, or using gh CLI patterns."
argument-hint: "[operation] [details]"
model: sonnet
version: 0.1.0
---

# GitHub

Unified skill for all GitHub workflow operations. Determine the operation from $ARGUMENTS and the conversation context, then load the appropriate supporting file.

## Current git state

- Branch: !`git branch --show-current`
- Status: !`git status --short | head -20`
- Remote: !`git remote -v | head -2`
- Recent commits: !`git log --oneline -5`

## Operation routing

Read the request and load ONLY the supporting file needed — do not load all files.

| If the request involves... | Load this file |
|---|---|
| Worktrees: create, list, cleanup, parallel work | [worktree-operations.md](worktree-operations.md) |
| Pull requests: create, review, update, merge, checks | [pr-operations.md](pr-operations.md) |
| Issues: view, triage, fix-issue flow, labels | [issue-operations.md](issue-operations.md) |
| Releases, tags, post-merge cleanup, MCP repo tools | [repo-operations.md](repo-operations.md) |
| GitHub Actions, CI/CD, claude-code-action | [ci-operations.md](ci-operations.md) |
| gh CLI: JSON/JQ, gh api, batch ops, extensions | [gh-cli-reference.md](gh-cli-reference.md) |

If the operation is ambiguous, ask the user to clarify. If the request spans multiple categories, load files sequentially as needed.

For PR description formatting, see [templates/pr-description.md](templates/pr-description.md).

## Universal principles

**Tool preference:** Use MCP GitHub tools (`mcp__github__*`) over `gh` CLI when available. Fall back to `gh` CLI if MCP is unavailable. See `.claude/rules/mcp-preferences.md`.

**Commit conventions:** Follow conventional commits format from project conventions: `type(scope): description`. Include `Co-Authored-By: Claude <noreply@anthropic.com>`.

**Branch naming:** `{type}/{short-description}` — e.g., `feat/add-dark-mode`, `fix/symlink-path`.

**Safety rules:**
- Never force-push to `main`/`master` without explicit user confirmation
- Never delete remote branches without user confirmation
- Never skip pre-commit hooks (`--no-verify`) unless explicitly asked
- Stage specific files, not `git add -A`, to avoid committing secrets or binaries
- Always create NEW commits rather than amending unless explicitly asked

**Worktree conventions:** Follow `.claude/rules/worktree-conventions.md` for all worktree operations.

## Completion Token

When this skill completes successfully, output: `[SKILL_COMPLETE:github]`
