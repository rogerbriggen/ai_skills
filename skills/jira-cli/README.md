# Jira CLI Skills

Claude Code and GitHub Copilot skills for interacting with Jira via the [Jira CLI (`jira`)](https://github.com/ankitpokhrel/jira-cli) with automatic fallback to the [Jira Cloud REST API v3](https://developer.atlassian.com/cloud/jira/platform/rest/v3/).

## Available Skills

| Skill | Command | Description |
|-------|---------|-------------|
| [jira-cli-read](./jira-cli-read.md) | `/jira-cli-read PROJECT-123` | Read a Jira issue: title, type, status, priority, assignee, description, and subtasks |
| [jira-cli-update-state](./jira-cli-update-state.md) | `/jira-cli-update-state PROJECT-123 In Progress` | Transition a Jira issue to a new state |

## Access Method Selection

Each skill automatically selects how to talk to Jira at runtime:

| Condition | Access method |
|-----------|--------------|
| `JIRA_API_TOKEN` **is set** | Jira Cloud REST API v3 (HTTP Basic auth) |
| `JIRA_API_TOKEN` **is not set** | Jira CLI (`jira`) |

This means you can use the skills in environments where only the CLI is configured (no API token required) or in environments where an API token is available for direct API access.

## Setup: Jira CLI

### 1. Install the `jira` CLI

```bash
# macOS
brew install ankitpokhrel/jira-cli/jira-cli

# Linux (see https://github.com/ankitpokhrel/jira-cli for platform-specific instructions)
# Or via Go:
go install github.com/ankitpokhrel/jira-cli/cmd/jira@latest
```

### 2. Configure and authenticate

```bash
jira init
```

Follow the interactive prompts to enter your Jira instance URL, account email, and API token. Configuration is saved to `~/.config/.jira/.config.yml`.

To verify the setup:

```bash
jira issue list
```

## Setup: REST API (optional)

If you prefer to use the REST API directly (or want to override the CLI path), export the following environment variables in your shell profile (`~/.bashrc`, `~/.zshrc`, etc.):

```bash
export JIRA_BASE_URL="https://yourcompany.atlassian.net"
export JIRA_EMAIL="your.email@example.com"
export JIRA_API_TOKEN="your_api_token_here"
```

To create an API token: go to [https://id.atlassian.com/manage-profile/security/api-tokens](https://id.atlassian.com/manage-profile/security/api-tokens), click **Create API token**, give it a label (e.g. `claude-code`), and copy the generated token.

## Install the Skills

**Claude Code** — copy the `.md` files into your project's `.claude/commands/` directory so they are available as slash commands:

```bash
cp skills/jira-cli/*.md /path/to/your/project/.claude/commands/
```

Or add them to your `.claude/settings.json`:

```json
{
  "skills": [
    "path/to/ai_skills/skills/jira-cli/jira-cli-read.md",
    "path/to/ai_skills/skills/jira-cli/jira-cli-update-state.md"
  ]
}
```

**GitHub Copilot** — reference the skill files in your Copilot chat session or add them to your workspace context.

## Usage Examples

```
/jira-cli-read PROJECT-42
/jira-cli-update-state PROJECT-42 Done
/jira-cli-update-state PROJECT-42 In Progress
```

## Troubleshooting

| Problem | Solution |
|---------|----------|
| `jira: command not found` | Install the `jira` CLI (see [Setup: Jira CLI](#setup-jira-cli)) |
| `401 Unauthorized` (REST API) | Check that `JIRA_EMAIL` and `JIRA_API_TOKEN` are correct |
| `404 Not Found` (REST API) | Verify `JIRA_BASE_URL` and the issue key |
| `No transition found` (REST API) | Run `/jira-cli-read` to confirm the issue exists; the update skill lists valid transitions |
| CLI transition not found | Run `jira issue move PROJECT-123` without a state to see valid transitions interactively |
| CLI not authenticated | Run `jira init` to (re-)configure your instance and credentials |

## Differences from `skills/jira`

The `skills/jira` skills require `JIRA_API_TOKEN` to be set and always use the REST API.  
These `skills/jira-cli` skills work **without** `JIRA_API_TOKEN` by falling back to the `jira` CLI, making them suitable for environments where the CLI is already configured but an API token is not available in the environment.
