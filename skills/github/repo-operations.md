# Repository Operations

Operations that extend beyond basic git commands: releases, tags, MCP-based repo management, and post-merge cleanup.

---

## Releases

### Create a release

Follow semver from project conventions: `vMAJOR.MINOR.PATCH`

```bash
# Tag on main
git tag v1.2.0 -m "Release v1.2.0: add biometric auth"

# Push tag
git push origin v1.2.0
```

### Create a GitHub release

```bash
gh release create v1.2.0 --title "v1.2.0" --notes "$(cat <<'EOF'
## What's New
- Added biometric authentication
- Fixed session timeout handling

## Breaking Changes
- None
EOF
)"
```

### Using MCP

```
mcp__github__get_latest_release({
  owner: "{owner}",
  repo: "{repo}"
})
```

### List releases

```bash
gh release list --limit 10
```

```
mcp__github__list_releases({
  owner: "{owner}",
  repo: "{repo}"
})
```

---

## Tags

### Create a tag

```bash
# Annotated tag (preferred for releases)
git tag -a v1.2.0 -m "Release v1.2.0"

# Lightweight tag
git tag v1.2.0
```

### List tags

```bash
git tag --sort=-version:refname | head -10
```

```
mcp__github__list_tags({
  owner: "{owner}",
  repo: "{repo}"
})
```

### Get tag details

```
mcp__github__get_tag({
  owner: "{owner}",
  repo: "{repo}",
  tag: "v1.2.0"
})
```

---

## Post-Merge Branch Cleanup

Use `gh-poi` (if installed) for batch cleanup of merged branches:

```bash
gh poi
```

Or manually:
```bash
# List merged branches
git branch --merged main | grep -v 'main'

# Delete merged local branches
git branch --merged main | grep -v 'main' | xargs git branch -d
```

---

## MCP Repository Tools

### Branches via MCP

```
mcp__github__list_branches({
  owner: "{owner}",
  repo: "{repo}"
})
```

```
mcp__github__create_branch({
  owner: "{owner}",
  repo: "{repo}",
  ref: "feat/new-feature",
  sha: "{base-sha}"
})
```

### Commits via MCP

```
mcp__github__get_commit({
  owner: "{owner}",
  repo: "{repo}",
  sha: "{commit-sha}"
})
```

```
mcp__github__list_commits({
  owner: "{owner}",
  repo: "{repo}",
  sha: "main",
  per_page: 10
})
```

### Fork and Create

```
mcp__github__fork_repository({
  owner: "{owner}",
  repo: "{repo}"
})
```

```
mcp__github__create_repository({
  name: "{repo-name}",
  description: "{description}",
  private: true
})
```
