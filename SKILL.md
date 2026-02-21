---
name: swarm
description: Launches parallel AI agents in tmux panes with isolated git worktrees for multi-tool brainstorming and development.
argument-hint: "init|start|stop|status|send <prompt>"
---

# Swarm - Multi-Agent Parallel Development

Launch multiple AI coding agents (Claude, Codex, Gemini) simultaneously in a tmux session, each working on its own git worktree branch. Ideal for brainstorming, parallel prototyping, and multi-perspective development.

## Usage

- `/swarm init` - Create `.swarm.json` config interactively in the project root
- `/swarm start` - Create tmux session with all agents on separate worktrees
- `/swarm start "your task description"` - Start and immediately send a prompt to all agents
- `/swarm stop` - Kill session and cleanup worktrees
- `/swarm status` - Show session, worktree, and branch status
- `/swarm send <prompt>` - Send the same prompt to all active agent panes

## Configuration

See [config-reference.md](./config-reference.md) for `.swarm.json` structure and defaults.

## Instructions

### Parse Arguments

Parse `$ARGUMENTS`:
- Empty or `help` → show usage summary and stop
- `init` → see [init.md](./init.md)
- `start` → see [start.md](./start.md) (optional: everything after "start" is the initial prompt)
- `stop` → see [stop.md](./stop.md)
- `status` → see [status.md](./status.md)
- `send` → see [send.md](./send.md) (everything after "send" is the prompt)

## Tmux Key Rules

See [tmux-notes.md](./tmux-notes.md) for critical `send-keys` patterns to prevent hanging.

## Examples

```
/swarm init                                     # Create .swarm.json interactively
/swarm start                                    # Start with default agents
/swarm start "implement user authentication"    # Start and send task to all
/swarm send "now add unit tests"                # Send follow-up to all agents
/swarm status                                   # Check what's running
/swarm stop                                     # Clean up everything
```

## Related Commands

| Command | Purpose |
|---------|---------|
| `/focus` | Set task focus before starting swarm |
| `/td` | Create retrospective after swarm session |
| `/commit` | Commit changes from a specific worktree |
