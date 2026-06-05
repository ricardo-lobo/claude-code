---
name: smart-commit
description: Smart commit workflow with a code-quality pass (flags redundant AI-generated comments and leftover debug code), risk-triggered security checks, repo-convention-aware conventional commits, and documentation awareness. Use this skill when the user asks to commit changes or create a git commit.
---

You are an expert at creating clear, meaningful git commits. You survey the changes, strip low-value noise (especially the redundant comments AI tends to over-produce), catch security issues and debug leftovers, conform to the repository's existing commit style, and write a tight conventional commit.

Be conservative and never surprise the user: do not delete code, rewrite history, push, or commit files the user did not intend without confirming first.

## Workflow

### Step 1: Survey the working tree (stage-aware)

Run in parallel to see the full picture:

```bash
git status
git diff --staged    # what is already staged
git diff             # unstaged changes to tracked files
```

Decide what to commit:

- **Something already staged** → commit that by default, and mention any unstaged/untracked files you're leaving out.
- **Nothing staged but there are changes** → this is the common case, so do NOT stop. Summarize what changed and confirm what to include (default: all related changes), then stage with `git add -A` or a targeted subset.
- **Working tree clean** → tell the user there's nothing to commit and stop.

Only the files you intend to commit should be staged by the time you reach Step 6.

### Step 2: Learn the repo's commit convention

Before writing anything, infer the project's style instead of imposing one:

```bash
git log --oneline -20
```

Conform to what you see:

- Whether commits use **conventional prefixes** (`feat:`, `fix(scope):`) or freeform subjects.
- Whether **scopes** are used and what they look like.
- Whether commits carry **trailers/footers** (`Co-Authored-By`, `Signed-off-by`, issue refs). Match the prevailing pattern — do NOT add a `Co-Authored-By` trailer if the history doesn't use one.
- Subject **language and capitalization**.

Prefer an explicit rule in `CONTRIBUTING.md` or `CLAUDE.md` over inferred style.

### Step 3: Fast-path trivial commits

If the staged set is ONLY:

- Lockfiles: `package-lock.json`, `bun.lock`, `yarn.lock`, `pnpm-lock.yaml`, `composer.lock`, `Gemfile.lock`, `Cargo.lock`, `go.sum`, `poetry.lock`
- A dependency manifest + its lockfile (e.g. `package.json` + lock)
- `.gitignore` only

…skip the quality review and questions and jump to Step 6 with a short, subject-only message (e.g. `chore(deps): update dependencies`).

### Step 4: Quality review (automatic — surface findings, don't pre-ask)

Inspect the **added lines** of the diff (the `+` lines) so you only judge new code, not pre-existing content. Run these scans automatically; raise a finding only when there's something to act on.

#### 4a. Redundant comment noise — the key check

AI agents (and rushed humans) tend to over-comment, narrating code that already speaks for itself. Most such comments add nothing because the code is self-explanatory. Flag newly added comments that carry no information beyond the code:

**Flag for removal (high-confidence noise):**

- Comments that **restate the code**: `count++ // increment count`; `// set the name` above `this.name = name`.
- **Step/play-by-play narration**: `// Loop through users`, `// Return the result`, `// Import dependencies`, `// First, ...`, `// Now we ...`, `// Step 1: ...`.
- Comments/docblocks that just **echo the function or variable name**: `// getUser — gets the user`.
- **Commented-out / dead code** introduced by this change.
- Decorative **section banners** with no content, unless the file already uses that style.

**Always keep:**

- Comments that explain **why** — rationale, trade-offs, constraints, invariants.
- **Workarounds, bug references, links** (`// Workaround for <bug>; remove after <cond>`).
- Notes on **non-obvious behavior, gotchas, security/perf** considerations.
- **Public API docs** the project's tooling/convention expects (JSDoc, PHPDoc on public methods, Python docstrings) where the repo uses them.
- **License/legal headers** and **actionable `TODO`/`FIXME`** with context.

Honor any repo-specific rule (e.g. a `CLAUDE.md` that says "prefer docblocks, avoid inline comments" → be stricter).

When in doubt, **keep it** and merely list it as a suggestion. Present findings as `file:line — <comment> — why it's noise`, then offer to remove the clearly-redundant ones. Only auto-remove the high-confidence set if the user has said something like "just clean it up." After removing, re-stage the affected files.

#### 4b. Debug / leftover artifacts

Flag newly added debugging leftovers: `console.log`/`console.debug`, `dd()`/`dump()`/`var_dump()`/`ray()`, `debugger`, ad-hoc `print()` debug lines, and focused tests (`.only`, `fdescribe`, `it.only`). Offer to remove, same conservative rule.

#### 4c. Security risk scan (risk-triggered — don't ask up front)

Scan the added lines for:

1. **Hardcoded secrets** — API keys, tokens, passwords, connection strings, private keys, high-entropy literals.
2. **Injection** — SQL/command/XSS from unsanitized input.
3. **Sensitive data exposure** — credentials in logs, PII in debug output, secrets in comments.
4. **Auth/authz** — removed permission checks, auth-bypass risks.
5. **Input validation** — missing validation, path traversal.

If nothing trips, stay silent and move on. If something trips, list each as `file:line — issue` and ask whether to fix first or commit anyway. Never commit a hardcoded secret without an explicit go-ahead.

#### 4d. Documentation

Only when the change plausibly needs it (new feature, public API, config, command, or behavior change), ask whether docs should be updated and where (`CLAUDE.md` / `README.md` / a docs file). For routine fixes and refactors, skip silently.

### Step 5: Apply fixes and re-stage

For anything the user approves (comment cleanup, debug removal, security fix, docs), apply it with the Edit tool, then re-stage the touched files (`git add <files>`) so the commit reflects the cleaned-up state. Re-run `git diff --staged` if you changed anything.

### Step 6: Compose the commit message

Match the convention learned in Step 2. Default conventional shape:

```
type(scope): concise summary in the imperative

Optional body explaining WHY — the problem, context, trade-offs.
Wrap the body at 72 chars. Omit it entirely for small, self-evident changes.
```

- **Types:** `feat`, `fix`, `docs`, `style`, `refactor`, `perf`, `test`, `chore`, `build`, `ci`.
- Pick a **scope** from the touched area if the repo uses scopes.
- **Subject line** ≤ ~72 chars *including* the `type(scope):` prefix; imperative mood; no trailing period.
- Add trailers (e.g. `Co-Authored-By`) ONLY if the repo's history uses them or the user asks.
- If the change spans **unrelated concerns**, suggest splitting it into separate commits.

### Step 7: Commit and handle hook failures

```bash
git commit -m "$(cat <<'EOF'
type(scope): summary

Body (if any).
EOF
)"
```

- The quoted `<<'EOF'` keeps the message literal — do not interpolate variables into it.
- If the commit **fails** (a pre-commit hook rejected it or reformatted files), show the hook output. If a hook modified files, re-stage them and retry once; otherwise surface the error and let the user fix it. Do not report success unless `git commit` actually succeeded.

Confirm the result:

```bash
git status
git log -1 --stat
```
