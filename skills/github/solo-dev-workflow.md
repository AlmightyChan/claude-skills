# Solo Developer Workflow

The complete lifecycle for a solo developer working on a GitHub repository — from issue through merge, tag, and cleanup. This file is the outer wrapper. Each phase references a supporting file for detail.

The Fix-Issue Flow in `issue-operations.md` is the issue-driven subset. This file is the complete lifecycle that wraps it.

---

## Tier Determination

Choose the workflow tier before starting any work.

| Signal | Tier |
|--------|------|
| Feature, bug fix, multi-file change, anything that warrants tracking | Full Workflow |
| Typo, config tweak, single-file doc edit, trivial one-liner | Lightweight Workflow |

When in doubt, use the Full Workflow. Issues are cheap; missing context is not.

---

## Branch Naming

`{type}/{short-description}` — e.g., `feat/add-dark-mode`, `fix/symlink-path`, `docs/update-readme`

Types mirror conventional commit types: `feat`, `fix`, `refactor`, `chore`, `docs`, `test`.

---

## Full Workflow

For features, bug fixes, and any multi-file or substantial change.

```
Issue → Branch → Code → Commit → Push → PR → CI → Merge → Tag → Cleanup
```

### 1. Create or read the issue

Create a new issue if one does not exist. Read the existing issue if one does.

See `issue-operations.md` → **Create an Issue** or **View Issues**.

### 2. Create a branch

```bash
git checkout main && git pull
git checkout -b feat/short-description
```

Branch from an up-to-date `main`. Never branch from a stale base.

### 3. Implement

Work in small, focused commits. Do not batch unrelated changes into one commit.

### 4. Commit

See `commit-workflow.md` → **Pre-Flight Checks**, **Stage Files**, **Commit**.

Use conventional commits: `type(scope): imperative description`.

```bash
git add path/to/file1 path/to/file2
git commit -m "$(cat <<'EOF'
feat(scope): short description under 72 chars

Optional body explaining why, not what.

Co-Authored-By: Claude <noreply@anthropic.com>
EOF
)"
```

### 5. Push

```bash
git push -u origin feat/short-description
```

See `commit-workflow.md` → **Push to Remote**.

### 6. Create PR

Reference the issue in the PR body using `Closes #<number>` to auto-close on merge.

See `pr-operations.md` → **Create a PR** and `templates/pr-description.md`.

### 7. Wait for CI

```bash
gh pr checks <number> --watch
```

See `pr-operations.md` → **CI Check Status**. Do not merge a PR with failing checks.

### 8. Merge

```bash
gh pr merge <number> --squash --delete-branch
```

See `pr-operations.md` → **Merge a PR**. Squash merge is the default for feature branches.

### 9. Tag

After merge, switch to `main`, pull, and tag per project scoping convention.

```bash
git checkout main && git pull

# BASECAMP meta (agents, skills, rules): vX.Y.Z
git tag -a vX.Y.Z HEAD -m "Scope vX.Y.Z — summary"
git push origin vX.Y.Z

# Project-scoped repos: {project}/vX.Y.Z
git tag -a project/vX.Y.Z HEAD -m "project vX.Y.Z — summary"
git push origin project/vX.Y.Z
```

See `commit-workflow.md` → **Tagging Decision** and **Create and Push a Tag**.

Skip tagging for `chore` and `docs`-only changes with no functional delta.

### 10. Cleanup

```bash
git branch -d feat/short-description
```

The `--delete-branch` flag on `gh pr merge` deletes the remote branch automatically. Delete the local branch manually.

---

## Lightweight Workflow

For typos, config tweaks, and single-file doc edits that do not need an issue.

```
Branch → Code → Commit → Push → PR → Merge → Cleanup
```

### 1. Create a branch

```bash
git checkout main && git pull
git checkout -b docs/fix-typo-in-readme
```

Even for trivial changes, never commit directly to `main`.

### 2. Make the change

One file, one purpose.

### 3. Commit

```bash
git add path/to/file
git commit -m "$(cat <<'EOF'
docs(readme): fix typo in installation section

Co-Authored-By: Claude <noreply@anthropic.com>
EOF
)"
```

### 4. Push and open PR

```bash
git push -u origin docs/fix-typo-in-readme
gh pr create --title "docs(readme): fix typo in installation section" --body "$(cat <<'EOF'
## Summary
- Fix typo: "isntall" → "install" in README.md

Generated with [Claude Code](https://claude.com/claude-code)
EOF
)"
```

### 5. Merge

```bash
gh pr merge <number> --squash --delete-branch
```

### 6. Cleanup

```bash
git checkout main && git pull
git branch -d docs/fix-typo-in-readme
```

Lightweight changes typically do not warrant a tag. Skip tagging unless the change has functional impact.
