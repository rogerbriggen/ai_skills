# Jira CLI Skills

Claude Code and GitHub Copilot skills for interacting with Jira using the
**official [Atlassian CLI (`acli`)](https://developer.atlassian.com/cloud/acli/)**.

These skills automatically choose the best available access method:

| Condition | Method used |
|-----------|-------------|
| `JIRA_API_TOKEN` is set | Jira Cloud REST API v3 |
| `JIRA_API_TOKEN` is **not** set | Official Atlassian CLI (`acli`) |

## Available Skills

| Skill | Command | Description |
|-------|---------|-------------|
| [jira-cli-read](./jira-cli-read.md) | `/jira-cli-read PROJECT-123` | Read a Jira issue: key, title, type, status, priority, assignee, description, and subtasks |
| [jira-cli-update-state](./jira-cli-update-state.md) | `/jira-cli-update-state PROJECT-123 In Progress` | Transition a Jira issue to a new state |

## Setup

### Option 1 — REST API (recommended for automation)

Set the following environment variables in your shell profile (`~/.bashrc`, `~/.zshrc`, etc.):

```bash
export JIRA_BASE_URL="https://yourcompany.atlassian.net"
export JIRA_EMAIL="your.email@example.com"
export JIRA_API_TOKEN="your_api_token_here"
```

Generate an API token at <https://id.atlassian.com/manage-profile/security/api-tokens>.

### Option 2 — Official Atlassian CLI (`acli`)

Used automatically when `JIRA_API_TOKEN` is **not** set.

#### Install

```bash
npm install -g @atlassian/acli
```

#### Authenticate

Run the interactive configuration wizard:

```bash
acli configure
```

You will be prompted for:
- Your Atlassian site URL (e.g. `https://yourcompany.atlassian.net`)
- Your Atlassian account email
- An Atlassian API token (generated at <https://id.atlassian.com/manage-profile/security/api-tokens>)

Verify the installation:

```bash
acli --version
acli jira workitem view --id PROJECT-123
```

#### acli reference

| Operation | Command |
|-----------|---------|
| View a workitem | `acli jira workitem view --id PROJECT-123` |
| Transition a workitem | `acli jira workitem transition --id PROJECT-123 --state "In Progress"` |

Full reference: <https://developer.atlassian.com/cloud/acli/reference/commands/>

## Install the Skills

**Claude Code** — copy the `.md` files into your project's `.claude/commands/` directory:

```bash
cp skills/jira-cli/*.md /path/to/your/project/.claude/commands/
```

They are then available as `/jira-cli-read` and `/jira-cli-update-state`.

Alternatively, reference them via your `.claude/settings.json`:

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
| `401 Unauthorized` (API) | Check `JIRA_EMAIL` and `JIRA_API_TOKEN` |
| `404 Not Found` (API) | Verify `JIRA_BASE_URL` and the issue key |
| `acli: command not found` | Run `npm install -g @atlassian/acli` |
| `acli` auth error | Run `acli configure` to re-authenticate |
| Invalid state (acli) | Check valid transitions via the Jira UI or REST API |
