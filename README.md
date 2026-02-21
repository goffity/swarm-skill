# Swarm

Multi-agent parallel development skill for Claude Code. Launches multiple AI coding agents (Claude, Codex, Gemini) simultaneously in a tmux session, each working on its own isolated git worktree branch.

## Prerequisites

- `tmux`
- `git`
- `jq`
- At least 2 AI CLI tools installed (e.g., `claude`, `codex`, `gemini`)

## Quick Start

```bash
/swarm init                                   # Create .swarm.json config
/swarm start                                  # Launch all agents in tmux
/swarm start "implement user authentication"  # Launch and send task to all
/swarm send "now add unit tests"              # Send follow-up to all agents
/swarm status                                 # Check what's running
/swarm stop                                   # Clean up everything
```

## How It Works

1. **Init** creates a `.swarm.json` config at your project root
2. **Start** creates a git worktree per agent, opens a tmux session with tiled panes, and launches each agent in its own pane
3. Each agent works on an isolated branch (`swarm/<agent>-<timestamp>`), so changes never conflict
4. **Send** broadcasts prompts to all agents simultaneously
5. **Stop** kills the session, removes worktrees, and optionally cleans up branches

## Configuration

Create `.swarm.json` at the git root (or use `/swarm init`):

```json
{
  "session_name": "swarm",
  "source_branch": "main",
  "worktree_base": ".worktrees",
  "agents": [
    {
      "name": "claude",
      "command": "claude --permission-mode bypassPermissions",
      "branch_prefix": "swarm/claude"
    },
    {
      "name": "codex",
      "command": "codex --dangerously-bypass-approvals-and-sandbox",
      "branch_prefix": "swarm/codex"
    },
    {
      "name": "gemini",
      "command": "gemini --approval-mode yolo",
      "branch_prefix": "swarm/gemini"
    }
  ]
}
```

| Field | Default | Description |
|-------|---------|-------------|
| `session_name` | `"swarm"` | tmux session name |
| `source_branch` | `"main"` | Branch to create worktrees from |
| `worktree_base` | `".worktrees"` | Directory for git worktrees |
| `agents` | 3 defaults | List of agent configs (`name`, `command`, `branch_prefix`) |

## Commands

| Command | Description |
|---------|-------------|
| `/swarm init` | Create `.swarm.json` interactively |
| `/swarm start [prompt]` | Create tmux session with agent worktrees |
| `/swarm stop` | Kill session and cleanup worktrees |
| `/swarm status` | Show session, panes, worktrees, and branches |
| `/swarm send <prompt>` | Broadcast prompt to all active agents |

## Architecture

```
project/
├── .swarm.json          # Config
├── .worktrees/          # Git worktrees (add to .gitignore)
│   ├── claude/          # Claude's isolated copy
│   ├── codex/           # Codex's isolated copy
│   └── gemini/          # Gemini's isolated copy
└── src/                 # Main working tree
```

Each agent gets:
- Its own git worktree (full repo copy)
- A unique branch (e.g., `swarm/claude-20260222-143000`)
- A dedicated tmux pane

## Tips

- Add `.worktrees/` to your `.gitignore`
- Use `/swarm start "task"` to skip the separate `/swarm send` step
- Run `/swarm status` to see which agents are active before sending prompts
- After a swarm session, compare branches with `git diff swarm/claude-... swarm/codex-...`
