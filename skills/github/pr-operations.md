# Pull Request Operations

---

## Create a PR

### Using MCP (preferred)

```
mcp__github__create_pull_request({
  owner: "{owner}",
  repo: "{repo}",
  title: "{title}",
  body: "{body}",
  head: "{branch}",
  base: "main"
})
```

### Using gh CLI (fallback)

```bash
gh pr create --title "{title}" --body "$(cat <<'EOF'
{body}
EOF
)"
```

**Before creating:**
1. Ensure branch is pushed: `git push -u origin <branch>`
2. Verify all checks pass locally: build, lint, tests
3. Use the PR template from `templates/pr-description.md`

**PR title rules:**
- Under 70 characters
- Use conventional commit style: `feat: add dark mode`, `fix: resolve auth timeout`
- Details go in the body, not the title

**PR body structure:**
- Summary: 1-3 bullet points of what changed and why
- Test plan: how to verify the changes work
- Footer: `Generated with [Claude Code](https://claude.com/claude-code)`

---

## Review a PR

### View PR details

```bash
# Summary
gh pr view <number>

# With full diff
gh pr diff <number>

# JSON for structured processing
gh pr view <number> --json title,body,reviews,comments,statusCheckRollup
```

### Using MCP

```
mcp__github__pull_request_read({
  owner: "{owner}",
  repo: "{repo}",
  pullNumber: {number}
})
```

### Submit a review

```
mcp__github__pull_request_review_write({
  owner: "{owner}",
  repo: "{repo}",
  pullNumber: {number},
  event: "APPROVE" | "REQUEST_CHANGES" | "COMMENT",
  body: "{review summary}"
})
```

### Inline comments

```
mcp__github__add_comment_to_pending_review({
  owner: "{owner}",
  repo: "{repo}",
  pullNumber: {number},
  path: "{file_path}",
  line: {line_number},
  body: "{comment}"
})
```

**Review checklist:**
- Read the full diff, not just changed files
- Check for security issues (OWASP Top 10 — see `.claude/rules/quality-guards.md`)
- Verify test coverage for new behavior
- Check naming conventions match project style
- Look for unintended file changes (secrets, binaries, lock files)

---

## Update a PR

```
mcp__github__update_pull_request({
  owner: "{owner}",
  repo: "{repo}",
  pullNumber: {number},
  title: "{new title}",
  body: "{new body}"
})
```

### Update branch (rebase on base)

```
mcp__github__update_pull_request_branch({
  owner: "{owner}",
  repo: "{repo}",
  pullNumber: {number}
})
```

Or via CLI:
```bash
gh pr update-branch <number>
```

---

## Merge a PR

```
mcp__github__merge_pull_request({
  owner: "{owner}",
  repo: "{repo}",
  pullNumber: {number},
  merge_method: "squash" | "merge" | "rebase"
})
```

Or via CLI:
```bash
gh pr merge <number> --squash --delete-branch
```

**Merge strategy guidance:**
- `squash`: Default for feature branches. Clean history, single commit on main.
- `merge`: For long-lived branches where commit history matters.
- `rebase`: For small, linear changes. Preserves individual commits without merge commit.

**Pre-merge checks:**
- All CI checks passing: `gh pr checks <number>`
- Required reviews approved
- No merge conflicts: `gh pr view <number> --json mergeable`

---

## CI Check Status

```bash
# Watch checks in real-time
gh pr checks <number> --watch

# Filter to failed checks only
gh pr checks <number> --json name,state --jq '.[] | select(.state != "SUCCESS")'
```

**On failure:** Read the failed check's log to diagnose. Do not re-run without understanding the failure.

---

## Session Linking

Start a Claude session from an existing PR:

```bash
claude --from-pr <number>
```

This loads the PR context (title, body, diff, comments) into the session.

---

## Add PR Comments

```
mcp__github__add_issue_comment({
  owner: "{owner}",
  repo: "{repo}",
  issue_number: {number},
  body: "{comment}"
})
```

### Reply to a review comment

```
mcp__github__add_reply_to_pull_request_comment({
  owner: "{owner}",
  repo: "{repo}",
  pullNumber: {number},
  commentId: {comment_id},
  body: "{reply}"
})
```
