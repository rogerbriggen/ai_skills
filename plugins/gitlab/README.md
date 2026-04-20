# GitLab Plugin

This plugin packages self-hosted GitLab merge request skills for Claude Code and GitHub Copilot.

## Included Skills

| Skill | Folder | Purpose |
| --- | --- | --- |
| `gitlab-mr-create` | `skills/gitlab-mr-create/` | Create a merge request from the current repository |
| `gitlab-mr-diff` | `skills/gitlab-mr-diff/` | Show the diff for an existing merge request |
| `gitlab-mr-checkout` | `skills/gitlab-mr-checkout/` | Check out a merge request branch locally |
| `gitlab-mr-assign-reviewer` | `skills/gitlab-mr-assign-reviewer/` | Add yourself as a reviewer on a merge request |
| `gitlab-mr-line-note` | `skills/gitlab-mr-line-note/` | Add an inline comment to a diff line |
| `gitlab-mr-summary` | `skills/gitlab-mr-summary/` | Generate and post an AI summary of a merge request |

## Structure

Each skill lives in its own folder and exposes an uppercase `SKILL.md` file:

```text
plugins/gitlab/
├── .claude-plugin/plugin.json
├── README.md
└── skills/
    ├── gitlab-mr-create/
    │   └── SKILL.md
    ├── gitlab-mr-diff/
    │   └── SKILL.md
    ├── gitlab-mr-checkout/
    │   └── SKILL.md
    ├── gitlab-mr-assign-reviewer/
    │   └── SKILL.md
    ├── gitlab-mr-line-note/
    │   └── SKILL.md
    └── gitlab-mr-summary/
        └── SKILL.md
```

## Setup

The skills select their access method from the `GITLAB_TOKEN` environment variable:

| Condition | Method used |
| --- | --- |
| `GITLAB_TOKEN` is set | GitLab REST API |
| `GITLAB_TOKEN` is not set | GitLab CLI (`glab`) |

### GitLab CLI

Install and authenticate `glab` if you want the CLI path:

```bash
# macOS
brew install glab

# Windows
winget install glab

# Login to a self-hosted instance
glab auth login --hostname gitlab.example.com
```

### REST API

Set these environment variables if you want the API path:

```bash
export GITLAB_HOST="https://gitlab.example.com"
export GITLAB_TOKEN="your_personal_access_token"
```

The token should have the `api` scope. `read_api` is enough for read-only operations.

## Install

### Claude Code

Install the whole plugin, or copy individual skill folders from `skills/` into `.claude/skills/`.

### GitHub Copilot

Install the whole plugin, or copy individual skill folders from `skills/` into `.github/skills/` in a repository or `~/.copilot/skills/` for user-wide use.

## Usage Examples

```text
/gitlab-mr-create --source feature/login --target main --title "Add login page"
/gitlab-mr-diff 42
/gitlab-mr-checkout 42
/gitlab-mr-assign-reviewer 42
/gitlab-mr-line-note 42 src/auth.go 87 "Consider using a constant here."
/gitlab-mr-summary 42
```

## Troubleshooting

| Problem | Solution |
| --- | --- |
| `401 Unauthorized` | Check `GITLAB_TOKEN` and required scopes |
| `404 Not Found` | Verify `GITLAB_HOST` and the current repository remote |
| `glab` not found | Install `glab` or set `GITLAB_TOKEN` |
| Inline note fails | The target line may not be part of the diff |
| Wrong tool selected | Unset `GITLAB_TOKEN` to force the CLI path |
