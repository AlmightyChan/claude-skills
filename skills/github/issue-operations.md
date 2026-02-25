# Issue Operations

---

## View Issues

### List open issues

```bash
gh issue list --state open --limit 20
```

### Structured JSON output

```bash
gh issue list --json number,title,labels,assignees,createdAt --jq '.[] | "\(.number)\t\(.title)\t\(.labels | map(.name) | join(","))"'
```

### Using MCP

```
mcp__github__list_issues({
  owner: "{owner}",
  repo: "{repo}",
  state: "open",
  per_page: 20
})
```

### View a specific issue

```
mcp__github__issue_read({
  owner: "{owner}",
  repo: "{repo}",
  issue_number: {number}
})
```

Or:
```bash
gh issue view <number>
```

---

## Fix-Issue Flow

End-to-end flow from issue to merged PR:

### 1. Read the issue

```
mcp__github__issue_read({ owner, repo, issue_number })
```

### 2. Create a worktree

Per `.claude/rules/worktree-conventions.md`:

```bash
git worktree add .claude/worktrees/fix/issue-<number> -b fix/issue-<number>
```

### 3. Implement the fix

Work within the worktree. Follow project conventions for code style, testing, and commits.

### 4. Commit and push

```bash
git add <specific-files>
git commit -m "$(cat <<'EOF'
fix(scope): resolve issue #<number>

<description of the fix>

Co-Authored-By: Claude <noreply@anthropic.com>
EOF
)"
git push -u origin fix/issue-<number>
```

### 5. Create PR referencing the issue

Use `Closes #<number>` or `Fixes #<number>` in the PR body to auto-close the issue on merge.

```bash
gh pr create --title "fix(scope): resolve issue #<number>" --body "$(cat <<'EOF'
## Summary
- Fixes #<number>: <brief description>

## Test plan
- [ ] <verification steps>

Generated with [Claude Code](https://claude.com/claude-code)
EOF
)"
```

### 6. Clean up after merge

```bash
git worktree remove .claude/worktrees/fix/issue-<number>
git branch -d fix/issue-<number>
```

---

## Triage Issues

### Label an issue

```
mcp__github__issue_write({
  owner: "{owner}",
  repo: "{repo}",
  issue_number: {number},
  labels: ["bug", "priority:high"]
})
```

### Assign an issue

```
mcp__github__issue_write({
  owner: "{owner}",
  repo: "{repo}",
  issue_number: {number},
  assignees: ["{username}"]
})
```

### Comment on an issue

```
mcp__github__add_issue_comment({
  owner: "{owner}",
  repo: "{repo}",
  issue_number: {number},
  body: "{triage notes}"
})
```

---

## Close an Issue

```
mcp__github__issue_write({
  owner: "{owner}",
  repo: "{repo}",
  issue_number: {number},
  state: "closed"
})
```

With a closing comment:

```bash
gh issue close <number> --comment "Resolved in PR #<pr-number>"
```

---

## Search Issues

```
mcp__github__search_issues({
  query: "repo:{owner}/{repo} is:open label:bug sort:created-desc"
})
```

Or:
```bash
gh issue list --label "bug" --state open --json number,title
```

---

## Create an Issue

```
mcp__github__issue_write({
  owner: "{owner}",
  repo: "{repo}",
  title: "{title}",
  body: "{description}",
  labels: ["{label}"],
  assignees: ["{username}"]
})
```

Or:
```bash
gh issue create --title "{title}" --body "{description}" --label "{label}"
```

---

## Assign Copilot to an Issue

```
mcp__github__assign_copilot_to_issue({
  owner: "{owner}",
  repo: "{repo}",
  issue_number: {number}
})
```

This triggers GitHub Copilot to work on the issue automatically.
