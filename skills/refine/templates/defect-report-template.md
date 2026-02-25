# Defect Report: {Title}

**Status:** {New | Assigned | In Progress | Fixed | Verified | Closed | Reopened | Deferred | Won't Fix | Duplicate}
**Owner:** {Name of the developer or team member responsible for investigating and resolving this defect}
**Date:** {YYYY-MM-DD}
**Defect ID:** {A unique identifier assigned to this defect for tracking and reference purposes. Follow your team's naming convention or use your bug tracking system's auto-generated ID.}
**Severity:** {Critical | High | Medium | Low}
**Priority:** {Critical | High | Medium | Low}

---

## 1. Defect Identification

### Reported By

{Name of the tester, developer, or user who discovered and reported the defect, along with the date of discovery.}

### Defect ID

> Example: BUG-2024-1234 or JIRA-456

### Title / Summary

{A clear, concise title (1-2 sentences) that describes the defect in a way that allows team members to quickly understand the issue without reading the full report.}

> Example: Login button unresponsive on iOS Safari 15.x when form validation fails

---

## 2. Defect Classification

### Severity

{The technical impact of the defect on system functionality. Choose one:}

- **Critical** — system crash, data loss, security breach
- **High** — major feature broken, no workaround
- **Medium** — feature partially impaired, workaround exists
- **Low** — minor issue, cosmetic

### Priority

{The business urgency for fixing this defect. Choose one:}

- **Critical** — fix immediately
- **High** — fix in current sprint/release
- **Medium** — fix in next release
- **Low** — fix when time permits

### Category / Type

{Classification: Functional, UI/UX, Performance, Security, Data, Integration, Configuration, Documentation, or Other.}

### Phase Injected

{The development phase during which this defect was introduced (e.g., Requirements, Design, Development, Testing, Deployment).}

---

## 3. Environment Details

### Build / Version

{The release build number or version of the software in which this defect was detected. Include specific version numbers, build tags, or commit hashes.}

> Example: v2.3.1-beta (build 4567) or commit SHA abc123def

### Environment

{Detailed specification of the environment where the bug was discovered: operating system (with version), browser (with version), device type, network conditions, database version, and any other relevant configuration details.}

> Example: macOS 13.2, Safari 15.6.1, iPhone 13 Pro, WiFi connection, PostgreSQL 14.5

### Test Data

{Description of specific test data, user accounts, or configuration settings required to reproduce the defect.}

> Example: Test account user@test.com with admin role, product SKU "ABC-123" in cart

---

## 4. Defect Description

### Steps to Reproduce

{Step-by-step instructions that allow developers to reproduce the defect consistently. Number each step clearly. Include all setup steps, navigation paths, and actions taken. Be specific about data entered, buttons clicked, and sequence of operations.}

> Example:
> 1. Navigate to login page
> 2. Enter invalid email format (e.g., "notanemail")
> 3. Click "Submit" button
> 4. Observe validation error appears
> 5. Correct email to valid format
> 6. Click "Submit" button again

### Expected Behavior

{A clear description of what should happen when following the steps above. Use Given-When-Then format for precision. Reference requirements, specifications, or design documents when applicable.}

> Example: Given the user corrects the email to valid format, When they click "Submit" again, Then the button should become enabled, submit the form to the server, and redirect to dashboard on success.

### Actual Behavior

{A precise description of what actually happened, contrasting with the expected behavior above. Include error messages (full text), unexpected outputs, system behavior, and any visible symptoms.}

> Example: Given the user corrects the email, When they click "Submit", Then the button remains disabled and unresponsive. Console shows "TypeError: Cannot read property 'valid' of undefined". No form submission occurs.

### Reproducibility

{Indicate whether this defect occurs:}

- **Always** — 100% of attempts
- **Often** — >50%
- **Sometimes** — <50%
- **Rarely** — isolated occurrence
- **Once** — cannot reproduce

---

## 5. Supporting Evidence

### Attachments

{Screenshots, screen recordings, log files, stack traces, network traces, or any other artifacts that help illustrate or diagnose the defect. Reference each attachment by filename and explain what it shows.}

> Example: screenshot-error-state.png shows the disabled button, console-log.txt contains the full stack trace

### Related Defects

{Links or references to related bug reports, duplicate issues, or dependent defects. Note whether this is a duplicate, blocks/is blocked by, or relates to other defects. Document any upstream or downstream service impact — services that may have caused this defect or services that are degraded as a result.}

> Example: Duplicate of BUG-1230, Blocks BUG-1235 (checkout flow), Related to BUG-1220 (form validation library upgrade). Downstream impact: checkout service returns 500 errors when this defect triggers.

---

<!-- lifecycle-phase: resolution -->
## 6. Fix Description

### Solution Implemented

{Detailed description of the code changes, configuration updates, or design modifications made to resolve this defect. Reference specific files, functions, classes, or components that were modified.}

> Example: Updated form validation handler in LoginForm.tsx to properly re-enable button after validation state changes. Added null check before accessing validation.valid property.

### Fix Approach

{Explanation of why this particular solution was chosen. Were alternative approaches considered? What trade-offs were evaluated?}

### Code Changes

{List of files modified, added, or deleted. Include commit hash or pull request number if applicable. Summarize the nature of changes in each file.}

> Example: Modified LoginForm.tsx (lines 45-52), added test case in LoginForm.test.tsx. Commit: abc123def, PR #456

---

<!-- lifecycle-phase: resolution -->
## 7. Root Cause Analysis

### Investigation Findings

{Summary of the investigation process. What code areas were examined? What debugging techniques were used? What hypotheses were tested?}

> Example: Examined form validation lifecycle, used Chrome DevTools to trace state changes, tested hypothesis that validation state wasn't propagating to button component.

### Root Cause

{The fundamental reason why this defect exists. Identify the specific code module, logic error, integration point, configuration issue, or design flaw that caused the defect.}

> Example: The button's enabled state was bound to initialValidation object, which was undefined after form reset. The validation library returns null on reset, but code expected an object with a 'valid' property.

### Contributing Factors

{Any secondary factors that enabled or exacerbated this defect (e.g., missing validation, inadequate testing, unclear requirements, technical debt, time pressure).}

> Example: Missing unit tests for form reset flow, insufficient type safety in validation library integration, rushed implementation during sprint deadline.

---

<!-- lifecycle-phase: resolution -->
## 8. Testing & Verification

### Test Strategy

{Description of how the fix should be tested and verified. What test cases need to be executed? What scenarios must be validated? What edge cases should be checked?}

> Example: Test form submission after validation failure, test form reset behavior, test with various invalid input formats, verify button state changes correctly in all scenarios.

### Verification Results

{Confirmation that the defect no longer reproduces and that the fix works as expected. Include test execution date, tester name, and build/version tested.}

> Example: Verified on 2024-03-15 by QA Team. Tested in v2.3.2-beta (build 4580). Defect no longer reproduces after 50 test attempts across iOS Safari, Chrome, and Firefox.

### Regression Considerations

{Assessment of whether this fix could impact other areas of the system. What regression testing was performed? Were any new issues introduced by this fix?}

> Example: Ran full login flow regression suite (45 test cases), all passed. No new issues detected. Validated signup and password reset flows use different validation logic.

### Impact Analysis

{Evaluation of which features, modules, or user workflows could be affected by this fix. What areas require additional testing or monitoring after deployment?}

> Example: Could affect other forms using the same validation library. Recommended testing: signup form, profile edit form, checkout form. Monitor form submission error rates for 48 hours post-deployment.

---

<!-- lifecycle-phase: resolution -->
## 9. Resolution Details

### Resolution Date

{Date when the defect was marked as fixed/resolved.}

> Example: 2024-03-15

### Fix Version

{The build/version/release in which this fix will be deployed to production.}

> Example: v2.4.0 (scheduled for 2024-03-20 release)

---

## 10. Workaround

{If a temporary workaround exists, document it here. Cover three areas:}

1. **End-User Workaround:** {Step-by-step instructions users can follow to avoid or mitigate the defect until the fix is deployed.}
2. **Support Team Guidance:** {How support staff should identify related tickets, triage incoming reports, and communicate status to affected users.}
3. **Customer Communication Protocol:** {For customer-facing defects: who communicates (support, product, engineering), through what channel (status page, email, in-app banner), and at what cadence. If the defect is internal-only, state "Not customer-facing — no external communication required."}

> Example: (1) Users can refresh the page after entering valid credentials to re-enable the button. Alternatively, use Chrome or Firefox where issue does not occur. (2) Support should tag related tickets with "login-ios-safari" and link to this defect. Provide affected users with the refresh workaround. (3) Post status note on status.example.com; update daily until fix is deployed.

---

## 11. Lessons Learned

{Capture process improvements, testing gaps identified, or development practices that could prevent similar defects in the future. For Critical or High severity defects, this section is required — include at least one concrete preventive action with an assigned owner and target completion date.}

> Example: Need to add integration tests for form library edge cases (Owner: @backend-lead, Target: 2024-04-01), improve TypeScript typing for validation state objects, add pre-merge checklist item for form validation testing.

---

## 12. Open Questions

{List unresolved investigation items, pending decisions, or areas requiring further analysis. For defects, this captures things like: unclear root cause hypotheses, untested environments, potential related defects not yet confirmed.}

| # | Question | Owner | Target Date | Status |
|---|----------|-------|-------------|--------|
| 1 | {Unresolved investigation item or pending decision} | {Who can answer} | {Date} | {Open/Resolved} |

---

## 13. Verification Checklist

Before marking this defect report as complete and closing the issue, verify:

- [ ] Defect can be consistently reproduced following the documented steps
- [ ] Root cause has been identified and documented with supporting evidence
- [ ] Fix has been implemented and code changes are documented with commit references
- [ ] All test scenarios pass including positive cases, edge cases, and regression tests
- [ ] Fix has been verified in the target environment by QA team
- [ ] No new defects were introduced by this fix (regression testing completed)
- [ ] Appropriate stakeholders have signed off on the resolution
- [ ] Customer-facing defects have communication protocol documented
- [ ] Critical/High severity defects have preventive actions in Lessons Learned (with owner and target date)

---

## 14. Glossary

{Define domain-specific terms, acronyms, or technical concepts used in this document. List alphabetically.}

---

## 15. References

{List related documents, research, external resources, and supporting materials. Use markdown links with descriptive text.}

- {Requirements documents, design specifications, user stories}
- {Support tickets or related documentation}

> Example: User Story US-456, Design Spec "Login Flow v2", Support Ticket SUPP-7890

---

## 16. Approval & Sign-off

| Role | Name | Date | Status |
|------|------|------|--------|
| {Role} | {Name} | {Date} | {Pending/Approved/Rejected} |
