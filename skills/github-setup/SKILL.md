---
description: Set up GitHub Actions with Claude Code integration using anthropics/claude-code-action. Use when user wants to "setup github actions", "add ci/cd with claude", "automate PR review with claude", or "install claude github app".
tools: [Bash, Read, Write, Edit]
---

# Skill: github-setup

Wire Claude Code into your GitHub repository for automated PR reviews, issue-to-PR workflows, and @claude mentions in comments.

## What this skill creates

1. `.github/workflows/claude.yml` — Claude responds to `@claude` mentions in PRs and issues
2. `.github/workflows/claude-pr-review.yml` — Auto-review every PR on push
3. Setup instructions for the GitHub App and API key secret

## Steps

### Step 1 — Check existing GitHub setup

```bash
ls .github/workflows/ 2>/dev/null || echo "No workflows yet"
git remote -v 2>/dev/null | head -5
```

### Step 2 — Create .github/workflows/claude.yml

```yaml
name: Claude Code

on:
  issue_comment:
    types: [created]
  pull_request_review_comment:
    types: [created]
  issues:
    types: [opened, assigned]
  pull_request:
    types: [opened]

jobs:
  claude:
    if: |
      (github.event_name == 'issue_comment' && contains(github.event.comment.body, '@claude')) ||
      (github.event_name == 'pull_request_review_comment' && contains(github.event.comment.body, '@claude')) ||
      (github.event_name == 'issues' && contains(github.event.issue.body, '@claude')) ||
      (github.event_name == 'pull_request' && github.event.action == 'opened')
    runs-on: ubuntu-latest
    permissions:
      contents: write
      pull-requests: write
      issues: write
    steps:
      - uses: anthropics/claude-code-action@v1
        with:
          anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
          trigger_phrase: "@claude"
```

### Step 3 — Create .github/workflows/claude-pr-review.yml

```yaml
name: Claude PR Auto-Review

on:
  pull_request:
    types: [opened, synchronize]

jobs:
  review:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      pull-requests: write
    steps:
      - uses: anthropics/claude-code-action@v1
        with:
          anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
          prompt: |
            Review this PR for:
            1. Correctness bugs and logic errors
            2. Security vulnerabilities (OWASP Top 10)
            3. Performance issues
            4. Code style and maintainability
            
            Format your review as inline comments on specific lines where possible.
            End with a summary: APPROVE / REQUEST_CHANGES / COMMENT
```

### Step 4 — Print setup instructions

After creating the files, show these instructions:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  GitHub Setup — Next Steps
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

1. Install the Claude GitHub App:
   Run: claude /install-github-app
   Or visit: https://github.com/apps/claude

2. Add your API key as a GitHub secret:
   Go to: Settings → Secrets → Actions → New secret
   Name: ANTHROPIC_API_KEY
   Value: <your key from https://console.anthropic.com>

3. Commit and push the workflows:
   git add .github/workflows/
   git commit -m "feat: add Claude Code GitHub Actions"
   git push

4. Test by opening a PR and commenting "@claude please review this"
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```
