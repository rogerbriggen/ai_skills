# AI skills for Claude Code and Github Copilot

This repository stores reusable skills (slash commands / prompt files) that extend Claude Code and GitHub Copilot with helpful workflows.

## Available Skills

### Jira

Skills for interacting with Jira. Automatically uses the Jira REST API when `JIRA_API_TOKEN` is set, and falls back to the official [Atlassian CLI (`acli`)](https://developer.atlassian.com/cloud/acli/) otherwise. See [`skills/jira/README.md`](skills/jira/README.md) for setup instructions.

| Skill | Command | Description |
| - | - | - |
| [jira-read](skills/jira/jira-read.md) | `/jira-read PROJECT-123` | Read a Jira issue: type, status, priority, assignee, description, and subtasks |
| [jira-update-state](skills/jira/jira-update-state.md) | `/jira-update-state PROJECT-123 In Progress` | Transition a Jira issue to a new state |

### GitLab (Self-Hosted)

Skills for interacting with a self-hosted GitLab instance. Automatically uses the GitLab REST API when `GITLAB_TOKEN` is set, and falls back to the [GitLab CLI (`glab`)](https://docs.gitlab.com/cli/) otherwise. See [`skills/gitlab/README.md`](skills/gitlab/README.md) for setup instructions.

| Skill | Command | Description |
| - | - | - |
| [gitlab-mr-create](skills/gitlab/gitlab-mr-create.md) | `/gitlab-mr-create --source feature/x --target main --title "My MR"` | Create a new merge request |
| [gitlab-mr-diff](skills/gitlab/gitlab-mr-diff.md) | `/gitlab-mr-diff 42` | Show the diff of a merge request for review |
| [gitlab-mr-checkout](skills/gitlab/gitlab-mr-checkout.md) | `/gitlab-mr-checkout 42` | Check out a merge request branch locally |
| [gitlab-mr-assign-reviewer](skills/gitlab/gitlab-mr-assign-reviewer.md) | `/gitlab-mr-assign-reviewer 42` | Assign yourself as a reviewer on a merge request |
| [gitlab-mr-line-note](skills/gitlab/gitlab-mr-line-note.md) | `/gitlab-mr-line-note 42 src/main.go 15 "Comment"` | Create an inline review note on a specific line |
| [gitlab-mr-summary](skills/gitlab/gitlab-mr-summary.md) | `/gitlab-mr-summary 42` | Generate an AI summary and post it as a general discussion note |

## GitHub Actions: Claude Code Integration

The workflow in [`.github/workflows/claude.yml`](.github/workflows/claude.yml) enables Claude Code to respond to `@claude` mentions in issues and pull requests.

See [`.github/copilot-instructions.md`](.github/copilot-instructions.md) for full details on customization.
