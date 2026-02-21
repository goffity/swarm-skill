# Stop Workflow

Kill tmux session and cleanup worktrees.

## Step 1: Load Config

See [config-reference.md](./config-reference.md) — get session_name, worktree_base, and agent names.

## Step 2: Kill Tmux Session

```bash
if tmux has-session -t "$SESSION_NAME" 2>/dev/null; then
  tmux kill-session -t "$SESSION_NAME"
  echo "Killed tmux session '$SESSION_NAME'"
else
  echo "No active session '$SESSION_NAME' found"
fi
```

## Step 3: Remove Worktrees

For each agent:

```bash
WORKTREE_PATH="$WORKTREE_DIR/$AGENT_NAME"
if [ -d "$WORKTREE_PATH" ]; then
  git worktree remove "$WORKTREE_PATH" --force
  echo "Removed worktree: $WORKTREE_PATH"
fi
```

Then prune:

```bash
git worktree prune
```

## Step 4: Ask About Branch Cleanup

List swarm branches:

```bash
git branch --list "swarm/*"
```

Ask the user: "Do you want to delete these swarm branches?"

If yes:

```bash
git branch --list "swarm/*" | xargs -r git branch -D
echo "Deleted all swarm/* branches"
```

If no, leave them for the user to review/merge later.

## Step 5: Report

```
Swarm stopped and cleaned up.
- Session: killed
- Worktrees: removed
- Branches: <deleted|kept>
```
