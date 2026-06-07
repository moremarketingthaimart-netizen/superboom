---
description: Perform a structured code review of a pull request or recent git changes. Use when user asks to "review pr", "code review", "review my changes", "check this diff", or "review before merge".
tools: [Bash, Read, Agent]
subagent: true
---

# Skill: pr-review

Deliver a structured, actionable PR review with severity-graded findings, inline references, and a clear verdict.

## Arguments

`$ARGUMENTS` — optional PR number, branch name, or file path (e.g. "PR #42", "feature/auth", "src/api/")

## Steps

### Step 1 — Get the diff

If `$ARGUMENTS` is a PR number, fetch it:
```bash
gh pr diff $PR_NUMBER 2>/dev/null
```

If `$ARGUMENTS` is a branch name:
```bash
git diff main...$BRANCH --stat
git diff main...$BRANCH
```

If no argument, review staged + unstaged changes:
```bash
git diff HEAD --stat
git diff HEAD
```

### Step 2 — Understand context

Read the files that were changed (not just the diff):
```bash
git diff --name-only HEAD | head -20
```

For each changed file, read the full file to understand the broader context around the diff.

Also check:
```bash
git log --oneline -5   # recent context
git diff HEAD --stat   # change volume
```

### Step 3 — Spawn specialized review subagents

Launch 3 parallel review agents:

**Correctness Agent** — focus on:
- Logic errors and off-by-one bugs
- Null/undefined handling
- Error handling and edge cases
- Race conditions and concurrency issues
- Return type mismatches

**Security Agent** — focus on:
- SQL/command injection points
- XSS via unescaped output
- Auth/authz gaps (missing checks, privilege escalation)
- Sensitive data logged or exposed
- Dependency CVEs introduced

**Quality Agent** — focus on:
- Naming clarity (variables, functions, types)
- Cyclomatic complexity
- Code duplication (DRY violations)
- Missing or insufficient tests
- Documentation gaps for public APIs

### Step 4 — Synthesize findings

Compile all subagent findings into one structured report:

```markdown
## PR Review

**Files changed:** X  
**Lines added:** +X  **Lines removed:** -X  
**Verdict:** APPROVE ✅ | REQUEST_CHANGES ❌ | NEEDS_DISCUSSION 💬

---

### Critical Issues (must fix before merge)
> [CRITICAL] `path/to/file.ts:42` — SQL query built with string concatenation
> Fix: Use parameterized queries: `db.query("SELECT * FROM users WHERE id = ?", [userId])`

### High Priority
> [HIGH] ...

### Medium Priority
> [MEDIUM] ...

### Low / Suggestions
> [LOW] ...

### Positive Observations
- Well-structured error handling in `auth.ts`
- Good test coverage for the happy path

---

### Summary
<2–3 sentence overall assessment>
```

### Step 5 — Output

Print the review directly to the conversation. If `$ARGUMENTS` was a PR number and the user wants to post it:

```bash
gh pr review $PR_NUMBER --comment --body "$REVIEW_CONTENT"
```

Ask the user before posting to GitHub.
