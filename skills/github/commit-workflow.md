# Commit Workflow

The full automated commit chain: pre-flight → stage → commit → push → tag → push tag.

---

## Pre-Flight Checks

Run all three before staging anything:

```bash
git status                  # see untracked and modified files
git diff                    # unstaged changes
git diff --cached           # already staged changes
git log --oneline -5        # read recent messages for style
```

Use the log output to match the existing commit message style in the repo.

---

## Stage Files

Stage specific files by name. Never use `git add -A` or `git add .`.

```bash
git add path/to/file1 path/to/file2
```

**Why:** Catch-all staging risks committing secrets, binaries, or lock files that do not belong in the commit.

---

## Commit

Use a HEREDOC to ensure correct formatting. Conventional commits format is required.

```bash
git commit -m "$(cat <<'EOF'
type(scope): imperative description under 72 chars

Optional body explaining why, not what. Wrap at 72 chars.

Co-Authored-By: Claude <noreply@anthropic.com>
EOF
)"
```

**Commit type reference:**

| Type | Use when |
|------|----------|
| `feat` | New capability, new file, new behavior |
| `fix` | Bug fix, broken behavior corrected |
| `refactor` | Code restructure with no behavior change |
| `chore` | Formatting, deps, config with no functional change |
| `docs` | Documentation only |
| `test` | Adding or updating tests |

**Rules:**
- Always create NEW commits. Never amend unless explicitly asked.
- Never skip hooks with `--no-verify` unless explicitly asked.
- Subject line: imperative mood, max 72 chars, no trailing period.

---

## Issue References

Link commits to their tracking issue when one exists.

| Context | Convention | Effect |
|---------|-----------|--------|
| Intermediate commit (work in progress) | `Refs #N` in commit body | Links to issue without closing it |
| Final commit (work complete) | `Fixes #N` or `Closes #N` in commit body | Auto-closes the issue on merge to default branch |

**Example — intermediate commit:**

```bash
git commit -m "$(cat <<'EOF'
feat(auth): add login form validation

Refs #12

Co-Authored-By: Claude <noreply@anthropic.com>
EOF
)"
```

**Example — final commit:**

```bash
git commit -m "$(cat <<'EOF'
feat(auth): complete login flow with error handling

Closes #12

Co-Authored-By: Claude <noreply@anthropic.com>
EOF
)"
```

**Scope:** This applies to shipping commits, not per-cycle TDD commits (which are rapid-fire and don't need issue references).

---

## Push to Remote

Push immediately after committing. Do not wait for the user to ask.

```bash
git push
```

If the branch has no upstream yet:

```bash
git push -u origin <branch-name>
```

**Never force-push to `main` or `master` without explicit user confirmation.**

---

## Tagging Decision

After pushing, determine if the change warrants a semver tag.

**Tag if the change is:**

| Bump | Trigger |
|------|---------|
| Major | Breaking or structural changes — skill format overhaul, agent system redesign, incompatible API change |
| Minor | New capability — new skill, new agent, new hook, new convention, new feature |
| Patch | Small fix — bug fix, typo correction, small adjustment |

**Skip tagging if:**
- The commit type is `chore` or `docs` only (no functional change, no new capability)
- The commit is reformatting or cleanup with no behavior delta

---

## Create and Push a Tag

When a tag is warranted:

```bash
# Get the latest tag
git tag --list | sort -V | tail -1

# Bump appropriately, then tag
git tag v{X.Y.Z} -m "{one-line summary}"

# Push the tag
git push origin v{X.Y.Z}
```

Tag on `main`. If the commit is on a feature branch, tag after merge.

**Example — new skill added (minor bump from v1.3.2):**

```bash
git tag --list | sort -V | tail -1
# → v1.3.2

git tag v1.4.0 -m "feat(skills): add commit-workflow supporting file"
git push origin v1.4.0
```

---

## Amend (Explicit Request Only)

Only amend when the user explicitly asks. Never amend as a default strategy.

```bash
git commit --amend --no-edit       # keep message, update staged content
git commit --amend                 # open editor to change message
git push --force-with-lease        # safer than --force; fails if remote advanced
```

Never force-push `main`/`master` even when amending.
