# AI Code Quality Benchmarks

Reference data for calibrating validation and review expectations when working with AI-generated code.

## Defect Rates

Source: CodeRabbit 2025 analysis of AI-assisted codebases.

| Metric | Value | Context |
|---|---|---|
| AI code defect rate vs human | 1.7x higher | Measured across production codebases with >50% AI-generated code |
| AI code review burden | +30% more comments per PR | AI PRs require more reviewer effort to reach same quality bar |
| Time-to-merge for AI PRs | +40% longer | Due to additional review cycles |

**Calibration note:** If validation finds zero issues in a non-trivial AI-generated change, increase suspicion — statistically unlikely given 1.7x defect rate.

## Security Failure Rates

Source: Veracode State of Software Security 2024-2025.

| Vulnerability Category | AI Code Failure Rate | Human Baseline |
|---|---|---|
| Overall security scan failure | 45% | ~30% |
| XSS (Cross-Site Scripting) | 86% | ~40% |
| Log injection | 88% | ~35% |
| SQL injection | ~60% | ~25% |
| Input validation gaps | ~70% | ~35% |

**Calibration note:** Security-sensitive code from AI requires dedicated sweep — the default failure rate is near coin-flip.

## Review Burden Metrics

Sources: Faros AI engineering metrics, DORA 2024 report.

| Metric | Before AI Coding Tools | After AI Coding Tools |
|---|---|---|
| PR volume per developer | ~4 PRs/week | ~7 PRs/week |
| Review throughput per reviewer | ~6 PRs/week | ~6 PRs/week (unchanged) |
| Review queue depth | Manageable | Growing — bottleneck shifted to review |
| Defects found in review | ~15% of submitted code | ~25% of submitted code |

**Calibration note:** The bottleneck has shifted from code authoring to code review. Validation thoroughness matters more, not less, in AI-assisted workflows.

## Implications for BASECAMP Agents

- **Validator**: Zero-issue reports on non-trivial AI code should trigger re-examination. Use Step 2.5 (AI code defect sweep) for every AI-assisted change.
- **Critic**: Weight security posture checks heavily — AI code fails security scans at 45%. Expand test quality review — AI tends to generate mock-heavy, happy-path-only tests.
- **TDD skill**: Smaller step sizes reduce blast radius of AI defects. Commit-on-green discipline prevents broken AI code from accumulating.
