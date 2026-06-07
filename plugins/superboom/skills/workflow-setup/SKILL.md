---
description: Scaffold a complete Claude Code project setup including CLAUDE.md, hooks, agents, and settings.local.json. Use when user wants to "setup project", "init claude", "initialize claude code", or "set up workspace for Claude Code".
tools: [Bash, Read, Write, Edit]
---

# Skill: workflow-setup

Bootstrap a Claude Code workspace from zero. Creates all necessary config files following the official Claude Code best practices.

## What this skill does

1. Creates `CLAUDE.md` with project rules, commands reference, and agent overview
2. Creates `.claude/settings.local.json` with hooks wired up
3. Creates `.claude/hooks/` with 4 essential hooks:
   - `token-alert.sh` — macOS notification when token usage > 60%
   - `stop-summary.sh` — display token stats on every stop
   - `session-logger.sh` — log all tool calls to `~/.claude/logs/`
   - `pre-tool-safety.sh` — block dangerous bash patterns before they run
4. Creates `.claude/agents/` scaffold with README
5. Creates `.claude/commands/` scaffold with README

## Steps

### Step 1 — Detect existing files

```bash
ls -la .claude/ 2>/dev/null || echo "No .claude directory yet"
ls CLAUDE.md 2>/dev/null || echo "No CLAUDE.md yet"
```

If `CLAUDE.md` already exists, ask user before overwriting.

### Step 2 — Create directory structure

```bash
mkdir -p .claude/hooks .claude/agents .claude/commands
```

### Step 3 — Write CLAUDE.md

Generate a CLAUDE.md tailored to the project. Read `package.json`, `pyproject.toml`, `Cargo.toml`, or `go.mod` (whichever exists) to detect the tech stack and include relevant build/test commands.

Template structure:
```markdown
# CLAUDE.md

## Project Overview
<detected from existing files>

## Build & Test Commands
<detected from package manager>

## Claude Code Agents
<list any .md files in .claude/agents/>

## Hooks
| Hook | Event | Purpose |
|------|-------|---------|
| token-alert.sh | Stop | macOS notification when token > 60% |
| stop-summary.sh | Stop | Show token stats |
| session-logger.sh | PostToolUse | Log all tool calls |
| pre-tool-safety.sh | PreToolUse(Bash) | Block dangerous patterns |

## Guidelines for Claude
- Always run tests before marking a task complete
- Prefer editing existing files over creating new ones
- Ask before destructive operations (rm, reset --hard, force push)
```

### Step 4 — Write hooks

**token-alert.sh:**
```bash
#!/bin/bash
# Notify when token usage exceeds 60% threshold
INPUT=$(cat)
TOKENS_USED=$(echo "$INPUT" | python3 -c "import json,sys; d=json.load(sys.stdin); print(d.get('usage',{}).get('input_tokens',0)+d.get('usage',{}).get('output_tokens',0))" 2>/dev/null || echo 0)
THRESHOLD=120000
if [ "$TOKENS_USED" -gt "$THRESHOLD" ] 2>/dev/null; then
  osascript -e "display notification \"Token usage: $TOKENS_USED / 200,000 — Consider /compact\" with title \"Claude Code Token Alert\""
fi
```

**stop-summary.sh:**
```bash
#!/bin/bash
INPUT=$(cat)
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
echo "  Claude Code Session Summary"
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
echo "$INPUT" | python3 -c "
import json, sys
d = json.load(sys.stdin)
u = d.get('usage', {})
inp = u.get('input_tokens', 0)
out = u.get('output_tokens', 0)
total = inp + out
pct = round(total / 200000 * 100, 1)
print(f'  Input:  {inp:,} tokens')
print(f'  Output: {out:,} tokens')
print(f'  Total:  {total:,} / 200,000 ({pct}%)')
if pct > 60:
    print(f'  ⚠️  Consider running /compact')
" 2>/dev/null || echo "  (token data unavailable)"
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
```

**session-logger.sh:**
```bash
#!/bin/bash
LOG_DIR="$HOME/.claude/logs"
mkdir -p "$LOG_DIR"
LOG_FILE="$LOG_DIR/session-$(date +%Y-%m-%d).log"
INPUT=$(cat)
TOOL=$(echo "$INPUT" | python3 -c "import json,sys; d=json.load(sys.stdin); print(d.get('tool_name','unknown'))" 2>/dev/null)
TIMESTAMP=$(date '+%Y-%m-%d %H:%M:%S')
echo "[$TIMESTAMP] tool=$TOOL" >> "$LOG_FILE"
```

**pre-tool-safety.sh:**
```bash
#!/bin/bash
INPUT=$(cat)
COMMAND=$(echo "$INPUT" | python3 -c "import json,sys; d=json.load(sys.stdin); print(d.get('tool_input',{}).get('command',''))" 2>/dev/null)
DANGEROUS_PATTERNS=("rm -rf /" "git push --force origin main" "git push --force origin master" "DROP TABLE" "DROP DATABASE" "format c:")
for pattern in "${DANGEROUS_PATTERNS[@]}"; do
  if echo "$COMMAND" | grep -qi "$pattern"; then
    echo "{\"decision\": \"block\", \"reason\": \"Blocked dangerous pattern: $pattern\"}"
    exit 0
  fi
done
```

### Step 5 — Write settings.local.json

```json
{
  "hooks": {
    "Stop": [
      { "type": "command", "command": ".claude/hooks/token-alert.sh" },
      { "type": "command", "command": ".claude/hooks/stop-summary.sh" }
    ],
    "PostToolUse": [
      { "matcher": ".*", "type": "command", "command": ".claude/hooks/session-logger.sh" }
    ],
    "PreToolUse": [
      { "matcher": "Bash", "type": "command", "command": ".claude/hooks/pre-tool-safety.sh" }
    ]
  }
}
```

### Step 6 — Make hooks executable

```bash
chmod +x .claude/hooks/*.sh
```

### Step 7 — Confirm

Report what was created:
- List all created files
- Show the hooks configuration
- Remind user to restart Claude Code for hooks to take effect
