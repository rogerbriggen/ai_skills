# Jira Skills

Skills for reading and updating Jira work items via the Jira Cloud REST API.

## Skills

| Skill file | Claude Code slash command | Description |
|---|---|---|
| `jira-read.md` | `/jira-read` | Read a work item: title, description, state, subtasks |
| `jira-update-state.md` | `/jira-update-state` | Update the state/status of a work item |

## Setup

### 1. Get your Jira API token

1. Log in to [Atlassian account settings](https://id.atlassian.com/manage-profile/security/api-tokens)
2. Click **Create API token**
3. Give it a label (e.g., `claude-code`) and copy the token

### 2. Set environment variables

Add the following to your shell profile (`.bashrc`, `.zshrc`, etc.) or to your Claude Code environment:

```bash
export JIRA_BASE_URL="https://yourcompany.atlassian.net"
export JIRA_EMAIL="your.email@example.com"
export JIRA_API_TOKEN="your-api-token-here"
```

For Claude Code projects, you can also add these to a `.env` file in your project root (make sure it is in `.gitignore`).

### 3. Install the skills

#### Claude Code

Copy the `.md` skill files to your Claude Code commands directory:

```bash
# Global installation (available in all projects)
cp skills/jira/jira-read.md ~/.claude/commands/jira-read.md
cp skills/jira/jira-update-state.md ~/.claude/commands/jira-update-state.md

# Project-local installation
mkdir -p .claude/commands
cp skills/jira/jira-read.md .claude/commands/jira-read.md
cp skills/jira/jira-update-state.md .claude/commands/jira-update-state.md
```

Then use the skills in Claude Code:

```
/jira-read PROJ-123
/jira-update-state PROJ-123 "In Progress"
```

#### GitHub Copilot

GitHub Copilot does not have a native slash command mechanism equivalent to Claude Code. To use these skills with Copilot Chat, paste the content of the skill file directly into the chat, replacing `$ARGUMENTS` with your actual values.

Alternatively, reference the skill files in `.github/copilot-instructions.md` to make Copilot aware of the Jira interaction patterns.

## Usage Examples

### Read a work item

```
/jira-read PROJ-123
```

Displays:
- Issue key, title, type, priority, assignee
- Current status
- Full description (converted from Atlassian Document Format)
- All subtasks with their current status

### Update work item state

```
/jira-update-state PROJ-123 "In Progress"
/jira-update-state PROJ-123 Done
```

The skill fetches available transitions for the issue, matches the requested state name (case-insensitive), and applies the transition.

## Troubleshooting

| Error | Likely cause | Fix |
|---|---|---|
| `401 Unauthorized` | Wrong credentials | Check `JIRA_EMAIL` and `JIRA_API_TOKEN` |
| `403 Forbidden` | Insufficient permissions | Ensure your account can view/edit the issue |
| `404 Not Found` | Wrong issue key or base URL | Check `JIRA_BASE_URL` and the issue key |
| No matching transition | State name not valid | Use `/jira-read` first to see the current state, then check Jira for valid transitions |

## API Reference

These skills use the [Jira Cloud REST API v3](https://developer.atlassian.com/cloud/jira/platform/rest/v3/intro/).

Key endpoints used:
- `GET /rest/api/3/issue/{issueIdOrKey}` — Fetch issue details
- `GET /rest/api/3/issue/{issueIdOrKey}/transitions` — List available transitions
- `POST /rest/api/3/issue/{issueIdOrKey}/transitions` — Apply a transition
