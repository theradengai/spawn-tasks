# spawn-tasks

A [Claude Code](https://claude.ai/code) skill that spawns parallel Claude Code sessions from planned subtasks. Each session gets its own git worktree and tmux pane with full task context.

## What it does

After you break down a project into subtasks in a Claude Code conversation, `/spawn-tasks` will:

1. Write self-contained task files to `.tasks/spawn/` in your project
2. Spawn an independent `claude` session for each task via `claude -w <name> --tmux`
3. Each session gets its own isolated git worktree — no conflicts between parallel sessions

## Installation

Copy `SKILL.md` into your Claude Code skills directory:

```bash
mkdir -p ~/.claude/skills/spawn-tasks
cp SKILL.md ~/.claude/skills/spawn-tasks/SKILL.md
```

Or clone this repo directly:

```bash
git clone https://github.com/<your-username>/spawn-tasks ~/.claude/skills/spawn-tasks
```

## Usage

1. In a Claude Code session, plan and confirm your subtasks with Claude
2. Run `/spawn-tasks`
3. Claude will show you the task list and ask for confirmation before spawning
4. Sessions open as new tmux panes (falls back to Terminal.app windows on macOS if tmux is not installed)

## Requirements

- [Claude Code](https://claude.ai/code) CLI (`claude`)
- tmux (recommended) or macOS Terminal.app
- A git repository (worktrees require git)

## How sessions are isolated

Each spawned session uses `claude -w <task-name>`, which creates a separate git worktree for that task. This means:

- Each session works on its own branch
- File edits don't conflict between sessions
- Sessions can run truly in parallel

## Task file format

Each task gets a file at `.tasks/spawn/{task-name}.md`:

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

The task file is passed as the initial prompt to the spawned session, so it must be fully self-contained.

## License

MIT
