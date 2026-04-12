# ai_skills

Skills for Claude Code and GitHub Copilot.

## What's in this repo

This repository stores reusable skills (slash commands / prompt files) that extend Claude Code and GitHub Copilot with helpful workflows.

## Available Skills

### Jira

Skills for interacting with Jira via the Jira Cloud REST API v3. See [`skills/jira/README.md`](skills/jira/README.md) for setup instructions.

| Skill | Command | Description |
|-------|---------|-------------|
| [jira-read](skills/jira/jira-read.md) | `/jira-read PROJECT-123` | Read a Jira issue: title, type, status, priority, assignee, description, and subtasks |
| [jira-update-state](skills/jira/jira-update-state.md) | `/jira-update-state PROJECT-123 In Progress` | Transition a Jira issue to a new state |

## GitHub Actions: Claude Code Integration

The workflow in [`.github/workflows/claude.yml`](.github/workflows/claude.yml) enables Claude Code to respond to `@claude` mentions in issues and pull requests.

See [`.github/copilot-instructions.md`](.github/copilot-instructions.md) for full details on customization.
