# Tmux Key Rules

Critical patterns for `tmux send-keys` to prevent hanging and input issues.

## Rules

1. **Always** use `C-m` instead of literal `Enter`
2. **Always** use `-l` flag for text content (literal mode — prevents interpreting `C-` prefixes)
3. **Always** send text and `C-m` as **separate** `send-keys` calls
4. **Always** add `sleep 0.2-0.5` between commands to different panes

## Correct Pattern

```bash
# Send command text (literal)
tmux send-keys -t "$SESSION_NAME:0.$PANE_INDEX" -l "$COMMAND_TEXT"
sleep 0.3
# Send Enter key separately
tmux send-keys -t "$SESSION_NAME:0.$PANE_INDEX" C-m
sleep 0.5
```

## Wrong Patterns (DO NOT USE)

```bash
# WRONG: Enter as string
tmux send-keys -t "$SESSION" "$CMD" Enter

# WRONG: text and C-m combined
tmux send-keys -t "$SESSION" -l "$CMD" C-m

# WRONG: no -l flag (special chars break)
tmux send-keys -t "$SESSION" "$CMD"
```

## Multi-line Prompts

Send each line separately:

```bash
echo "$PROMPT" | while IFS= read -r line; do
  tmux send-keys -t "$SESSION_NAME:0.$i" -l "$line"
  tmux send-keys -t "$SESSION_NAME:0.$i" C-m
  sleep 0.1
done
```
