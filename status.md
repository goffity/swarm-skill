# Status Workflow

Show current swarm session, panes, worktrees, and branches.

## Step 1: Load Config

See [config-reference.md](./config-reference.md) — get session_name.

## Step 2: Display Status

Run these commands and display results:

```bash
echo "=== Tmux Session ==="
tmux has-session -t "$SESSION_NAME" 2>/dev/null && echo "  $SESSION_NAME: ACTIVE" || echo "  $SESSION_NAME: NOT FOUND"

if tmux has-session -t "$SESSION_NAME" 2>/dev/null; then
  echo ""
  echo "=== Panes ==="
  tmux list-panes -t "$SESSION_NAME" -F "  Pane #{pane_index}: #{pane_current_command} (#{pane_width}x#{pane_height})"
fi

echo ""
echo "=== Git Worktrees ==="
git worktree list

echo ""
echo "=== Swarm Branches ==="
git branch --list "swarm/*"
```
