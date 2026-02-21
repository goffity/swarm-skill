# Config Reference

## File Location

`.swarm.json` at the git repository root.

## Default Config (when `.swarm.json` is absent)

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

## Fields

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `session_name` | string | `"swarm"` | tmux session name |
| `source_branch` | string | `"main"` | Branch to checkout worktrees from |
| `worktree_base` | string | `".worktrees"` | Directory for git worktrees (relative to git root) |
| `agents` | array | 3 defaults | List of agent configurations |

## Agent Fields

| Field | Type | Description |
|-------|------|-------------|
| `name` | string | Agent identifier (used for worktree directory and pane label) |
| `command` | string | Startup command to run in the agent's pane |
| `branch_prefix` | string | Prefix for the git branch (timestamp appended automatically) |

## Loading Config

```bash
GIT_ROOT=$(git rev-parse --show-toplevel)
CONFIG_FILE="$GIT_ROOT/.swarm.json"

SESSION_NAME=$(jq -r '.session_name // "swarm"' "$CONFIG_FILE" 2>/dev/null || echo "swarm")
SOURCE_BRANCH=$(jq -r '.source_branch // "main"' "$CONFIG_FILE" 2>/dev/null || echo "main")
WORKTREE_BASE=$(jq -r '.worktree_base // ".worktrees"' "$CONFIG_FILE" 2>/dev/null || echo ".worktrees")
```

## Branch Naming

Branches are auto-generated with timestamp to avoid conflicts:

```
<branch_prefix>-<YYYYMMDD-HHMMSS>
```

Example: `swarm/claude-20260222-143000`
