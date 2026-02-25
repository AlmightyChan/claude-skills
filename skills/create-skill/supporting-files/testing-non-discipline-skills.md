# Testing Non-Discipline Skills

**Load this reference when:** testing technique, pattern, or reference skills — skills that don't enforce rules agents want to break.

## Overview

Not all skills need pressure testing. Discipline-enforcing skills (TDD, verification) need adversarial scenarios because agents have incentives to bypass them. Other skill types need different test approaches focused on correct application rather than compliance under pressure.

## Technique Skills (How-To Guides)

**Examples:** condition-based-waiting, root-cause-tracing, defense-in-depth

**Test with:**
- **Application scenarios**: Can the agent apply the technique correctly to a realistic problem?
- **Variation scenarios**: Does the agent handle edge cases and non-standard situations?
- **Gap testing**: Are the instructions complete enough? Does the agent get stuck at any step?

**Success criteria:** Agent successfully applies technique to a new scenario without requiring clarification.

**Example test:**
```
Scenario: A test suite has 3 flaky tests that pass/fail inconsistently.
Task: Apply the condition-based-waiting technique to stabilize them.
Expected: Agent identifies timing dependencies and replaces with event-based waiting.
```

## Pattern Skills (Mental Models)

**Examples:** reducing-complexity, information-hiding, flatten-with-flags

**Test with:**
- **Recognition scenarios**: Does the agent recognize when the pattern applies?
- **Application scenarios**: Can the agent use the mental model to solve a problem?
- **Counter-examples**: Does the agent know when NOT to apply the pattern?

**Success criteria:** Agent correctly identifies when the pattern applies AND when it doesn't.

**Example test:**
```
Scenario A: A function has 6 nested if/else blocks with duplicate logic.
Expected: Agent recognizes flatten-with-flags pattern applies.

Scenario B: A function has 3 clear, non-overlapping conditions.
Expected: Agent recognizes the pattern does NOT apply — conditions are already clean.
```

## Reference Skills (Documentation/APIs)

**Examples:** API documentation, command references, library guides

**Test with:**
- **Retrieval scenarios**: Can the agent find the right information for a given question?
- **Application scenarios**: Can the agent use what they found correctly?
- **Completeness testing**: Are common use cases covered? Try the 5 most common operations.

**Success criteria:** Agent finds correct information and applies it without errors.

**Example test:**
```
Task: "How do I create a new table with a foreign key using this database library?"
Expected: Agent locates the correct API methods and produces working code.
Watch for: Agent using deprecated APIs or inventing non-existent methods.
```

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Using pressure scenarios for reference skills | Reference skills don't need pressure — test retrieval accuracy instead |
| Testing only happy path | Include edge cases and "what if the agent misidentifies the pattern?" scenarios |
| Skipping gap testing for techniques | If the agent gets stuck on step 3, the instructions have a gap at step 3 |
| Not testing counter-examples for patterns | An agent that applies a pattern everywhere is as bad as one that never uses it |

## Quick Reference

| Skill Type | Test Focus | Key Question |
|------------|-----------|--------------|
| **Technique** | Correct application | Can they follow the steps on a new problem? |
| **Pattern** | Recognition + boundaries | Do they know when to use it AND when not to? |
| **Reference** | Retrieval accuracy | Can they find and correctly apply the information? |
