# Jira Plugin

This plugin packages Jira-focused skills for Claude Code and GitHub Copilot.

## Included Skills

| Skill | Folder | Purpose |
| --- | --- | --- |
| `jira-read` | `jira-read/` | Read a Jira issue and summarize the important fields |
| `jira-update-state` | `jira-update-state/` | Transition a Jira issue to a new workflow state |

## Structure

Each skill lives in its own folder and exposes an uppercase `SKILL.md` file:

```text
plugins/jira/
├── .claude-plugin/plugin.json
├── README.md
├── jira-read/
│   └── SKILL.md
└── jira-update-state/
    └── SKILL.md
```

## Setup

The skills automatically choose the best available access method:

| Condition | Method used |
| --- | --- |
| `JIRA_API_TOKEN` is set | Jira Cloud REST API v3 |
| `JIRA_API_TOKEN` is not set | Atlassian CLI (`acli`) |

### REST API

Set these environment variables:

```bash
export JIRA_BASE_URL="https://yourcompany.atlassian.net"
export JIRA_EMAIL="your.email@example.com"
export JIRA_API_TOKEN="your_api_token_here"
```

Generate a token at <https://id.atlassian.com/manage-profile/security/api-tokens>.

### Atlassian CLI

Install and authenticate `acli` if you do not want to provide `JIRA_API_TOKEN`:

```bash
npm install -g @atlassian/acli
acli configure
```

## Install

### Claude Code

Install the whole plugin, or copy individual skill folders into `.claude/skills/` in your project or home directory.

### GitHub Copilot

Install the whole plugin, or copy individual skill folders into `.github/skills/` in your repository or `~/.copilot/skills/` for user-wide usage.

## Usage Examples

```text
/jira-read PROJECT-42
/jira-update-state PROJECT-42 In Progress
/jira-update-state PROJECT-42 Done
```

## Troubleshooting

| Problem | Solution |
| --- | --- |
| `401 Unauthorized` | Check `JIRA_EMAIL` and `JIRA_API_TOKEN` |
| `404 Not Found` | Verify `JIRA_BASE_URL` and the issue key |
| `acli: command not found` | Install `@atlassian/acli` |
| Invalid state | Review the issue and available transitions in Jira |
