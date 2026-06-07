# superboom

> Supercharge your Claude Code workflow with battle-tested Skills for multi-agent orchestration, GitHub automation, parallel security audits, large-scale codebase migrations, and more.

Built on the latest Claude Code capabilities: Dynamic Workflows, Hooks system, Agent teams, MCP integration, and GitHub Actions.

## Installation

```bash
claude plugin install moremarketingthaimart/superboom
```

Or add to your project's `.claude/settings.json`:
```json
{
  "plugins": ["moremarketingthaimart/superboom"]
}
```

## Skills

All skills are available as `/superboom:<skill-name>` or Claude will invoke them automatically when context matches.

### `/superboom:workflow-setup`

Scaffold a complete Claude Code project setup from zero.

**Creates:** `CLAUDE.md`, `.claude/settings.local.json`, and 4 production-ready hooks:
- `token-alert.sh` — macOS notification when token usage exceeds 60%
- `stop-summary.sh` — Token stats every time Claude stops
- `session-logger.sh` — Log all tool calls to `~/.claude/logs/`
- `pre-tool-safety.sh` — Block dangerous bash patterns before they run

```
/superboom:workflow-setup
```

---

### `/superboom:github-setup`

Wire Claude Code into your GitHub repository.

**Creates:**
- `.github/workflows/claude.yml` — Respond to `@claude` in PR/issue comments
- `.github/workflows/claude-pr-review.yml` — Auto-review every PR on push

```
/superboom:github-setup
```

---

### `/superboom:agent-team`

Generate a specialized multi-agent team for your project.

**Creates 3 agents in `.claude/agents/`:**
- `coding-agent` — Feature implementation, bug fixes (Sonnet, full read/write)
- `devops-agent` — CI/CD, Docker, deployment (Sonnet, Bash-heavy)
- `review-agent` — Code review, security audit (Haiku, read-only, isolated worktree)

```
/superboom:agent-team python backend
/superboom:agent-team react app
```

---

### `/superboom:security-audit`

Run a comprehensive parallel security audit using 5 specialized subagents simultaneously.

**Covers:** Injection (OWASP A03), Authentication (A07), Access Control (A01), Cryptography (A02), Secrets/Data Exposure (A09), Dependency CVEs (A06)

**Output:** Severity-graded report saved as `security-audit-YYYY-MM-DD.md`

```
/superboom:security-audit
/superboom:security-audit src/api
```

---

### `/superboom:migration`

Orchestrate large-scale codebase changes using dynamic parallel workflows.

For 10+ files, generates a JavaScript orchestration script that fans out to parallel subagents — keeping intermediate state out of the main context window.

```
/superboom:migration upgrade React 17 to 19
/superboom:migration rename getUserById to fetchUserById everywhere
/superboom:migration migrate Express.js to Fastify
```

---

### `/superboom:pr-review`

Structured code review with parallel correctness, security, and quality subagents.

**Output:** Severity-graded findings with file:line references and a clear APPROVE / REQUEST_CHANGES verdict.

```
/superboom:pr-review
/superboom:pr-review PR #42
/superboom:pr-review feature/auth-refactor
```

---

### `/superboom:daily-digest`

Generate a daily or weekly digest of commits, PRs, and issues.

**Formats:** Markdown (default) or Slack (`format=slack`)

```
/superboom:daily-digest
/superboom:daily-digest this week format=slack
/superboom:daily-digest yesterday
```

---

### `/superboom:plugin-scaffold`

Scaffold a new Claude Code plugin with correct structure, ready to publish.

**Creates:** `plugin.json`, example `SKILL.md`, `hooks.json`, `marketplace.json`, `README.md`, and GitHub Actions validation workflow.

```
/superboom:plugin-scaffold my-tools A toolkit for Python developers
```

---

## Agents

Three specialized agents are included and auto-invoked by context:

| Agent | Model | Role |
|-------|-------|------|
| `security-reviewer` | Haiku | Read-only security audit, isolated worktree |
| `code-researcher` | Haiku | Fast symbol lookup and code understanding |
| `devops-specialist` | Sonnet | CI/CD, Docker, infrastructure |

---

## Hooks

`hooks/hooks.json` contains ready-to-use hooks. Copy relevant sections to your project's `.claude/settings.local.json`.

| Hook | Event | Action |
|------|-------|--------|
| Token summary | `Stop` | Print token usage stats |
| Token alert | `Stop` | macOS notification when > 60% used |
| Auto-format | `PostToolUse(Write\|Edit)` | Prettier (JS/TS) + Black (Python) |
| Session logger | `PostToolUse` | Log all tool calls |
| Safety guard | `PreToolUse(Bash)` | Block `rm -rf /`, force push to main, DROP TABLE |
| Subagent logger | `SubagentStop` | Log agent name + token cost |

---

## Requirements

- Claude Code CLI
- `gh` (GitHub CLI) — for `github-setup` and `pr-review`
- `node` — for `migration` orchestration scripts
- `prettier` — optional, for auto-format hook
- `black` — optional, for Python auto-format hook

---

## Development

```bash
# Test locally
claude --plugin-dir ./superboom

# Validate structure
claude plugin validate

# Run CI locally
act -j validate-structure
```

---

## License

MIT © superboom contributors
