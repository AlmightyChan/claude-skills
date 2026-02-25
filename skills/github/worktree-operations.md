# Worktree Operations

Reference: `.claude/rules/worktree-conventions.md` defines the base path, branch rules, and lifecycle. This file covers operational patterns.

---

## Create a Worktree

```bash
# Standard creation
git worktree add .claude/worktrees/<branch-name> -b <branch-name>

# From an existing branch
git worktree add .claude/worktrees/<branch-name> <existing-branch>
```

**Pre-flight checks:**
1. Verify `.claude/worktrees/` is in `.gitignore`: `git check-ignore -q .claude/worktrees/`
2. Check concurrent cap: `git worktree list | wc -l` (max 4 active)
3. Confirm branch name follows conventions (see worktree-conventions.md)

**On failure:** If the branch already exists or the working tree is dirty, ask the user. Do not force-create.

---

## List Active Worktrees

```bash
git worktree list
```

Correlate with active agents (check `iterate-status.md` or build-status files) to identify which worktrees are in use vs. stale.

---

## Cleanup a Worktree

After merge and verification:

```bash
git worktree remove .claude/worktrees/<branch-name>
git branch -d <branch-name>
```

**Force cleanup** (only with user confirmation):
```bash
git worktree remove --force .claude/worktrees/<branch-name>
git branch -D <branch-name>
```

---

## Stale Worktree Audit

```bash
# List all worktrees
git worktree list

# Prune references to deleted worktrees
git worktree prune
```

If a worktree exists but has no associated active agent or open branch, it is stale. Present the list to the user and offer cleanup.

---

## Parallel Work Patterns

### Competitive Pattern

Two builders tackle the same problem independently in separate worktrees. Validator picks the better solution.

```
Task(builder-a, isolation: "worktree", run_in_background: true)
Task(builder-b, isolation: "worktree", run_in_background: true)
# Wait for both → validate → merge winner
```

### Divide-and-Conquer Pattern

Split work across files/modules. Each builder gets a disjoint file set in its own worktree.

```
Task(builder-auth, isolation: "worktree", run_in_background: true)   # auth module
Task(builder-ui, isolation: "worktree", run_in_background: true)     # UI module
# Wait for both → merge sequentially → integration build
```

**File-overlap guard:** Extract target files from each task. If any file appears in multiple tasks, serialize those tasks. Only parallelize tasks with completely disjoint file sets.

### Writer/Reviewer Pattern

Builder works in a worktree. Validator reads the worktree branch (read-only) and provides feedback. Builder iterates in the same worktree.

---

## Task Tool Integration

The `Task` tool with `isolation: "worktree"` creates and manages worktrees automatically:

```
Task({
  description: "Build auth module",
  prompt: "...",
  subagent_type: "builder",
  isolation: "worktree",
  run_in_background: true
})
```

The Task tool handles creation and cleanup. The orchestrator handles merge-back per the merge procedure in worktree-conventions.md.

### CLI Flag

The `--worktree` flag on `claude` CLI creates a worktree for the entire session:

```bash
claude --worktree "implement the auth module"
```

---

## Merge-Back Procedure

1. **Pre-merge build:** Build in the worktree to confirm it compiles.
2. **Merge:** From the main tree: `git merge <branch>`
3. **Conflict resolution:** If conflicts, present diff to user. Do not auto-resolve.
4. **Post-merge integration check:** Build from main tree after merge.
5. **Cleanup:** `git worktree remove` + `git branch -d`

---

## Xcode Worktree Builds

Per `.claude/rules/xcode-mcp.md` and `.claude/rules/worktree-conventions.md`:

```bash
xcodebuild build -workspace .claude/worktrees/<branch>/App.xcworkspace \
  -scheme App -derivedDataPath .claude/worktrees/<branch>/DerivedData 2>&1 | xcsift
```

MCP build tools (`BuildProject`, `RunSomeTests`) cannot target worktrees. Use `xcodebuild` CLI exclusively for worktree builds.
