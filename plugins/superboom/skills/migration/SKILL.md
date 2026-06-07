---
description: Orchestrate a large-scale codebase migration using dynamic parallel subagents. Use when user wants to "migrate codebase", "port to new framework", "large refactor", "rename across all files", or "upgrade dependency across codebase".
tools: [Bash, Read, Write, Edit, Agent]
subagent: true
---

# Skill: migration

For large-scale changes (hundreds of files, framework migrations, language ports), this skill generates a JavaScript orchestration script that fans out to parallel subagents — keeping intermediate state out of the main context window.

## Arguments

`$ARGUMENTS` — describe the migration (e.g. "upgrade React 17 to 19", "rename getUserById to fetchUserById everywhere", "migrate Express.js to Fastify")

## Steps

### Step 1 — Scope the migration

Understand what needs to change:
```bash
# Count affected files
grep -rl "OLD_PATTERN" . --include="*.ts" --include="*.js" | wc -l

# Preview affected files
grep -rl "OLD_PATTERN" . --include="*.ts" --include="*.js" | head -20
```

If there are fewer than 10 files, do the migration directly without orchestration. If 10+ files, proceed with the parallel workflow.

### Step 2 — Create migration plan

Write a `migration-plan.md`:

```markdown
# Migration Plan: <description>

## Scope
- Total files affected: X
- File types: <list>

## Batching Strategy
- Batch size: 10 files per subagent
- Total batches: X
- Estimated subagents: X

## Change Pattern
<describe exactly what changes in each file>

## Validation
<how to verify the migration is correct>

## Rollback
git checkout -- . (before committing)
```

### Step 3 — Generate orchestration script

Write `migration-orchestrate.js`:

```javascript
#!/usr/bin/env node
/**
 * Migration Orchestrator
 * Fans out to parallel subagents, each handling a batch of files.
 * Run with: node migration-orchestrate.js
 */

const { execSync } = require('child_process');
const fs = require('fs');

// Discover all files to migrate
const files = execSync(
  'grep -rl "OLD_PATTERN" . --include="*.ts" --include="*.js" 2>/dev/null',
  { encoding: 'utf8' }
).trim().split('\n').filter(Boolean);

console.log(`Found ${files.length} files to migrate`);

// Split into batches of 10
const BATCH_SIZE = 10;
const batches = [];
for (let i = 0; i < files.length; i += BATCH_SIZE) {
  batches.push(files.slice(i, i + BATCH_SIZE));
}

// Track results
const results = { success: [], failed: [], skipped: [] };

// Process batches (in real Claude Code, these would fan out as parallel subagents)
for (let i = 0; i < batches.length; i++) {
  const batch = batches[i];
  console.log(`\nBatch ${i + 1}/${batches.length}: ${batch.length} files`);
  
  for (const file of batch) {
    try {
      let content = fs.readFileSync(file, 'utf8');
      const original = content;
      
      // Apply migration pattern
      content = content.replace(/OLD_PATTERN/g, 'NEW_PATTERN');
      
      if (content !== original) {
        fs.writeFileSync(file, content);
        results.success.push(file);
        console.log(`  ✓ ${file}`);
      } else {
        results.skipped.push(file);
      }
    } catch (err) {
      results.failed.push({ file, error: err.message });
      console.error(`  ✗ ${file}: ${err.message}`);
    }
  }
}

// Report
console.log('\n━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━');
console.log('Migration Complete');
console.log(`  ✓ Success: ${results.success.length}`);
console.log(`  ↷ Skipped: ${results.skipped.length}`);
console.log(`  ✗ Failed:  ${results.failed.length}`);
if (results.failed.length > 0) {
  console.log('\nFailed files:');
  results.failed.forEach(f => console.log(`  ${f.file}: ${f.error}`));
}
```

Customize `OLD_PATTERN` and `NEW_PATTERN` based on `$ARGUMENTS`.

### Step 4 — Adapt orchestration script

Replace the placeholder patterns in `migration-orchestrate.js` with the actual migration logic based on what `$ARGUMENTS` describes. For framework migrations, the replacement logic will be more complex — add proper AST-based transformations if needed (using `@babel/core`, `ts-morph`, `jscodeshift`, etc.).

### Step 5 — Dry run

```bash
node migration-orchestrate.js --dry-run 2>&1 | head -50
```

Report the preview to the user and ask for confirmation before proceeding.

### Step 6 — Execute (after confirmation)

```bash
node migration-orchestrate.js
```

### Step 7 — Validate

Run the project's test suite to verify the migration didn't break anything:
```bash
npm test || yarn test || pytest || cargo test || go test ./...
```

Report pass/fail status and any errors that need manual attention.

### Step 8 — Cleanup

```bash
rm migration-orchestrate.js migration-plan.md
```

Offer to create a git commit with the migration changes.
