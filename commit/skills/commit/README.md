---
name: commit
description: Smart commit workflow with security review and documentation awareness
user_invocable: true
---

# Smart Commit

A commit workflow that handles security reviews and documentation updates.

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

For trivial commits, jump directly to Step 5 (Generate Commit).

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

1. **Hardcoded secrets**
   - API keys, tokens, passwords
   - Connection strings with credentials
   - Private keys or certificates

2. **Injection vulnerabilities**
   - SQL injection (unsanitized queries)
   - Command injection (shell commands with user input)
   - XSS (unescaped user content in HTML/templates)

3. **Sensitive data exposure**
   - Credentials in logs or error messages
   - PII in debugging output
   - Secrets in comments

4. **Auth/authz changes**
   - Permission checks being removed
   - Authentication bypass risks
   - Session handling changes

5. **Input validation**
   - Missing validation on user input
   - Type coercion issues
   - Path traversal risks

If issues found:
- List each issue with file and line number
- Ask user: "Security issues found. Continue anyway?" (Yes/No)
- If No, stop and let user fix issues

### Step 5: Documentation Updates (if requested)

If user requested documentation:

1. **Read existing docs**
   - If CLAUDE.md exists, read it and identify relevant section
   - If README.md exists, check for feature sections

2. **Suggest updates**
   - Propose specific text to add/modify
   - Show where it should go in the document

3. **Apply changes**
   - Use Edit tool to update documentation
   - Stage the documentation changes: `git add <doc-file>`

### Step 6: Generate Commit

Create a conventional commit:

**Format:**
```
type(scope): short description

Detailed explanation of WHY this change was made.
What problem does it solve? What's the context?

Co-Authored-By: Claude <noreply@anthropic.com>
```

**Types:**
- `feat` - New feature
- `fix` - Bug fix
- `docs` - Documentation only
- `style` - Formatting, no code change
- `refactor` - Code change that neither fixes nor adds
- `test` - Adding or updating tests
- `chore` - Maintenance tasks, dependencies

**Scope:** The component or area affected (e.g., `auth`, `api`, `ui`)

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

## Examples

### Trivial Commit (no questions)
```
chore(deps): update dependencies

Updated package-lock.json after npm install.

Co-Authored-By: Claude <noreply@anthropic.com>
```

### Feature Commit
```
feat(auth): add password reset flow

Users can now reset their password via email link.
This addresses frequent support requests about locked accounts.

Co-Authored-By: Claude <noreply@anthropic.com>
```

### Security Fix
```
fix(api): sanitize user input in search endpoint

Prevents SQL injection by using parameterized queries.
Found during security review of search functionality.

Co-Authored-By: Claude <noreply@anthropic.com>
```
