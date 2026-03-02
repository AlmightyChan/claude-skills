---
version: 1.0.0
description: "Capture gotchas, ideas, and to-dos when triggered by user conversation"
user-invocable: false
---

# Capture Rules

Three types of captures: gotchas, ideas, and to-dos. When triggered, do not act on the item — simply capture it and confirm.

---

## Gotcha Capture

**Trigger:** A mistake, error, poor implementation, or improvement opportunity is identified — through direct experience or user feedback.

**Log file:** `docs/logs/gotcha-log.md`

**Process:**
1. Clean up the phrasing into concise, clear language.
2. Assign a category tag: `workflow`, `implementation`, `config`, `convention`, `tooling`, or `process`.
3. Identify the root cause — not the symptom, but the underlying reason the event occurred.
4. Append a new entry at the top of the log (below the header, above previous entries):

```markdown
## [{tag}] {Title}
**Date:** {YYYY-MM-DD}

**What happened:** {What went wrong, in 1-2 sentences.}

**Root cause:** {Why it happened — the underlying process, convention, or structural gap that allowed it.}

---
```

5. Confirm to the user that the gotcha was captured.

> **Agent access:** Agents with write access (researcher, designer) write directly using the enhanced agent entry format in the log. Agents without write access (critic, validator) include gotchas in their output's `## Gotchas Identified` section; the orchestrator writes them using the agent entry format.

---

## Ideas Capture

**Trigger:** The user shares an idea, brainstorm, or concept — even casually in conversation.

**Log file:** `docs/logs/ideas-log.md`

**Process:**
1. Clean up the phrasing into concise, clear language.
2. Assign a category tag: `app`, `feature`, `enhancement`, `workflow`, `tool`, or `concept`.
3. Append a new entry at the top of the log (below the header, above previous entries):

```markdown
## [{tag}] {Title}
**Date:** {YYYY-MM-DD}

{Clean description of the idea in 1-3 sentences.}

---
```

4. Confirm to the user that the idea was captured.

---

## To-Do Capture

**Trigger:** The user mentions something they need to do, a task they want to remember, or an action item — even casually.

**Log file:** `docs/logs/todo-log.md`

**Process:**
1. Clean up the phrasing into a concise, actionable description.
2. Assign a category tag: `app`, `infra`, `docs`, `design`, `research`, `admin`, or `personal`.
3. Append a new entry at the top of the log (below the header, above previous entries):

```markdown
## [{tag}] {Title}
**Date:** {YYYY-MM-DD} | **Status:** Open

{Clean description of what needs to be done in 1-3 sentences.}

---
```

4. Confirm to the user that the to-do was captured.

**Completion:** When the user completes a to-do, update the entry: add ~~strikethrough~~ to the title, change Status to `Done`, and add `**Completed:** {YYYY-MM-DD}`.

**Review:** When the user asks what they need to do, read the log and summarize all Open items.
