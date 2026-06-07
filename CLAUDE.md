# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

A Claude Code **marketplace** that distributes the **superboom** plugin — a bundle of Skills, Agents, and Hooks. Users add it via the Claude Code plugin manager using the HTTPS URL:

```
https://github.com/moremarketingthaimart-netizen/superboom
```

There is no build system, no runtime, and no package manager. All content is Markdown and JSON.

## Structure

```
.claude-plugin/marketplace.json   ← marketplace catalog (parsed by Claude Code)
plugins/superboom/
  .claude-plugin/plugin.json      ← plugin manifest ($schema required)
  skills/<name>/SKILL.md          ← skill instruction files
  agents/<name>.md                ← agent persona files
  hooks/hooks.json                ← hook templates (users copy to their project)
```

## Validation

```bash
# Check all required files are present and JSON is valid
python3 -c "
import json, os, sys
json.load(open('.claude-plugin/marketplace.json'))
json.load(open('plugins/superboom/.claude-plugin/plugin.json'))
json.load(open('plugins/superboom/hooks/hooks.json'))
print('JSON valid')
"

for skill in workflow-setup github-setup agent-team security-audit migration pr-review daily-digest plugin-scaffold; do
  test -f "plugins/superboom/skills/$skill/SKILL.md" || echo "MISSING: $skill"
done
```

## Architecture

### `.claude-plugin/marketplace.json`

The catalog file Claude Code reads when this repo is added as a marketplace. Uses `$schema`, `name`, `owner`, and `plugins[]` array. Each entry in `plugins[]` has `name`, `source` (relative path to the plugin dir), `version`, `category`, `keywords`. This file is at the **repo root level** `.claude-plugin/`, not inside the plugin subfolder.

### `plugins/superboom/.claude-plugin/plugin.json`

The plugin manifest. Must use `$schema`, `displayName`, `author` (object with `name`/`url`), `description`, `homepage`. This is distinct from the marketplace catalog above.

### Skills (`plugins/superboom/skills/<name>/SKILL.md`)

Each SKILL.md has a YAML frontmatter block (required: `description`, `tools`) followed by instruction prose Claude follows when the skill is invoked. `subagent: true` runs the skill in an isolated context window.

### Agents (`plugins/superboom/agents/<name>.md`)

Agent frontmatter: `name`, `model`, `effort`, `maxTurns`, `tools`, `disallowedTools`, `isolation`. Body is the agent's system prompt.

### Hooks (`plugins/superboom/hooks/hooks.json`)

Reference/template file — not auto-loaded. Users copy relevant sections into their own `.claude/settings.local.json`.

## Key constraints

- `.claude-plugin/marketplace.json` must use the `$schema` URL and have a `plugins[]` array where `source` matches an actual subdirectory
- SKILL.md files must start with `---` frontmatter and include `description:` — CI checks this
- Agent `.md` files must start with `---` frontmatter — CI checks this
- All three JSON files must be valid — CI checks this
