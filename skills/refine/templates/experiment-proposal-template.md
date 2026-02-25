# Experiment Proposal: {Experiment Name}

**Status:** {Draft | In Review | Approved | Running | Complete}
**Owner:** {Name/Team}
**Date:** {YYYY-MM-DD}

---

## 1. Executive Summary

{Provide a 2-3 sentence overview of what you're testing, why it matters, and what decision this experiment will inform.}

> Example: "We will A/B test a streamlined checkout flow against the current 5-step process to determine if reducing friction increases conversion rate. This experiment will inform whether to invest $200K in full checkout redesign."

---

## 2. Background & Context

{Describe the current situation, problem space, or opportunity that motivates this experiment. Include relevant history, prior attempts, user feedback, or market signals. This section answers: "Why are we doing this experiment now?" 3-5 sentences covering current state, problem evidence, and urgency. Where available, include competitive benchmarks for the primary metric being tested to establish how the current baseline compares to industry or competitor performance.}

> Example: "Our checkout conversion rate has declined from 26% to 22% over the past 6 months. User research identified steps 3-5 as primary friction points, with 62% of abandonments occurring after step 3. Competitors launched streamlined checkouts in Q4 2025, creating pressure to modernize."

---

## 3. Hypothesis

{State the testable hypothesis in the form: "We believe that [change/intervention] will result in [expected outcome] because [reasoning/evidence]." The hypothesis must be falsifiable and specific enough to prove or disprove with the chosen metrics.}

> Example: "We believe that reducing checkout from 5 steps to 2 steps will increase conversion rate by at least 8% because user research shows 62% of cart abandonments occur after step 3."

---

## 4. Research Questions

{List 2-5 specific questions this experiment must answer. Each question should be concrete enough that you can determine when it has been answered. Format as bulleted list.}

- {Research question 1}
- {Research question 2}
- {Research question 3}

> Example: "Does the streamlined flow increase completion rate? Do users trust the simplified flow (measured by support tickets and refund requests)? Which step removal drives the most impact?"

---

## 5. Experiment Design

### Type of Experiment

{Specify the methodology: A/B test, multivariate test, controlled rollout, comparative pilot, observational study, or other. Explain why this approach is appropriate.}

### Variants

- **Control**: {The baseline or current experience}
- **Treatment(s)**: {What changes in each treatment variant, and how it differs from control}

{Ensure variants differ in only one variable to enable clear attribution.}

### Randomization Unit

{Specify how users/entities will be assigned to variants (e.g., user ID, session, device, organization). Justify this choice based on consistency needs and attribution requirements.}

> Example: "User ID, because we want consistent experience across sessions and devices for each user to avoid confusion."

---

## 6. Out of Scope

{Explicitly state what this experiment will NOT test, to prevent scope creep and clarify boundaries. List 3-6 specific exclusions.}

- {Feature or change excluded 1}
- {Feature or change excluded 2}
- {Related variant deferred to future experiment}
- {User segment or market excluded}

> Example: "This experiment does NOT test: pricing changes, mobile app experience (web only), post-purchase upsells, or international markets (US only)."

---

## 7. Target Audience & Scope

### Participant Selection

{Define who will be included. Use specific, measurable inclusion/exclusion criteria.}

> Example: "Include: US users, logged-in accounts, cart value > $50. Exclude: enterprise customers, users with active discount codes, repeat purchasers within 30 days."

### Sample Size

{State the required sample size per variant and how it was calculated. Include: baseline conversion rate, minimum detectable effect (MDE), statistical power (typically 80%), and confidence level (typically 95%).}

> Example: "N=12,500 per variant. Baseline conversion: 22%, MDE: 8% relative lift (1.76 percentage points), 80% power, 95% confidence. Calculated using standard two-proportion z-test."

### Duration

{Specify start date, end date, and total duration. Justify the length based on sample size requirements and seasonality.}

> Example: "March 10-24, 2026 (14 days). Requires 14 days to achieve N=12,500 per variant at current traffic (1,800 eligible users/day). Avoids March 1-7 (first week of month shows atypical behavior)."

---

## 8. Success Criteria & Metrics

### Primary Metric

{Define the single most important metric that determines experiment success. Specify the measurement method, target threshold, and decision rule.}

> Example: "Checkout conversion rate (completed purchases / initiated checkouts). Target: 8% relative lift. Decision: Ship if p < 0.05 and lift ≥ 8%."

### Secondary Metrics

{List 2-4 additional metrics that provide supporting evidence or measure side effects. Include measurement method for each.}

- {Secondary metric 1}: {How measured}
- {Secondary metric 2}: {How measured}

### Guardrail Metrics

{Identify 3-5 metrics that must NOT regress during the experiment. These protect against unintended negative consequences (e.g., load time, error rate, user retention). Specify acceptable threshold for each. Include at least one economic guardrail metric (e.g., revenue per user, LTV, AOV) to ensure the experiment does not optimize engagement or conversion at the expense of unit economics.}

- {Guardrail metric 1}: {Must stay < threshold}
- {Guardrail metric 2}: {Must not drop > threshold}

> Example: "Page load time must stay < 2.5s, error rate < 0.5%, 7-day retention must not drop > 2pp, refund rate < 3%."

---

## 9. Data Collection Plan

### Instrumentation

{Describe what events, logs, or measurements will be captured. Specify tracking implementation, data pipelines, and validation checks. List specific events to track and where data flows.}

> Example: "Track checkout_started, step_completed, purchase_completed events via Segment -> Snowflake. Validate tracking in staging environment before launch. Add variant_id to all events."

### Counterfactual Logging

{Explain how you'll identify which users were exposed to which variant. Specify the logging mechanism and data schema.}

> Example: "Log assignment events to experiment_assignments table (user_id, experiment_id, variant_id, timestamp). Join with event stream for attribution."

### Bias Detection

{Document pre-experiment checks to detect randomization bias: A/A analysis, seedfinder testing, or variance reduction techniques. Specify what checks will be run and acceptance criteria.}

> Example: "Run A/A test for 48 hours pre-launch. Acceptance: p-value > 0.5 on conversion rate, <5% variance in sample size per variant."

---

## 10. Analysis Approach

### Statistical Method

{Specify the analysis technique: t-test, chi-square, regression, Bayesian inference, or other. State the significance threshold and whether one-tailed or two-tailed.}

> Example: "Two-proportion z-test, two-tailed, α = 0.05. Testing for difference in either direction."

### Confidence & Effect Size

{Define the confidence level (typically 95%) and minimum effect size worth detecting (MDE). Specify both absolute and relative effect sizes.}

> Example: "95% confidence level. MDE: 1.76pp absolute increase (8% relative lift from 22% baseline). Smaller lifts are not worth the implementation cost."

### Analysis Timeline

{State when interim analyses will occur and when final analysis will be conducted. Specify stopping rules if the experiment reaches significance early or shows harmful effects.}

> Example: "Day 7: interim check for guardrail violations only (no peeking at primary metric). Day 14: final analysis. Early stop if any guardrail degrades > 5pp."

---

## 11. Resource Requirements

### Engineering

{List development work needed: feature implementation, instrumentation, infrastructure changes. Include estimated effort in person-days or sprints.}

- {Task 1}: {Effort estimate}
- {Task 2}: {Effort estimate}

### Design

{Specify design deliverables: mockups, prototypes, content, visual assets. Include estimated effort.}

- {Deliverable 1}: {Effort estimate}
- {Deliverable 2}: {Effort estimate}

### Data / Analytics

{Describe analysis resources: analyst time, dashboard creation, statistical modeling. Include estimated effort.}

- {Task 1}: {Effort estimate}
- {Task 2}: {Effort estimate}

### Other Resources

{Note additional needs: legal review, security audit, infrastructure costs, third-party services. Include cost or effort estimates.}

- {Resource 1}: {Cost or effort estimate}

---

## 12. Risk Assessment

### Technical Risks

{Identify implementation risks: performance degradation, integration failures, edge cases, scalability concerns. Rate each (low/medium/high).}

### User Experience Risks

{Assess potential negative user impacts: confusion, frustration, friction, privacy concerns.}

### Business Risks

{Evaluate threats: revenue loss, churn, reputation damage, competitive disadvantage, opportunity cost.}

### Mitigation Strategies

{For each risk identified above, describe preventive measures or contingency plans. Format as risk ID + mitigation approach.}

- **Risk 1**: {Mitigation strategy}
- **Risk 2**: {Mitigation strategy}

> Example: "Technical Risk - Performance: Pre-launch load test with 2x expected traffic. Set 2.5s timeout as kill switch trigger."

---

## 13. Rollback Plan

### Rollback Triggers

{Define 3-5 conditions requiring immediate termination: guardrail metric violations, critical bugs, significant negative feedback. Make each trigger specific and measurable.}

- {Trigger 1}: {Threshold or condition}
- {Trigger 2}: {Threshold or condition}

> Example: "Error rate >0.5%, page load time >2.5s for >5min, conversion rate drops >15pp, >10 support tickets/hour mentioning checkout."

### Rollback Procedure

{Step-by-step instructions for reverting: feature flags to toggle, code to deploy, configs to restore. Number each step in sequence.}

1. {Step 1}
2. {Step 2}
3. {Step 3}

### Communication Protocol

{Who must be notified during rollback and through what channels. Include escalation paths and response time expectations.}

- {Stakeholder group}: {Channel, timing}
- {Escalation path}: {When and how to escalate}

### Recovery Validation

{How to verify successful rollback: metrics to check, user experience validation, system health monitoring. Create verification checklist.}

- [ ] {Check 1}
- [ ] {Check 2}
- [ ] {Check 3}

---

## 14. Rollout Strategy

### Phased Approach

{If gradual rollout: specify phases (e.g., dogfooding/beta users -> 1% -> 5% -> 10% -> full population). State criteria for advancing between phases and minimum dwell time per phase.}

> Example: "Phase 1: Internal users (50) for 2 days. Phase 2: 1% (1,800 users) for 3 days. Advance if error rate <0.5% and no P0 bugs."

### Population Segmentation

{If testing across segments sequentially, explain order and rationale. Specify which segments tested first and why.}

> Example: "Test on US users first (largest market, best instrumentation). Then EU (requires GDPR audit). Asia-Pacific last (lowest traffic, translation dependencies)."

### Monitoring at Each Phase

{Define 3-5 checks at each rollout stage before proceeding. Create phase gate checklist.}

- [ ] {Check 1}
- [ ] {Check 2}
- [ ] {Check 3}

---

## 15. Expected Outcomes

### Positive Outcome

{What success looks like: metrics achieved, insights gained, next steps if hypothesis is confirmed. 2-3 sentences covering decision and action.}

> Example: "Conversion rate increases ≥8% with p<0.05, guardrails hold. Ship to 100% within 2 weeks. Invest in checkout redesign phase 2 (payment method optimization)."

### Negative Outcome

{What failure looks like: hypothesis disproved, what you'll learn from negative results. 2-3 sentences covering decision and learnings.}

> Example: "No significant lift or conversion decreases. Do not ship. Learnings: users need step-by-step guidance; explore alternative hypothesis (trust signals vs. step count)."

### Neutral / Inconclusive Outcome

{How to interpret ambiguous results: insufficient data, confounding factors, need for follow-up experiments. 2-3 sentences covering interpretation and next steps.}

> Example: "Lift is 4-6% (below 8% MDE) but directionally positive. Extend experiment 1 week or re-run with refined treatment. Consider segment-specific rollout if certain cohorts show strong signal."

---

## 16. Learning Objectives

{Beyond the primary hypothesis, state 2-4 broader insights this experiment should produce: user behavior patterns, technical feasibility, market validation, methodology improvements. Format as bulleted list.}

- {Learning objective 1}
- {Learning objective 2}
- {Learning objective 3}

> Example: "Understand which checkout step causes most friction. Validate whether simplified flows reduce trust. Test our A/B testing infrastructure at scale."

---

## 17. Milestones & Checkpoints

Define the key milestones for this experiment. These provide the structure needed for scheduling — the actual dates will be determined during planning.

| Milestone | Description | Success Criteria | Dependencies |
|-----------|-------------|------------------|--------------|
| Proposal approval | {Experiment design reviewed and approved} | {Approval by designated decision-maker} | {None} |
| Implementation complete | {All variants built and instrumented} | {QA sign-off, tracking validated in staging} | {Proposal approval} |
| Pre-launch validation | {A/A test and bias checks pass} | {p-value > 0.5, sample size variance < 5%} | {Implementation complete} |
| Experiment start | {Traffic allocation begins} | {Correct variant assignment confirmed} | {Pre-launch validation} |
| Interim check | {Guardrail metrics reviewed} | {No guardrail violations detected} | {Experiment start} |
| Experiment end | {Required sample size reached} | {Statistical power achieved} | {Interim check} |
| Analysis complete | {Results documented and reviewed} | {Statistical analysis peer-reviewed} | {Experiment end} |
| Decision communicated | {Go/no-go decision shared with stakeholders} | {Decision documented with rationale} | {Analysis complete} |

---

## 18. Decision Framework

{State how results will inform the go-forward decision. For each outcome, specify the decision, decision-maker, and timeline.}

- **If hypothesis confirmed**: {What happens next (full rollout, iteration, expansion)? Who decides? By when?}
- **If hypothesis rejected**: {What alternatives will be considered? Who decides? By when?}
- **If inconclusive**: {What follow-up experiments or data collection is needed? Who decides? By when?}

{Specify decision-makers and approval process for each outcome.}

> Example: "Confirmed: VP Product approves full rollout within 5 days. Rejected: Product team evaluates 3 alternative hypotheses, decision in 2 weeks. Inconclusive: Extend 1 week with PM approval."

---

## 19. Assumptions, Limitations & Constraints

### 19.1 Assumptions

{Document 3-5 known assumptions and limitations: technical debt, data limitations, time pressure, scope boundaries. Explain impact of each.}

- {Assumption/Limitation 1}: {Impact}
- {Assumption/Limitation 2}: {Impact}

### 19.2 Limitations & Constraints

{Document known constraints that limit the experiment's design, execution, or analysis: budget caps, technical limitations, timeline pressures, tool availability, sample size limits, regulatory restrictions, or data access limitations. For each constraint, explain its impact on the experiment and any workarounds.}

> Example: Constraint - A/B testing infrastructure only supports 2 variants simultaneously. Impact: Cannot test 3 treatment variants in parallel. Workaround: Run as sequential experiments or use multivariate statistical framework.

---

## 20. Open Questions

{List unresolved items, pending decisions, or areas requiring further investigation.}

| # | Question | Owner | Target Date | Status |
|---|----------|-------|-------------|--------|
| 1 | {Unresolved question or pending decision} | {Who can answer} | {Date} | {Open/Resolved} |

---

## 21. Verification Checklist

Before submitting this experiment proposal, verify:

- [ ] Hypothesis is falsifiable and includes specific expected outcome with magnitude
- [ ] Sample size calculation is documented with baseline, MDE, power, and confidence level
- [ ] Primary metric has clear decision threshold and statistical test specified
- [ ] All guardrail metrics have defined thresholds and monitoring plans
- [ ] Rollback triggers are specific, measurable, and tied to alerting
- [ ] Resource requirements include effort estimates for all teams involved
- [ ] Timeline includes all phases from approval through decision communication
- [ ] Constraints are documented with impact on experiment design and workarounds
- [ ] Guardrail metrics include at least one economic/revenue metric (revenue, LTV, AOV)

---

## 22. Glossary

{Define domain-specific terms, acronyms, or technical concepts used in this document. List alphabetically.}

---

## 23. References

{List related documents, research, external resources, and supporting materials. Use markdown links with descriptive text.}

---

## 24. Approval & Sign-off

| Role | Name | Date | Status |
|------|------|------|--------|
| {Approver role} | {Name} | {Date} | {Pending/Approved/Rejected} |
