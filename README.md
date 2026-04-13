# ai_skills

Skills for Claude Code and GitHub Copilot.

## What's in this repo

This repository stores reusable skills (slash commands / prompt files) that extend Claude Code and GitHub Copilot with helpful workflows.

## Available Skills

### Jira (REST API)

Skills for interacting with Jira via the Jira Cloud REST API v3. See [`skills/jira/README.md`](skills/jira/README.md) for setup instructions.

| Skill | Command | Description |
|-------|---------|-------------|
| [jira-read](skills/jira/jira-read.md) | `/jira-read PROJECT-123` | Read a Jira issue: title, type, status, priority, assignee, description, and subtasks |
| [jira-update-state](skills/jira/jira-update-state.md) | `/jira-update-state PROJECT-123 In Progress` | Transition a Jira issue to a new state |

### Jira CLI (Official Atlassian CLI / REST API)

Skills that automatically use the official [Atlassian CLI (`acli`)](https://developer.atlassian.com/cloud/acli/) when `JIRA_API_TOKEN` is not set, and the Jira REST API otherwise. See [`skills/jira-cli/README.md`](skills/jira-cli/README.md) for setup instructions.

| Skill | Command | Description |
|-------|---------|-------------|
| [jira-cli-read](skills/jira-cli/jira-cli-read.md) | `/jira-cli-read PROJECT-123` | Read a Jira issue using `acli jira workitem view` or the REST API |
| [jira-cli-update-state](skills/jira-cli/jira-cli-update-state.md) | `/jira-cli-update-state PROJECT-123 In Progress` | Transition a Jira issue using `acli jira workitem transition` or the REST API |

### GitLab (Self-Hosted)

Skills for interacting with a self-hosted GitLab instance via the GitLab REST API v4 and `glab` CLI. See [`skills/gitlab/README.md`](skills/gitlab/README.md) for setup instructions.

| Skill | Command | Description |
|-------|---------|-------------|
| [gitlab-mr-create](skills/gitlab/gitlab-mr-create.md) | `/gitlab-mr-create --source feature/x --target main --title "My MR"` | Create a new merge request |
| [gitlab-mr-diff](skills/gitlab/gitlab-mr-diff.md) | `/gitlab-mr-diff 42` | Show the diff of a merge request for review |
| [gitlab-mr-checkout](skills/gitlab/gitlab-mr-checkout.md) | `/gitlab-mr-checkout 42` | Check out a merge request branch locally |
| [gitlab-mr-assign-reviewer](skills/gitlab/gitlab-mr-assign-reviewer.md) | `/gitlab-mr-assign-reviewer 42` | Assign yourself as a reviewer on a merge request |
| [gitlab-mr-line-note](skills/gitlab/gitlab-mr-line-note.md) | `/gitlab-mr-line-note 42 src/main.go 15 "Comment"` | Create an inline review note on a specific line |
| [gitlab-mr-summary](skills/gitlab/gitlab-mr-summary.md) | `/gitlab-mr-summary 42` | Generate an AI summary and post it as a general discussion note |

## GitHub Actions: Claude Code Integration

The workflow in [`.github/workflows/claude.yml`](.github/workflows/claude.yml) enables Claude Code to respond to `@claude` mentions in issues and pull requests.

See [`.github/copilot-instructions.md`](.github/copilot-instructions.md) for full details on customization.
