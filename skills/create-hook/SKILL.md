---
name: create-hook
description: Create Claude Code hooks for workflow automation. Proactively use when creating, designing, or configuring hooks, automation, or lifecycle events.
user-invocable: false
version: 0.1.0
---

# Creating a Hook

- **Agents:** builder

When creating a hook, follow this complete process:

## 0. Research Official Documentation
Before proceeding, WebFetch the latest official docs:
- Hooks reference: https://code.claude.com/docs/en/hooks
- Hooks guide: https://code.claude.com/docs/en/hooks-guide

Apply any patterns, fields, or conventions from the official docs.
If the docs describe features not covered below, prefer the official docs.

## 1. Identify the Trigger
Choose when the hook should fire:

| Event | When it fires | Matcher filters | Can block? |
|-------|---------------|-----------------|------------|
| `SessionStart` | Session begins/resumes | `startup`, `resume`, `clear`, `compact` | No |
| `UserPromptSubmit` | Before Claude processes prompt | (none) | Yes |
| `PreToolUse` | Before tool executes | tool name | Yes |
| `PermissionRequest` | Permission dialog appears | tool name | Yes |
| `PostToolUse` | After tool succeeds | tool name | No |
| `PostToolUseFailure` | After tool fails | tool name | No |
| `Stop` | Claude finishes responding | (none) | Yes |
| `SubagentStart` | Subagent spawns | agent type | No |
| `SubagentStop` | Subagent finishes | agent type | Yes |
| `Notification` | Claude sends notification | `permission_prompt`, `idle_prompt` | No |
| `PreCompact` | Before compaction | `manual`, `auto` | No |
| `SessionEnd` | Session terminates | `clear`, `logout`, `other` | No |

## 2. Choose Hook Type
Three hook types available:

| Type | Use case | Decision control |
|------|----------|------------------|
| `command` | Run shell scripts | Exit codes + JSON output |
| `prompt` | Single-turn LLM evaluation | Returns `{ok: true/false}` |
| `agent` | Multi-turn with tool access | Returns `{ok: true/false}` |

## 3. Design the Matcher
Use regex patterns to filter when hooks fire:
- Tool events: match tool name (`Bash`, `Edit|Write`, `mcp__.*`)
- SessionStart: match source (`startup`, `resume`, `clear`, `compact`)
- Notification: match type (`permission_prompt`, `idle_prompt`)
- Omit matcher or use `"*"` to match all occurrences

## 4. Choose Configuration Location
| Location | Scope | Use case |
|----------|-------|----------|
| `~/.claude/settings.json` | All your projects | Personal automation |
| `.claude/settings.json` | Project (version controlled) | Team-shared automation |
| `.claude/settings.local.json` | Project (gitignored) | Local overrides |
| Skill/agent frontmatter | Component lifetime | Component-specific |

## 5. Implement the Handler
For command hooks, handle JSON input on stdin:
```bash
#!/bin/bash
INPUT=$(cat)
COMMAND=$(echo "$INPUT" | jq -r '.tool_input.command')
# Your logic here
exit 0  # or exit 2 to block
```

For prompt/agent hooks, write a verification prompt:
```json
{
  "type": "prompt",
  "prompt": "Evaluate if this action is safe: $ARGUMENTS"
}
```

## 6. Test the Hook
Test manually before deploying:
```bash
echo '{"tool_name":"Bash","tool_input":{"command":"npm test"}}' | ./hook.sh
```

---

# Reference

## Configuration Format

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "jq -r '.tool_input.file_path' | xargs prettier --write"
          }
        ]
      }
    ]
  }
}
```

## Exit Codes (command hooks)
- **Exit 0**: Action proceeds; stdout parsed for JSON
- **Exit 2**: Action blocked; stderr becomes Claude's feedback
- **Other**: Action proceeds; stderr logged only

## JSON Output Format

```json
{
  "hookSpecificOutput": {
    "hookEventName": "PreToolUse",
    "permissionDecision": "deny",
    "permissionDecisionReason": "Use rg instead of grep"
  }
}
```

## Hook Types

### Command Hook
```json
{
  "type": "command",
  "command": "/path/to/script.sh",
  "timeout": 600,
  "async": false
}
```

### Prompt Hook
```json
{
  "type": "prompt",
  "prompt": "Evaluate if Claude should proceed: $ARGUMENTS",
  "model": "haiku",
  "timeout": 30
}
```

### Agent Hook
```json
{
  "type": "agent",
  "prompt": "Verify all tests pass before allowing: $ARGUMENTS",
  "timeout": 60
}
```

**Documentation**: https://code.claude.com/docs/en/hooks

## Completion Token
When this skill completes successfully, output: `[SKILL_COMPLETE:create-hook]`
