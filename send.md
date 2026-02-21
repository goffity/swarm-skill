# Send Workflow

Send the same prompt to ALL panes in the swarm session simultaneously.

## Step 1: Load Config

See [config-reference.md](./config-reference.md) — get session_name.

## Step 2: Verify Session Exists

```bash
tmux has-session -t "$SESSION_NAME" 2>/dev/null || echo "ERROR: no active session '$SESSION_NAME'"
```

If no session, tell the user to run `/swarm start` first and stop.

## Step 3: Send Prompt

See [tmux-notes.md](./tmux-notes.md) for critical key rules.

```bash
PROMPT="<everything after 'send'>"

# Count panes
PANE_COUNT=$(tmux list-panes -t "$SESSION_NAME" 2>/dev/null | wc -l | tr -d ' ')

# Send to each pane
for i in $(seq 0 $(($PANE_COUNT - 1))); do
  tmux send-keys -t "$SESSION_NAME:0.$i" -l "$PROMPT"
  sleep 0.2
  tmux send-keys -t "$SESSION_NAME:0.$i" C-m
  sleep 0.3
done

echo "Sent prompt to $PANE_COUNT agents in session '$SESSION_NAME'"
```

**CRITICAL**:
- Always use `-l` flag to send text literally (prevents `C-` prefix interpretation)
- Always send `C-m` separately after the text
- The prompt may contain quotes, special characters — `-l` handles this safely
- Add small sleep between panes to prevent tmux buffer issues
