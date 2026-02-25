# gh CLI Reference

Advanced patterns for the GitHub CLI. For basic operations, see the operation-specific files.

---

## JSON Output and JQ Patterns

Most `gh` commands support `--json` for structured output and `--jq` for filtering.

### Common patterns

```bash
# PR list with specific fields
gh pr list --json number,title,author,createdAt,labels

# Filter with jq
gh pr list --json number,title,state --jq '.[] | select(.state == "OPEN") | "\(.number): \(.title)"'

# Issue labels as comma-separated
gh issue view 42 --json labels --jq '.labels | map(.name) | join(",")'

# PR review status
gh pr view 123 --json reviews --jq '.reviews[] | "\(.author.login): \(.state)"'

# Count open issues by label
gh issue list --json labels --jq '[.[].labels[].name] | group_by(.) | map({(.[0]): length}) | add'

# Check if PR is mergeable
gh pr view 123 --json mergeable --jq '.mergeable'

# Get PR check status
gh pr checks 123 --json name,state --jq '.[] | select(.state != "SUCCESS") | "\(.name): \(.state)"'
```

### Available JSON fields

```bash
# Discover available fields for any command
gh pr list --json help
gh issue list --json help
gh release list --json help
```

---

## gh api (REST and GraphQL)

### REST API

```bash
# GET request
gh api repos/{owner}/{repo}/pulls/123

# POST request
gh api repos/{owner}/{repo}/issues -f title="Bug report" -f body="Description"

# With pagination
gh api repos/{owner}/{repo}/issues --paginate --jq '.[].title'

# Custom headers
gh api repos/{owner}/{repo} -H "Accept: application/vnd.github.v3+json"
```

### GraphQL

```bash
# Simple query
gh api graphql -f query='
  query {
    repository(owner: "{owner}", name: "{repo}") {
      pullRequests(first: 10, states: OPEN) {
        nodes {
          number
          title
          mergeable
        }
      }
    }
  }
'

# With variables
gh api graphql -f query='
  query($owner: String!, $repo: String!, $number: Int!) {
    repository(owner: $owner, name: $repo) {
      pullRequest(number: $number) {
        title
        body
        reviewDecision
      }
    }
  }
' -f owner="{owner}" -f repo="{repo}" -F number=123
```

### Pagination

```bash
# Auto-paginate all results
gh api repos/{owner}/{repo}/issues --paginate

# With jq filtering on paginated results
gh api repos/{owner}/{repo}/issues --paginate --jq '.[].title'

# Manual pagination
gh api repos/{owner}/{repo}/issues?page=2&per_page=100
```

---

## Workflow and CI Operations

```bash
# List workflows
gh workflow list

# Trigger a workflow manually
gh workflow run <workflow-name> --ref <branch>

# With inputs
gh workflow run deploy.yml -f environment=staging -f version=1.2.0

# View recent runs
gh run list --limit 10

# View a specific run
gh run view <run-id>

# View run logs
gh run view <run-id> --log

# Watch a running workflow
gh run watch <run-id>

# Re-run failed jobs
gh run rerun <run-id> --failed
```

---

## Batch Operations

### Pipe patterns

```bash
# Close all issues with a specific label
gh issue list --label "wontfix" --json number --jq '.[].number' | xargs -I {} gh issue close {}

# Add a label to all open PRs by an author
gh pr list --author "username" --json number --jq '.[].number' | xargs -I {} gh pr edit {} --add-label "needs-review"

# Delete all merged branches
gh pr list --state merged --json headRefName --jq '.[].headRefName' | xargs -I {} git push origin --delete {}
```

### Bulk issue creation

```bash
# From a file (one issue per line: "title|body")
while IFS='|' read -r title body; do
  gh issue create --title "$title" --body "$body"
done < issues.txt
```

---

## gh Extensions

### Notable extensions

```bash
# gh-poi: clean up merged branches interactively
gh extension install seachicken/gh-poi
gh poi

# gh-dash: terminal dashboard for PRs and issues
gh extension install dlvhdr/gh-dash
gh dash

# gh-copilot: AI assistance
gh extension install github/gh-copilot
gh copilot suggest "how to rebase"
```

### Manage extensions

```bash
gh extension list
gh extension install <owner>/<repo>
gh extension upgrade --all
gh extension remove <name>
```

---

## Custom Aliases

```bash
# Create an alias
gh alias set prs 'pr list --json number,title,author --jq ".[] | \"#\(.number) \(.title) (\(.author.login))\"" '

# List aliases
gh alias list

# Use the alias
gh prs
```

---

## Authentication

```bash
# Check auth status
gh auth status

# Login
gh auth login

# Switch accounts
gh auth switch

# View token scopes
gh auth status --show-token
```

---

## Search

### Code search

```
mcp__github__search_code({
  query: "repo:{owner}/{repo} language:swift func handleLogin"
})
```

Or:
```bash
gh search code "repo:{owner}/{repo} func handleLogin" --json path,repository
```

### Repository search

```
mcp__github__search_repositories({
  query: "topic:swift-package stars:>100"
})
```

### User search

```
mcp__github__search_users({
  query: "type:org language:swift"
})
```
