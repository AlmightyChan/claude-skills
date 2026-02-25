# CI Operations

GitHub Actions and CI/CD patterns, including the `claude-code-action` for AI-powered automation.

---

## claude-code-action Setup

### Quick start

1. Install the GitHub App: `/install-github-app` in Claude Code
2. Add the workflow file below to `.github/workflows/`

### @claude Responder Workflow

Responds to `@claude` mentions in PR comments and issues:

```yaml
name: Claude Code
on:
  issue_comment:
    types: [created]
  pull_request_review_comment:
    types: [created]
  issues:
    types: [opened]

jobs:
  claude:
    if: |
      (github.event_name == 'issue_comment' && contains(github.event.comment.body, '@claude')) ||
      (github.event_name == 'pull_request_review_comment' && contains(github.event.comment.body, '@claude')) ||
      (github.event_name == 'issues' && contains(github.event.issue.body, '@claude'))
    runs-on: ubuntu-latest
    permissions:
      contents: write
      pull-requests: write
      issues: write
    steps:
      - uses: anthropics/claude-code-action@v1
        with:
          anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
```

### Auto PR Review Workflow

Automatically reviews all new PRs:

```yaml
name: Claude PR Review
on:
  pull_request:
    types: [opened, synchronize]

jobs:
  review:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      pull-requests: write
    steps:
      - uses: anthropics/claude-code-action@v1
        with:
          anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
          prompt: |
            Review this PR for:
            1. Code correctness and potential bugs
            2. Security vulnerabilities (OWASP Top 10)
            3. Performance concerns
            4. Test coverage gaps
            Provide actionable feedback as inline review comments.
```

### Issue Triage Workflow

Auto-labels and triages new issues:

```yaml
name: Claude Issue Triage
on:
  issues:
    types: [opened]

jobs:
  triage:
    runs-on: ubuntu-latest
    permissions:
      issues: write
    steps:
      - uses: anthropics/claude-code-action@v1
        with:
          anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
          prompt: |
            Triage this issue:
            1. Categorize as bug/feature/question/docs
            2. Assess severity (critical/high/medium/low)
            3. Add appropriate labels
            4. If a bug: suggest which files/modules might be affected
```

### Security Review Workflow

Triggered on PRs touching sensitive files:

```yaml
name: Claude Security Review
on:
  pull_request:
    types: [opened, synchronize]
    paths:
      - 'src/auth/**'
      - 'src/api/**'
      - '*.env*'
      - 'config/**'

jobs:
  security:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      pull-requests: write
    steps:
      - uses: anthropics/claude-code-action@v1
        with:
          anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
          prompt: |
            Security review this PR against OWASP Top 10:
            - A01: Broken Access Control
            - A02: Cryptographic Failures
            - A03: Injection
            - A04: Insecure Design
            - A07: Auth Failures
            Flag any findings as blocking review comments.
```

---

## Permission Recommendations

Minimum permissions per workflow type:

| Workflow | contents | pull-requests | issues |
|----------|----------|---------------|--------|
| PR responder | write | write | write |
| PR review | read | write | - |
| Issue triage | - | - | write |
| Security review | read | write | - |

Always use least-privilege. Only grant `write` where the action needs to modify.

---

## Cost Controls

### max-turns

Limit the number of agentic turns to prevent runaway costs:

```yaml
- uses: anthropics/claude-code-action@v1
  with:
    anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
    max_turns: 10
```

### Timeout

GitHub Actions has a default 6-hour timeout. Set a shorter one:

```yaml
jobs:
  claude:
    runs-on: ubuntu-latest
    timeout-minutes: 15
```

### Concurrency

Prevent parallel runs on the same PR:

```yaml
concurrency:
  group: claude-${{ github.event.pull_request.number || github.event.issue.number }}
  cancel-in-progress: true
```

---

## Monitoring Workflow Runs

```bash
# List recent workflow runs
gh run list --limit 10

# View a specific run
gh run view <run-id>

# View logs for a specific run
gh run view <run-id> --log

# Watch a running workflow
gh run watch <run-id>

# Re-run failed jobs only
gh run rerun <run-id> --failed

# Cancel a running workflow
gh run cancel <run-id>
```

---

## Triggering Workflows from CLI

```bash
# Trigger a workflow dispatch event
gh workflow run <workflow-file> --ref <branch>

# With inputs
gh workflow run deploy.yml -f environment=production -f version=1.2.0

# List available workflows
gh workflow list
```

---

## Scheduled Maintenance Workflow

Example: weekly dependency audit and stale issue cleanup:

```yaml
name: Weekly Maintenance
on:
  schedule:
    - cron: '0 9 * * 1'  # Monday 9am UTC

jobs:
  maintenance:
    runs-on: ubuntu-latest
    permissions:
      contents: write
      issues: write
      pull-requests: write
    steps:
      - uses: anthropics/claude-code-action@v1
        with:
          anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
          prompt: |
            Weekly maintenance:
            1. Check for outdated dependencies
            2. Review issues older than 30 days without activity
            3. Close stale PRs (no activity in 14 days) with a comment
            4. Create a summary issue with findings
```
