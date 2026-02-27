---
name: review-performance
description: "Performance review lens for code. Proactively use when reviewing for N+1 queries, async blocking, hot path inefficiencies, memory leaks, or unnecessary allocations."
version: 0.1.0
---

# Performance Review

- **Agents:** auditor

## Overview

Performance-focused code review targeting common patterns that cause latency, memory issues, or scalability bottlenecks. Findings are grounded in specific code patterns, not speculative optimization.

**Core principle:** Performance bugs are correctness bugs that manifest under load. Review for them with the same rigor as logic errors.

## When to Use

- Reviewing database query patterns (N+1, missing indexes)
- Checking async code for blocking operations
- Auditing hot paths for unnecessary work
- Investigating memory leak patterns or retain cycles
- Reviewing code that processes large datasets
- Pre-release performance audits

## Review Methodology

### Four Review Dimensions

#### 1. N+1 Queries & Inefficient Loops

- Database queries inside loops (fetch once, process many)
- Sequential API calls that could be batched
- Repeated collection lookups that could be pre-indexed (Map/Dictionary)
- Sorting or filtering operations repeated on the same data
- String concatenation in loops (use builders/arrays)

**Detection pattern:** Find any loop body that contains a database call, network request, or file I/O. Each is a potential N+1.

#### 2. Async Blocking

- Synchronous I/O on async/event loop threads
- Long-running computation on the main/UI thread
- Missing `await` on async operations (fire-and-forget without intention)
- Thread pool exhaustion from blocking operations in async contexts

**Language-specific patterns:**

| Language | Pattern | Issue |
|----------|---------|-------|
| Swift | Heavy work on `@MainActor` | UI thread blocking, dropped frames |
| Swift | Retain cycles in closures (`self` without `[weak self]`) | Memory leak |
| JavaScript | `fs.readFileSync()` in async handler | Event loop blocking |
| JavaScript | `JSON.parse()` on large payloads in request handler | Event loop blocking |
| Python | CPU-bound work in `async def` without executor | GIL contention, event loop blocking |
| Python | `time.sleep()` in async context | Event loop blocking |

#### 3. Hot Path Inefficiencies

- Unnecessary allocations in loops (create once outside, reuse inside)
- Regex compilation inside loops (compile once, match many)
- Excessive logging in high-frequency code paths
- Redundant computation (same value calculated multiple times)
- Collection copying when mutation is not needed

**Detection pattern:** Identify code paths executed per-request, per-frame, or per-iteration. Any allocation, I/O, or computation in these paths is worth examining.

#### 4. Memory & Allocations

- Unbounded caches or collections (missing eviction/size limits)
- Large objects held longer than necessary (scope them tightly)
- Event listeners or observers not cleaned up (onAppear/onDisappear, subscribe/unsubscribe)
- Retain cycles in closures or delegate patterns
- Large intermediate collections created then discarded (use lazy/streaming)

**Language-specific patterns:**

| Language | Pattern | Issue |
|----------|---------|-------|
| Swift | Strong reference in closure capturing `self` | Retain cycle, memory leak |
| Swift | `NotificationCenter` observer without removal | Observer leak |
| Swift | Timer without invalidation | Timer leak |
| JavaScript | Event listener without `removeEventListener` | DOM leak |
| JavaScript | Closure over large scope in long-lived callback | Memory pressure |
| Python | Circular references without `__del__` or weakref | GC pressure |

## Checklist

For each file under review:

- [ ] No database/network calls inside loops
- [ ] No synchronous blocking on async threads
- [ ] No heavy computation on main/UI thread
- [ ] Closures use weak references where appropriate (Swift: `[weak self]`)
- [ ] Observers/listeners paired with cleanup
- [ ] Caches have size limits or eviction policies
- [ ] No regex compilation inside loops
- [ ] No unnecessary collection copying
- [ ] Large datasets processed with pagination or streaming
- [ ] Resource cleanup on all exit paths

## Finding Format

```
### [SEVERITY] file:line

**Category:** {n-plus-one|async-blocking|hot-path|memory}
**Confidence:** {high|medium|low}
**Description:** {What the performance issue is}
**Evidence:** {Exact code snippet}
**Impact:** {Estimated effect -- e.g., "O(n) database queries where O(1) is possible"}
**Remediation:** {Specific fix with approach}
```

Severity levels: HIGH, MEDIUM, LOW
- HIGH: Will cause user-visible latency, OOM, or scalability failure under normal load
- MEDIUM: Noticeable under moderate load or with larger datasets
- LOW: Suboptimal but unlikely to cause problems at current scale

## Constraints

- Only flag performance issues you can demonstrate in the code -- no speculative optimization
- Do not recommend premature optimization for code paths that are not hot
- Distinguish between "slow" and "slow enough to matter" -- context determines severity
- Flag items needing profiling data as "requires runtime measurement" (delegate to specialist)
- Do not rewrite code -- identify the pattern and suggest the approach
- Performance improvements that sacrifice readability need explicit justification

## Related Skills

- `review-code-quality` -- General quality (this skill focuses on performance dimension only)
- `review-architecture` -- Structural patterns that cause systemic performance issues

## Completion Token

When this skill completes successfully, output: `[SKILL_COMPLETE:review-performance]`
