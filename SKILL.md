---
name: spawn-tasks
description: Spawn parallel Claude Code sessions from planned subtasks. Each session gets its own worktree and tmux pane with full task context.
user_invocable: true
---

# Spawn Tasks — Parallel Session Launcher

## When to Use

User triggers this skill after breaking down a project into subtasks in the current conversation. This skill writes task files and spawns independent Claude Code sessions for each subtask.

## Workflow

1. **Collect tasks from conversation context**: Gather all subtasks that were discussed and confirmed with the user. Each task needs: name, goal, context, relevant files, constraints, and acceptance criteria.

2. **Write task files**: For each subtask, create a markdown file at `.tasks/spawn/{task-name}.md` in the current project directory. Create the directory if it doesn't exist.

Task file template:

```markdown
# Task: {task name}

## Goal
{what this subtask should accomplish}

## Context
{architectural background, dependencies, related modules}

## Relevant Files
{list of key file paths}

## Constraints
{consensus docs to follow, technical limitations, things to avoid}

## Acceptance Criteria
{how to verify the task is complete}
```

3. **Spawn sessions**: Choose the spawning method based on the current environment.

**Detect environment first** — run `test -t 0` in Bash. If it succeeds, you are in an interactive terminal; if it fails, you are in a non-terminal environment (e.g., Claude Code desktop app, IDE embedded terminal).

### Method A: Terminal environment (tmux available)

When in an interactive terminal with tmux installed, spawn each task via:

```bash
claude -w "{task-name}" --tmux "$(cat .tasks/spawn/{task-name}.md)"
```

This command:
- `-w "{task-name}"`: Creates an isolated git worktree named after the task
- `--tmux`: Opens in a new tmux pane (uses iTerm2 native panes when available)
- The task file content is passed as the initial prompt

### Method B: Non-terminal environment (Agent fallback)

When `test -t 0` fails or tmux is unavailable (Claude Code desktop app, IDE terminals), use the **Agent tool** to spawn parallel sessions. For each task, launch an Agent with `run_in_background: true` and `isolation: "worktree"`:

```
Agent tool call:
  description: "Task: {task-name}"
  prompt: <contents of .tasks/spawn/{task-name}.md>
  isolation: "worktree"
  run_in_background: true
```

Launch **all agents in a single message** to maximize parallelism. Each agent gets its own worktree automatically. You will be notified when each agent completes — do NOT poll or sleep.

4. **Report**: After all sessions are spawned, tell the user:

   **For Method A (tmux)**:
   - How many sessions were created
   - The tmux session name so they can attach (`tmux attach` or check iTerm2 panes)
   - How to switch between panes (Ctrl+B then number, or iTerm2 Cmd+[ ])
   - Remind them that each session has its own worktree, so changes don't conflict

   **For Method B (Agent fallback)**:
   - How many background agents were launched
   - That results will be reported automatically when each agent completes
   - Each agent works in an isolated worktree, so changes don't conflict

## Important Notes

- **Always confirm with the user** before spawning. Show the task list and ask for approval.
- **Environment detection is automatic** — do not ask the user which method to use. Detect and use the appropriate method silently.
- If in a terminal but tmux is not installed, fall back to Method B (Agent).
- Task files are kept in `.tasks/spawn/` for reference. They are NOT deleted after spawning.
- The task file content should be self-contained — the spawned session has NO access to the main conversation's context.
- Include references to project docs (like `docs/final-consensus/`) in the task context so the spawned session knows to check them.
