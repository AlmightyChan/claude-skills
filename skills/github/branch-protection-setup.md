# Branch Protection Setup

One-time setup reference for enabling branch protection on GitHub repositories. Requires GitHub Pro (or any paid plan) — free accounts only get branch protection on public repos.

---

## Discover Repos to Protect

Find all repos with a `main` default branch:

```bash
gh repo list AlmightyChan --json name,defaultBranch \
  --jq '.[] | select(.defaultBranch == "main") | .name'
```

Skip `halcyonofficial` — it has no default branch and cannot receive protection rules.

---

## Protection Tiers

### Tier 1 — Repos with CI (e.g., basecamp-app)

Require PRs, require CI status checks to pass, block force push and deletion.

```bash
gh api \
  --method PUT \
  -H "Accept: application/vnd.github+json" \
  repos/AlmightyChan/{REPO}/branches/main/protection \
  --input - <<'EOF'
{
  "required_status_checks": {
    "strict": true,
    "contexts": ["ci"]
  },
  "enforce_admins": false,
  "required_pull_request_reviews": {
    "required_approving_review_count": 0
  },
  "restrictions": null,
  "allow_force_pushes": false,
  "allow_deletions": false
}
EOF
```

Replace `{REPO}` with the repo name. Replace `"ci"` in `contexts` with the actual job name from your workflow file (the `jobs.<job-id>` key in the YAML).

### Tier 2 — Repos without CI (all others)

Require PRs, block force push and deletion. No required status checks.

```bash
gh api \
  --method PUT \
  -H "Accept: application/vnd.github+json" \
  repos/AlmightyChan/{REPO}/branches/main/protection \
  --input - <<'EOF'
{
  "required_status_checks": null,
  "enforce_admins": false,
  "required_pull_request_reviews": {
    "required_approving_review_count": 0
  },
  "restrictions": null,
  "allow_force_pushes": false,
  "allow_deletions": false
}
EOF
```

---

## API Payload Notes

- `"required_pull_request_reviews": { "required_approving_review_count": 0 }` — do not set this to `null`. A `null` value skips the field entirely; an object with count `0` enables the PR requirement without requiring a human reviewer.
- `"restrictions": null` — required field. Pass `null` to skip push restrictions (allows any collaborator to push via PR).
- `"enforce_admins": false` — repo owner can bypass protection in emergencies without disabling it globally.

---

## Break Glass

Temporarily disable protection on a repo when you need to push directly to `main` in an emergency.

### Disable

```bash
gh api \
  --method DELETE \
  -H "Accept: application/vnd.github+json" \
  repos/AlmightyChan/{REPO}/branches/main/protection
```

### Re-enable

Re-run the appropriate Tier 1 or Tier 2 command from above.

---

## Verify Protection is Active

```bash
gh api \
  -H "Accept: application/vnd.github+json" \
  repos/AlmightyChan/{REPO}/branches/main/protection \
  --jq '{
    pr_required: .required_pull_request_reviews.required_approving_review_count,
    status_checks: .required_status_checks.contexts,
    force_push_blocked: (.allow_force_pushes == false),
    deletion_blocked: (.allow_deletions == false)
  }'
```

---

## Rollback — Remove Protection from All Repos

To remove branch protection from every repo at once:

```bash
gh repo list AlmightyChan --json name,defaultBranch \
  --jq '.[] | select(.defaultBranch == "main") | .name' | \
while read repo; do
  echo "Removing protection from $repo..."
  gh api \
    --method DELETE \
    -H "Accept: application/vnd.github+json" \
    repos/AlmightyChan/$repo/branches/main/protection
done
```
