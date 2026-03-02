---
name: review-security
version: 1.0.0
description: "Security review lens for code. Proactively use when auditing auth, authz, injection risks, secrets exposure, input validation, or insecure defaults."
---

# Security Review

- **Agents:** auditor

## Overview

Systematic security review that checks code against known vulnerability categories. Every finding is grounded in specific code evidence -- no speculative warnings.

**Core principle:** Security bugs are logic bugs with higher stakes. The same rigor that finds off-by-one errors finds auth bypasses.

## When to Use

- Auditing authentication or authorization code
- Reviewing input handling at trust boundaries
- Checking for injection risks (SQL, command, template)
- Scanning for secrets exposure or insecure defaults
- Any code that processes user-controlled input
- Pre-release security sweeps

## Review Methodology

### 1. Map Trust Boundaries

Identify where untrusted data enters the system:
- HTTP request parameters, headers, body
- File uploads and user-provided paths
- Database query results used in subsequent operations
- Environment variables and configuration
- Inter-service communication payloads

### 2. Five Review Dimensions

#### A. Authentication & Authorization

- Every protected endpoint has an auth check
- Auth checks cannot be bypassed via parameter manipulation
- Session tokens are validated on every request, not just login
- Password reset flows cannot be exploited (no token reuse, expiry enforced)
- Role checks use allowlists, not denylists

#### B. Input Validation & Output Encoding

- All user input is validated at the trust boundary (type, length, format)
- Output is encoded for its context (HTML, URL, SQL, shell)
- File paths are canonicalized before access (no `../` traversal)
- Content-Type headers match actual content
- Numeric inputs have range checks

#### C. Injection Prevention

- SQL queries use parameterized statements, never string concatenation
- Shell commands never include user input; if unavoidable, use allowlists
- Template rendering escapes user content by default
- Regex patterns are anchored to prevent ReDoS
- Deserialization only accepts known-safe types

#### D. Secrets Management

- No hardcoded credentials, API keys, or tokens in source
- Secrets loaded from environment or secure vault at runtime
- Secrets excluded from logs, error messages, and API responses
- `.env` files are gitignored; `.env.example` contains placeholders only
- JWT secrets are sufficiently random and rotatable

#### E. Insecure Defaults

- Debug mode disabled in production configuration
- CORS policy is restrictive, not `*`
- Error messages do not expose internal details (stack traces, SQL, file paths)
- Default credentials are not present
- TLS is enforced; HTTP redirects to HTTPS

### 3. OWASP Top 10 Cross-Reference

After the dimensional review, cross-reference findings against OWASP Top 10:

| Category | What to Check |
|----------|---------------|
| A01 Broken Access Control | Missing auth checks, direct object references without ownership validation |
| A02 Cryptographic Failures | Weak hashing (MD5/SHA1 for passwords), plaintext sensitive data |
| A03 Injection | String concatenation in queries, eval() with user input |
| A04 Insecure Design | Missing rate limiting, no account lockout, client-side-only validation |
| A05 Security Misconfiguration | Debug mode, default credentials, verbose errors |
| A06 Vulnerable Components | Outdated dependencies with known CVEs |
| A07 Auth Failures | Session tokens in URLs, missing session invalidation |
| A08 Data Integrity | Deserialization of untrusted data, missing integrity checks |
| A09 Logging Failures | Missing audit logs, sensitive data in logs |
| A10 SSRF | Unvalidated URLs in server-side requests |

This extends the OWASP quick reference in `.claude/rules/code-review.md` -- that rule provides the index, this skill provides the methodology.

## Checklist

For each file under review:

- [ ] All user input validated at trust boundary
- [ ] No string concatenation in SQL/shell/template contexts
- [ ] No hardcoded secrets (grep for `password=`, `api_key=`, `secret=`, `sk-`, `ghp_`, `AKIA`)
- [ ] Auth checks present on every protected endpoint
- [ ] Error messages do not expose internals
- [ ] File paths canonicalized before filesystem access
- [ ] Session management follows secure patterns
- [ ] CORS, CSP, and security headers configured restrictively
- [ ] Cryptographic operations use current algorithms (no MD5/SHA1 for security)
- [ ] Sensitive data excluded from logs

## Finding Format

```
### [SEVERITY] [OWASP-CATEGORY] file:line

**Category:** {auth|input-validation|injection|secrets|insecure-default}
**Confidence:** {high|medium|low}
**Description:** {What the vulnerability is}
**Evidence:** {Exact code snippet}
**Impact:** {What an attacker could do}
**Remediation:** {Specific fix with code example}
```

Severity levels: CRITICAL, HIGH, MEDIUM, LOW
- CRITICAL: Actively exploitable with high impact (RCE, auth bypass, data exfiltration)
- HIGH: Exploitable with moderate impact or requires specific conditions
- MEDIUM: Defense-in-depth issue, exploitable in combination with other flaws
- LOW: Best practice violation, minimal direct impact

## Constraints

- Ground every finding in specific code evidence -- file path, line number, exact code
- Do not speculate about vulnerabilities you cannot demonstrate in the code
- Do not duplicate findings already covered by `.claude/rules/code-review.md` -- extend, don't repeat
- Flag items needing runtime verification as "requires execution-level testing" (delegate to validator)
- Prioritize findings by exploitability, not theoretical severity
- False positives erode trust -- when uncertain, note confidence as LOW

## Related Skills

- `review-code-quality` -- General code quality (this skill focuses on security dimension only)
- `deslop` -- Catches hardcoded secrets as a slop pattern
- `test-coverage-review` -- Validates security-relevant test coverage

