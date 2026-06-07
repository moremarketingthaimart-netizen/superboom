# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

A Claude Code plugin named **superboom** — a distributable bundle of Skills, Agents, and Hooks that users install via `claude plugin install moremarketingthaimart-netizen/superboom`. There is no build system, no runtime, and no package manager. All content is Markdown and JSON.

## Validation

The only "test" command is the GitHub Actions workflow, which can be replicated locally:

```bash
# Check all required files are present
bash -c '
  for skill in workflow-setup github-setup agent-team security-audit migration pr-review daily-digest plugin-scaffold; do
    test -f "skills/$skill/SKILL.md" || echo "MISSING: skills/$skill/SKILL.md"
  done
  for agent in security-reviewer code-researcher devops-specialist; do
    test -f "agents/$agent.md" || echo "MISSING: agents/$agent.md"
  done
'

# Validate JSON syntax
python3 -c "import json; json.load(open('.claude-plugin/plugin.json')); json.load(open('marketplace.json')); json.load(open('hooks/hooks.json'))" && echo "JSON valid"
```

## Architecture

### Skills (`skills/<name>/SKILL.md`)

Each skill is a Markdown file with a YAML frontmatter block followed by freeform instruction prose. The frontmatter is what Claude Code parses; the prose is the instruction set Claude follows when the skill is invoked.

Required frontmatter fields:
- `description` — the trigger phrase Claude uses to auto-invoke the skill
- `tools` — list of Claude Code tools the skill is allowed to use
- `subagent: true` — optional; runs the skill in an isolated context window

When adding a new skill, register its name in both `skills[]` in `.claude-plugin/plugin.json` and in the `validate.yml` `SKILLS` array, or CI will fail.

### Agents (`agents/<name>.md`)

Agent files declare a specialized subagent persona. Key frontmatter fields: `model`, `effort`, `maxTurns`, `tools`, `disallowedTools`, `isolation`. The body is the system prompt for that agent. Register new agents in `agents[]` in `plugin.json` and the `AGENTS` array in `validate.yml`.

### Hooks (`hooks/hooks.json`)

A reference/template file — it is **not** auto-loaded by Claude Code. Users copy relevant sections into their own `.claude/settings.local.json`. The hooks cover `Stop`, `PostToolUse`, `PreToolUse`, and `SubagentStop` lifecycle events.

### Manifest files

- `.claude-plugin/plugin.json` — parsed by Claude Code; `skills[]` and `agents[]` arrays must match actual file paths exactly
- `marketplace.json` — catalog metadata; `install` field is the user-facing install command
- Both must stay in sync when skills or agents are added/removed

## Key constraints

- SKILL.md files must start with a `---` YAML frontmatter block and include a `description:` field — CI checks this
- Agent `.md` files must also start with `---` frontmatter — CI checks this
- JSON files (plugin.json, marketplace.json, hooks.json) must be valid JSON — CI checks this
- Skill names in `plugin.json` must exactly match directory names under `skills/`
- Agent names in `plugin.json` must exactly match filenames (without `.md`) under `agents/`
