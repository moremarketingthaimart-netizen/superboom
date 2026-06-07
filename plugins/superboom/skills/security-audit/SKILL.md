---
description: Run a comprehensive parallel security audit of the codebase using multiple specialized subagents. Use when user asks for "security audit", "scan for vulnerabilities", "check security", "OWASP review", or "find security issues".
tools: [Bash, Read, Agent]
subagent: true
---

# Skill: security-audit

Spawn parallel security subagents — each focused on a specific vulnerability category — then synthesize findings into a prioritized report.

## Arguments

`$ARGUMENTS` — optional scope (e.g. "src/api", "only authentication", "skip frontend")

## Steps

### Step 1 — Understand scope

Detect the project structure:
```bash
find . -type f \( -name "*.js" -o -name "*.ts" -o -name "*.py" -o -name "*.go" -o -name "*.java" \) \
  -not -path "*/node_modules/*" -not -path "*/.git/*" | head -30
```

If `$ARGUMENTS` narrows the scope, use that directory or constraint.

### Step 2 — Spawn parallel security subagents

Launch all 5 subagents simultaneously using the Agent tool:

**Agent 1: Injection Vulnerabilities**
- Focus: SQL injection, command injection, XSS, SSTI, path traversal
- Look for: string concatenation in queries, `eval()`, `exec()`, unescaped user input in HTML

**Agent 2: Authentication & Authorization**
- Focus: hardcoded credentials, weak JWT configs, missing auth checks, IDOR
- Look for: API keys in source, `Bearer` tokens, role checks, session management

**Agent 3: Dependency Vulnerabilities**
- Run: `npm audit --json` / `pip-audit` / `cargo audit` / `govulncheck ./...`
- Report: CVE IDs, severity, affected version, fix version

**Agent 4: Secrets & Data Exposure**
- Focus: API keys, passwords, private keys committed to code
- Look for: patterns like `sk-`, `AKIA`, `-----BEGIN`, `password =`, `.env` in git history

**Agent 5: Cryptography & Transport**
- Focus: weak algorithms (MD5, SHA1, DES), HTTP instead of HTTPS, certificate pinning, random number generation

Each agent reports findings in this format:
```
SEVERITY: CRITICAL|HIGH|MEDIUM|LOW
FILE: path/to/file.ext:line
ISSUE: description
EVIDENCE: code snippet
FIX: recommended remediation
```

### Step 3 — Synthesize findings

After all agents complete, compile results into a unified security report:

```markdown
# Security Audit Report
Generated: <date>
Scope: <scope>

## Executive Summary
- CRITICAL: X issues
- HIGH: X issues  
- MEDIUM: X issues
- LOW: X issues

## Critical Findings (fix immediately)
<findings>

## High Priority
<findings>

## Medium Priority
<findings>

## Low Priority / Informational
<findings>

## Dependency Vulnerabilities
<npm audit / pip-audit output summary>

## Recommendations
1. <top priority action>
2. <second priority>
...
```

### Step 4 — Save report

Write the report to `security-audit-<YYYY-MM-DD>.md` in the project root and confirm the file was created.
