# Init Workflow

Create `.swarm.json` config file interactively in the git repository root.

## Step 1: Check Prerequisites

```bash
git rev-parse --git-dir >/dev/null 2>&1 || echo "ERROR: not in a git repository"
```

If not in a git repo, tell the user and stop.

## Step 2: Check for Existing Config

```bash
GIT_ROOT=$(git rev-parse --show-toplevel)
CONFIG_FILE="$GIT_ROOT/.swarm.json"

if [ -f "$CONFIG_FILE" ]; then
  echo "Config already exists at $CONFIG_FILE"
fi
```

If config already exists, show the current contents and ask the user:
- **Overwrite** — replace with a new config
- **Cancel** — keep the existing config and stop

## Step 3: Gather Session Settings

Ask the user for session settings using AskUserQuestion:

**Question 1**: "What should the tmux session name be?"
- Options: `swarm` (default), `agents`, `dev`

**Question 2**: "Which branch should worktrees be created from?"
- Detect available branches with `git branch --list` and show top options
- Common defaults: `main`, `develop`, `master`

**Question 3**: "Where should worktrees be stored?"
- Options: `.worktrees` (default), `.swarm-worktrees`

## Step 4: Gather Agent Configuration

Ask the user which agents to include using AskUserQuestion (multiSelect):

**Question**: "Which AI agents do you want in the swarm?"
- `Claude` — `claude --permission-mode bypassPermissions`
- `Codex` — `codex --dangerously-bypass-approvals-and-sandbox`
- `Gemini` — `gemini --approval-mode yolo`
- Other — user provides custom agent name and command

At least 2 agents must be selected.

For each selected agent, the defaults are:

| Agent | command | branch_prefix |
|-------|---------|---------------|
| Claude | `claude --permission-mode bypassPermissions` | `swarm/claude` |
| Codex | `codex --dangerously-bypass-approvals-and-sandbox` | `swarm/codex` |
| Gemini | `gemini --approval-mode yolo` | `swarm/gemini` |

Ask the user: "Do you want to customize agent commands or use defaults?"
- **Use defaults** — proceed with the defaults above
- **Customize** — ask for each agent's command and branch_prefix individually

## Step 5: Write Config File

Build the JSON and write `.swarm.json` to the git root:

```bash
GIT_ROOT=$(git rev-parse --show-toplevel)
CONFIG_FILE="$GIT_ROOT/.swarm.json"
```

Write the config using the Write tool (NOT bash echo/cat). The structure:

```json
{
  "session_name": "<user_choice>",
  "source_branch": "<user_choice>",
  "worktree_base": "<user_choice>",
  "agents": [
    {
      "name": "<agent_name>",
      "command": "<agent_command>",
      "branch_prefix": "swarm/<agent_name>"
    }
  ]
}
```

## Step 6: Update .gitignore

Check if the worktree directory is already in `.gitignore`:

```bash
GIT_ROOT=$(git rev-parse --show-toplevel)
grep -qF "<worktree_base>" "$GIT_ROOT/.gitignore" 2>/dev/null
```

If NOT present, ask the user: "Add `<worktree_base>/` to .gitignore?"

If yes, append to `.gitignore`:

```bash
echo "<worktree_base>/" >> "$GIT_ROOT/.gitignore"
```

## Step 7: Report

Display the created config and next steps:

```
Config created: .swarm.json

Session:    <session_name>
Source:     <source_branch>
Worktrees:  <worktree_base>/
Agents:     <count>

| Agent   | Command                                           |
|---------|---------------------------------------------------|
| claude  | claude --permission-mode bypassPermissions        |
| codex   | codex --dangerously-bypass-approvals-and-sandbox  |
| gemini  | gemini --approval-mode yolo                       |

Next: /swarm start
```
