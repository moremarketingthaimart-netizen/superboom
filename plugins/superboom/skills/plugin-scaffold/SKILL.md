---
description: Scaffold a new Claude Code plugin from scratch with the correct directory structure, plugin.json manifest, example skill, and marketplace.json. Use when user wants to "create plugin", "new plugin", "scaffold plugin", "build claude code plugin", or "make a plugin for claude".
tools: [Bash, Read, Write]
---

# Skill: plugin-scaffold

Generate a complete, publishable Claude Code plugin with best-practice structure, ready to push to GitHub and list in a marketplace.

## Arguments

`$ARGUMENTS` — plugin name and optional description (e.g. "my-tools A toolkit for Python developers")

## Steps

### Step 1 — Parse plugin name

Extract the plugin name from `$ARGUMENTS`. The name must be:
- lowercase
- kebab-case only (no spaces, no underscores)
- 3–30 characters

If not provided, ask: "What should the plugin be named? (e.g. my-tools)"

### Step 2 — Confirm location

Create the plugin in the current directory as a new folder: `./<plugin-name>/`

### Step 3 — Create directory structure

```bash
PLUGIN_NAME="<from args>"
mkdir -p "$PLUGIN_NAME/.claude-plugin"
mkdir -p "$PLUGIN_NAME/skills/example-skill"
mkdir -p "$PLUGIN_NAME/agents"
mkdir -p "$PLUGIN_NAME/hooks"
mkdir -p "$PLUGIN_NAME/.github/workflows"
```

### Step 4 — Write plugin.json

```json
{
  "name": "<plugin-name>",
  "version": "0.1.0",
  "description": "<from args or placeholder>",
  "author": "<git config user.name>",
  "homepage": "https://github.com/<git config user.name>/<plugin-name>",
  "repository": "https://github.com/<git config user.name>/<plugin-name>",
  "license": "MIT",
  "keywords": [],
  "skills": ["example-skill"],
  "agents": []
}
```

Populate `author` from:
```bash
git config user.name
```

### Step 5 — Write example skill

`skills/example-skill/SKILL.md`:
```markdown
---
description: Replace this description with when Claude should auto-invoke this skill. Be specific about trigger phrases.
tools: [Bash, Read, Write]
---

# Skill: example-skill

Replace this with your skill's purpose and instructions.

## Arguments

`$ARGUMENTS` — describe what arguments this skill accepts

## Steps

### Step 1 — ...

### Step 2 — ...
```

### Step 6 — Write hooks/hooks.json (starter)

```json
{
  "Stop": [],
  "PostToolUse": [],
  "PreToolUse": []
}
```

### Step 7 — Write marketplace.json

```json
{
  "name": "<plugin-name>",
  "description": "<description>",
  "repository": "https://github.com/<author>/<plugin-name>",
  "version": "0.1.0",
  "tags": [],
  "install": "claude plugin install <author>/<plugin-name>"
}
```

### Step 8 — Write README.md

```markdown
# <plugin-name>

<description>

## Installation

\`\`\`bash
claude plugin install <author>/<plugin-name>
\`\`\`

## Skills

| Skill | Command | Description |
|-------|---------|-------------|
| `example-skill` | `/example-skill` | ... |

## Development

\`\`\`bash
# Test locally
claude --plugin-dir ./<plugin-name>

# Validate structure
claude plugin validate
\`\`\`

## License

MIT
```

### Step 9 — Write .github/workflows/validate.yml

```yaml
name: Validate Plugin

on: [push, pull_request]

jobs:
  validate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Validate plugin structure
        run: |
          test -f .claude-plugin/plugin.json || (echo "Missing plugin.json" && exit 1)
          echo "Plugin structure valid ✓"
```

### Step 10 — Git init

```bash
cd <plugin-name>
git init
echo "node_modules/" > .gitignore
echo "*.log" >> .gitignore
git add .
git commit -m "feat: initial plugin scaffold"
```

### Step 11 — Confirm

Show the full directory tree and next steps:
```
Plugin "<plugin-name>" created!

Next steps:
1. Edit skills/example-skill/SKILL.md with your skill logic
2. Test locally: claude --plugin-dir ./<plugin-name>
3. Push to GitHub: gh repo create <plugin-name> --public --push
4. Submit to marketplace: claude.ai/settings/plugins/submit
```
