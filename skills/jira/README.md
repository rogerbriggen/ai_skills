# Jira Skills

Claude Code and GitHub Copilot skills for interacting with Jira via the [Jira Cloud REST API v3](https://developer.atlassian.com/cloud/jira/platform/rest/v3/).

## Available Skills

| Skill | Command | Description |
|-------|---------|-------------|
| [jira-read](./jira-read.md) | `/jira-read PROJECT-123` | Read a Jira issue: title, type, status, priority, assignee, description, and subtasks |
| [jira-update-state](./jira-update-state.md) | `/jira-update-state PROJECT-123 In Progress` | Transition a Jira issue to a new state |

## Setup

### 1. Create an Atlassian API Token

1. Go to [https://id.atlassian.com/manage-profile/security/api-tokens](https://id.atlassian.com/manage-profile/security/api-tokens)
2. Click **Create API token**
3. Give it a label (e.g. `claude-code`) and copy the generated token

### 2. Set Environment Variables

Export the following variables in your shell profile (e.g. `~/.bashrc`, `~/.zshrc`) or set them in your environment:

```bash
export JIRA_BASE_URL="https://yourcompany.atlassian.net"
export JIRA_EMAIL="your.email@example.com"
export JIRA_API_TOKEN="your_api_token_here"
```

### 3. Install the Skills

**Claude Code** — add to your `.claude/settings.json` (user or project level):

```json
{
  "skills": [
    "path/to/ai_skills/skills/jira/jira-read.md",
    "path/to/ai_skills/skills/jira/jira-update-state.md"
  ]
}
```

Or copy the `.md` files into your project's `.claude/commands/` directory so they are available as `/jira-read` and `/jira-update-state`.

**GitHub Copilot** — reference the skill files in your Copilot chat session or add them to your workspace context.

## Usage Examples

```
/jira-read PROJECT-42
/jira-update-state PROJECT-42 Done
/jira-update-state PROJECT-42 In Progress
```

## Troubleshooting

| Problem | Solution |
|---------|----------|
| `401 Unauthorized` | Check that `JIRA_EMAIL` and `JIRA_API_TOKEN` are correct |
| `404 Not Found` | Verify `JIRA_BASE_URL` and the issue key |
| `No transition found` | Run `/jira-read` to confirm the issue exists; the update skill lists valid transitions |
