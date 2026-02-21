# Start Workflow

Create tmux session with isolated git worktrees and launch all agents.

## Step 1: Prerequisites

Verify required tools. Run each check and report any missing tool:

```bash
command -v tmux >/dev/null 2>&1 || echo "ERROR: tmux not found"
command -v git >/dev/null 2>&1 || echo "ERROR: git not found"
command -v jq >/dev/null 2>&1 || echo "ERROR: jq not found"
git rev-parse --git-dir >/dev/null 2>&1 || echo "ERROR: not in a git repository"
```

If any tool is missing, tell the user and stop.

## Step 2: Load Configuration

See [config-reference.md](./config-reference.md) for loading details.

```bash
GIT_ROOT=$(git rev-parse --show-toplevel)
CONFIG_FILE="$GIT_ROOT/.swarm.json"

if [ -f "$CONFIG_FILE" ]; then
  echo "Loading config from $CONFIG_FILE"
else
  echo "No .swarm.json found — using defaults (claude, codex, gemini)"
fi
```

## Step 3: Check Existing Session

```bash
if tmux has-session -t "$SESSION_NAME" 2>/dev/null; then
  echo "Session '$SESSION_NAME' already exists!"
  echo "Use '/swarm stop' first, or '/swarm status' to inspect."
fi
```

If session exists, tell the user and stop. Do NOT create a duplicate.

## Step 4: Verify Source Branch Exists

```bash
git rev-parse --verify "$SOURCE_BRANCH" >/dev/null 2>&1 || echo "ERROR: branch '$SOURCE_BRANCH' not found"
```

If source branch doesn't exist, tell the user and stop.

## Step 5: Create Git Worktrees

Create a worktree for each agent with a unique branch name using timestamp:

```bash
GIT_ROOT=$(git rev-parse --show-toplevel)
WORKTREE_DIR="$GIT_ROOT/$WORKTREE_BASE"
mkdir -p "$WORKTREE_DIR"

TIMESTAMP=$(date +%Y%m%d-%H%M%S)
```

For EACH agent, run:

```bash
AGENT_NAME="<name>"             # e.g., "claude"
BRANCH_PREFIX="<branch_prefix>" # e.g., "swarm/claude"
BRANCH="${BRANCH_PREFIX}-${TIMESTAMP}"
WORKTREE_PATH="$WORKTREE_DIR/$AGENT_NAME"

# Clean up any existing worktree at this path
if [ -d "$WORKTREE_PATH" ]; then
  git worktree remove "$WORKTREE_PATH" --force 2>/dev/null
fi

# Delete branch if it already exists (unlikely with timestamp)
git branch -D "$BRANCH" 2>/dev/null

# Create worktree with new branch from source
git worktree add -b "$BRANCH" "$WORKTREE_PATH" "$SOURCE_BRANCH"
```

Run `git worktree add` for each agent **sequentially** (not parallel) to avoid git lock conflicts.

## Step 6: Create Tmux Session with Panes

Create the session with the FIRST agent's worktree as initial directory:

```bash
tmux new-session -d -s "$SESSION_NAME" -c "$WORKTREE_DIR/<first_agent_name>"
```

For each REMAINING agent (index 1+), split a new pane:

```bash
tmux split-window -t "$SESSION_NAME" -c "$WORKTREE_DIR/<agent_name>"
tmux select-layout -t "$SESSION_NAME" tiled
```

After all panes are created, apply final tiled layout:

```bash
tmux select-layout -t "$SESSION_NAME" tiled
```

## Step 7: Send Startup Commands

See [tmux-notes.md](./tmux-notes.md) for critical key rules.

For EACH agent (pane index 0, 1, 2, ...):

```bash
# Send the agent's startup command
tmux send-keys -t "$SESSION_NAME:0.$PANE_INDEX" -l "<agent_command>"
sleep 0.3
tmux send-keys -t "$SESSION_NAME:0.$PANE_INDEX" C-m
sleep 0.5
```

## Step 8: Send Initial Prompt (if provided)

If the user provided a prompt after "start" (e.g., `/swarm start "implement auth"`), wait for agents to initialize, then send the prompt to all panes:

```bash
sleep 3  # Wait for agents to start up

for PANE_INDEX in 0 1 2 ...; do
  tmux send-keys -t "$SESSION_NAME:0.$PANE_INDEX" -l "<prompt_text>"
  sleep 0.3
  tmux send-keys -t "$SESSION_NAME:0.$PANE_INDEX" C-m
  sleep 0.3
done
```

## Step 9: Remind .gitignore

Check if worktree directory is in .gitignore:

```bash
GIT_ROOT=$(git rev-parse --show-toplevel)
grep -qF "$WORKTREE_BASE" "$GIT_ROOT/.gitignore" 2>/dev/null
```

If NOT in .gitignore, warn the user:
> Add `<worktree_base>/` to your `.gitignore` to avoid committing worktree files.

## Step 10: Report

Display a summary to the user:

```
Swarm started!

Session: <session_name>
Agents:  <count>

| Agent  | Branch                         | Worktree Path              |
|--------|--------------------------------|----------------------------|
| claude | swarm/claude-20260222-143000   | .worktrees/claude          |
| codex  | swarm/codex-20260222-143000    | .worktrees/codex           |
| gemini | swarm/gemini-20260222-143000   | .worktrees/gemini          |

Attach: tmux attach -t <session_name>
Send:   /swarm send "your prompt"
Stop:   /swarm stop
```
