# Claude Code Plugins

A collection of Claude Code plugins by Ricardo Lobo.

## Plugins

| Plugin | Description |
|--------|-------------|
| [commit](#commit) | Smart commits with security review and documentation awareness |
| [sentry-mcp](#sentry-mcp) | Interact with Sentry for error monitoring and debugging |

---

## commit

A smart commit workflow that handles security reviews and documentation updates.

### Features

- **Trivial commit detection** - Skips questions for lockfiles, dependency updates, and .gitignore changes
- **Security review** - Optional check for hardcoded secrets, injection vulnerabilities, sensitive data exposure
- **Documentation awareness** - Prompts to update CLAUDE.md, README.md, or create new docs
- **Conventional commits** - Generates proper format with type(scope), detailed body, and Co-Authored-By footer

### Installation

```
/plugin install commit@ricardo-lobo
```

### Usage

Stage your changes, then run:

```
/commit
```

The skill will:
1. Analyze staged changes
2. Skip questions for trivial commits (lockfiles, deps)
3. For non-trivial changes, ask about security review and documentation
4. Generate and execute a conventional commit

---

## sentry-mcp

A plugin for Sentry self-hosted instances. Query issues, manage errors, and debug with Sentry context directly from Claude Code.

### Features

- **6 MCP Tools** for Sentry API operations
- **6 Slash Commands** for quick access
- **Debugging Skill** - workflow guidance for error investigation
- **LLM-Optimized Output** - summarized, actionable responses

### Prerequisites

#### 1. Create a Sentry Auth Token

1. **Open Sentry** and log in to your account
2. **Navigate to API tokens:**
   - Click your avatar (top-left) → **Settings**
   - In the left sidebar: **Auth Tokens** (under Account)
   - Or go directly to: `{SENTRY_URL}/settings/account/api/auth-tokens/`
3. **Create a new token:**
   - Click **"Create New Token"**
   - Give it a name (e.g., "Claude Code Plugin")
   - Select these scopes:
     - `project:read` - List projects and read project settings
     - `event:read` - Read issues and events
     - `event:write` - Update issue status (resolve/ignore)
   - Click **"Create Token"**
4. **Copy the token** - it will only be shown once!

#### 2. Configure Environment Variables

Create a `.env` file in your project root:

```bash
# Required
SENTRY_URL=https://sentry.your-company.com
SENTRY_AUTH_TOKEN=sntrys_eyJ...
SENTRY_ORG=my-org

# Optional - default project for commands
SENTRY_PROJECT=frontend
```

| Variable | Required | Description |
|----------|----------|-------------|
| `SENTRY_URL` | Yes | Your Sentry instance URL |
| `SENTRY_AUTH_TOKEN` | Yes | API token with required scopes |
| `SENTRY_ORG` | Yes | Organization slug |
| `SENTRY_PROJECT` | No | Default project for commands |

> **Note:** Bun automatically loads `.env` files. Add `.env` to your `.gitignore`.

### Installation

#### From GitHub (Marketplace-style)

```
/plugin marketplace add ricardo-lobo/claude-code
/plugin install sentry-mcp@ricardo-lobo
```

#### Manual Installation

```bash
# Clone the repository
git clone https://github.com/ricardo-lobo/claude-code.git ~/.claude/plugins/claude-code

# Install dependencies
cd ~/.claude/plugins/claude-code/sentry && bun install
```

Then use with:
```bash
claude --plugin-dir ~/.claude/plugins/claude-code/sentry
```

### Quick Start

1. **Set up your `.env` file** with the required variables (see above)

2. **List recent issues:**
   ```
   /sentry:issues
   ```

3. **Get issue details:**
   ```
   /sentry:issue FRONTEND-123
   ```

### Commands

| Command | Description |
|---------|-------------|
| `/sentry:setup` | Show environment variable configuration instructions |
| `/sentry:issues [project] [--status ...]` | List recent issues with filters |
| `/sentry:issue <id>` | Get detailed issue information |
| `/sentry:resolve <id> [--unresolve\|--ignore]` | Update issue status |
| `/sentry:find-event <event-id>` | Find issue by event ID |
| `/sentry:stats [project]` | Get project error statistics |

### MCP Tools

The plugin exposes these tools for programmatic access:

| Tool | Purpose |
|------|---------|
| `list_projects` | List all projects in an organization |
| `list_issues` | List issues with filters (project, status, query) |
| `get_issue` | Get detailed issue info (stack trace, tags, users) |
| `list_issue_events` | Get individual error occurrences |
| `update_issue_status` | Resolve, unresolve, or ignore issues |
| `get_project_stats` | Get error rates and top issues |

### Example Usage

Ask Claude naturally:
- "Show me unresolved issues in the frontend project"
- "What's the error rate for backend-api?"
- "Get details for issue FRONTEND-123 and help me debug it"
- "Resolve issue 456"

---

## Development

```bash
cd sentry
bun install      # Install dependencies
bun test         # Run tests (36 tests)
bun run start    # Run MCP server standalone
```

## Project Structure

```
claude-code/
├── .claude-plugin/
│   └── marketplace.json   # Marketplace registration
├── commit/                # commit plugin
│   ├── .claude-plugin/
│   │   └── plugin.json
│   └── skills/
│       └── commit/        # Smart commit skill
├── sentry/                # sentry-mcp plugin
│   ├── .claude-plugin/
│   │   └── plugin.json
│   ├── .mcp.json
│   ├── commands/          # 6 slash commands
│   ├── skills/            # Debugging skill
│   ├── src/               # MCP server (TypeScript)
│   └── tests/             # 36 tests
└── README.md
```

## License

MIT
