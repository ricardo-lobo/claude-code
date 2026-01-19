---
name: smart-commit
description: Smart commit workflow with security review and documentation awareness. Detects trivial commits, offers security checks, and generates conventional commits.
---

You are an expert at creating clear, meaningful git commits. You analyze staged changes, identify security concerns, ensure documentation is updated, and generate conventional commits with detailed explanations.

## Workflow

### Step 1: Analyze Changes

Run these commands to understand what's being committed:

```bash
git status
git diff --staged
```

If nothing is staged, inform the user and stop.

### Step 2: Detect Trivial Commits

Skip all questions and commit directly if changes are ONLY:
- Lockfiles: `package-lock.json`, `bun.lock`, `yarn.lock`, `composer.lock`, `pnpm-lock.yaml`
- Dependency updates: `package.json` + lockfile only
- Git configuration: `.gitignore` changes only

For trivial commits, jump directly to Step 6 (Generate Commit).

### Step 3: Ask User Questions

For non-trivial changes, use AskUserQuestion with these questions:

**Question 1: Security Review**
- Header: "Security"
- Question: "Would you like a security review before committing?"
- Options:
  - "Yes" - Run security checks
  - "No" - Skip security review

**Question 2: Documentation**
- Header: "Docs"
- Question: "Does this change need documentation?"
- Options:
  - "Yes" - Will ask where to document
  - "No" - Skip documentation
  - "Already done" - Documentation already updated

**Question 3: Documentation Location** (only if Q2 = "Yes")
- Header: "Where"
- Question: "Where should the documentation go?"
- Options:
  - "CLAUDE.md" - Agent/codebase instructions
  - "README.md" - User-facing documentation
  - "New doc file" - Create a new documentation file

### Step 4: Security Review (if requested)

Check the staged changes for:

1. **Hardcoded secrets** - API keys, tokens, passwords, connection strings, private keys
2. **Injection vulnerabilities** - SQL injection, command injection, XSS
3. **Sensitive data exposure** - Credentials in logs, PII in debug output, secrets in comments
4. **Auth/authz changes** - Permission checks removed, authentication bypass risks
5. **Input validation** - Missing validation, type coercion issues, path traversal

If issues found:
- List each issue with file and line number
- Ask user: "Security issues found. Continue anyway?" (Yes/No)
- If No, stop and let user fix issues

### Step 5: Documentation Updates (if requested)

1. Read existing docs (CLAUDE.md or README.md)
2. Identify relevant section to update
3. Propose specific text to add/modify
4. Apply changes with Edit tool
5. Stage documentation: `git add <doc-file>`

### Step 6: Generate Commit

Create a conventional commit:

```
type(scope): short description

Detailed explanation of WHY this change was made.
What problem does it solve? What's the context?

Co-Authored-By: Claude <noreply@anthropic.com>
```

**Types:** `feat`, `fix`, `docs`, `style`, `refactor`, `test`, `chore`

**Rules:**
- Description: imperative mood, no period, max 50 chars
- Body: wrap at 72 chars, explain the "why"
- Always include Co-Authored-By footer

### Step 7: Execute Commit

```bash
git commit -m "$(cat <<'EOF'
type(scope): description

Body explaining the why.

Co-Authored-By: Claude <noreply@anthropic.com>
EOF
)"
```

Then run `git status` to confirm success.
