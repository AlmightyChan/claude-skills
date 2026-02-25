# ISSUES.md Format Specification

This file defines the format for per-project ISSUES.md files used by the iterate skill to track bugs, features, and polish items across sessions.

This file is the sole authoritative specification for all ISSUES.md files across every project.

---

## Header Format

When creating a new ISSUES.md for a project that does not have one:

```markdown
# {Project Name} Issues

Tracking bugs, missing features, and UX concerns.

**Last audit:** {YYYY-MM-DD}

---
```

The `Last audit` field is updated whenever a comprehensive review of the issues is performed.

---

## Entry Format

```markdown
## {PREFIX}-{NNN}: {Title}

**Labels:** {category}, {area}, priority/{level}
**Status:** {Open|Resolved|Trashed}

**Description:**
{1-2 paragraph description of the issue}

**Steps to reproduce:** (optional, for bugs with non-obvious reproduction)
1. {Step 1}
2. {Step 2}

**Expected:** {Expected behavior, for bugs}

**Files:** {affected files with line references where relevant}

**Acceptance criteria:**
- [ ] {Criterion 1}
- [ ] {Criterion 2}

**Verification:** (added when marking Resolved)
- **Type:** {Automated|Manual}
- **Command/Steps:** {test command that was run, or manual steps that were confirmed}
- **Result:** {PASS with brief evidence, e.g., "1 test, 0 failures" or "User confirmed 2026-02-23"}

---
```

---

## Prefixes

| Prefix | Use For | Examples |
|--------|---------|---------|
| `BUG-` | Defects, broken behavior, crashes, regressions | BUG-001, BUG-042 |
| `FEAT-` | New features, missing capabilities, integrations | FEAT-005, FEAT-023 |
| `POLISH-` | Cosmetic improvements, UX refinements, code quality | POLISH-001, POLISH-010 |

---

## Priority Levels

| Priority | Use When |
|----------|----------|
| `critical` | App Store blocker, core functionality broken, data loss risk, security vulnerability |
| `high` | Major functionality broken, significant reliability issue, compliance risk |
| `medium` | UX issue, data integrity concern, missing non-critical feature |
| `low` | Code quality, polish, edge cases, nice-to-have improvements |

---

## ID Auto-Increment Rules

When appending a new entry to ISSUES.md:

1. Scan all existing entries for the relevant prefix (BUG-, FEAT-, or POLISH-)
2. Find the highest numeric ID for that prefix
3. Increment by 1 for the new entry

**Example:** If the file contains BUG-001, BUG-005, and BUG-012, the next bug entry is BUG-013.

IDs are never reused. If BUG-005 is Trashed, BUG-005 remains taken. The next bug is still one above the highest existing ID.

IDs are zero-padded to 3 digits (e.g., BUG-001, not BUG-1). If IDs exceed 999, use 4 digits (BUG-1000).

---

## Status Transitions

| From | To | When |
|------|----|------|
| Open | Resolved | Fix verified via automated test (command run, output shown) or manual test (user explicitly confirmed). Verification field populated. |
| Open | Trashed | Duplicate, invalid, won't fix, or no longer relevant |

Resolved and Trashed are terminal states. There is no transition back to Open. If a resolved issue regresses, create a new entry referencing the original (e.g., "Regression of BUG-005").

**Hard rule:** An issue cannot transition to Resolved without a populated Verification field. The Verification field must contain the type (Automated or Manual), the command or steps, and the result. "Code looks correct" is not a valid verification result.

---

## Optional Fields

- **Steps to reproduce** -- Include for bugs where reproduction is non-obvious. Omit for features and polish items.
- **Expected** -- Include for bugs where the expected behavior differs from what happens. Omit for features and polish items where the description and acceptance criteria are sufficient.
- **Files** -- Include when the affected files are known. Omit for broad or exploratory issues where file scope is unclear.
- **Acceptance criteria** -- Strongly recommended for all entries. May be omitted for low-priority code quality items where the fix is self-evident.
- **Verification** -- Added when marking an issue Resolved. Records the proof that the fix works. Must include Type (Automated or Manual), the Command/Steps used, and the Result. This field is required for all Resolved issues -- no issue can be marked Resolved without it.

---

## Section Comments (Optional)

For large ISSUES.md files, priority sections can be separated with HTML comments for readability:

```markdown
<!-- ============================================================ -->
<!-- CRITICAL: APP STORE BLOCKERS & CORE FUNCTIONALITY BROKEN      -->
<!-- ============================================================ -->

{critical entries}

<!-- ============================================================ -->
<!-- HIGH PRIORITY: MAJOR FUNCTIONALITY & RELIABILITY ISSUES        -->
<!-- ============================================================ -->

{high entries}
```

---

## Concrete Example

```markdown
## BUG-005: Restore Purchases button is non-functional

**Labels:** bug, paywall, storekit, priority/critical, app-store-compliance
**Status:** Open

**Description:**
The "Restore Purchases" button in the onboarding paywall (`OnboardingView.swift:805`) has an empty closure: `Button("Restore Purchases") {}`. It does nothing when tapped. Apple requires a functional restore purchases mechanism for all apps with in-app purchases. This will cause App Store rejection.

**Files:** `Views/Onboarding/OnboardingView.swift` (line 805), `Auth/Auth.swift` (restorePurchases() exists but is not connected)

**Acceptance criteria:**
- [ ] Restore Purchases button calls `subscriptionManager.restorePurchases()`
- [ ] Shows loading state during restore
- [ ] Displays success/error feedback to user
- [ ] Also available from Settings (see FEAT-012)

---
```

This example demonstrates: prefixed ID with zero-padded number, multiple labels including priority, Open status, a description that explains both the symptom and the impact, the Files field with line references, and specific acceptance criteria with checkboxes. The Steps to reproduce and Expected fields are omitted because the issue is self-evident from the description.
