# AI skills for Claude Code and GitHub Copilot

This repository now publishes two separate plugins through the marketplace
manifest in `.claude-plugin/marketplace.json`:

- `jira-skills` from `plugins/jira`
- `gitlab-skills` from `plugins/gitlab`

Each individual skill lives in its own folder and exposes an uppercase
`SKILL.md` file that matches the Skills Directory structure.

## Marketplace Plugins

| Plugin | Source | Purpose |
| --- | --- | --- |
| `jira-skills` | `plugins/jira` | Jira issue reading and workflow transitions |
| `gitlab-skills` | `plugins/gitlab` | Self-hosted GitLab merge request workflows |

## Skills format and Agent plugins

Format of a SKILL.md file, see [https://www.skillsdirectory.com/docs/skill-file-structure]

Infos about agent plugins, see  [https://code.visualstudio.com/docs/copilot/customization/agent-plugins#_what-plugins-provide]

## Repository Layout

```text
.claude-plugin/
	marketplace.json
plugins/
	jira/
		.claude-plugin/plugin.json
		README.md
		skills/
			jira-read/SKILL.md
			jira-update-state/SKILL.md
	gitlab/
		.claude-plugin/plugin.json
		README.md
		skills/
			gitlab-mr-list-open/SKILL.md
			gitlab-mr-create/SKILL.md
			gitlab-mr-diff/SKILL.md
			gitlab-mr-checkout/SKILL.md
			gitlab-mr-assign-reviewer/SKILL.md
			gitlab-mr-line-note/SKILL.md
			gitlab-mr-summary/SKILL.md
```

## Install as Plugins

Use a plugin client that understands the `.claude-plugin/marketplace.json`
format and install either `jira-skills` or `gitlab-skills` from this
repository. Each plugin has its own metadata file at
`plugins/<plugin>/.claude-plugin/plugin.json`.

Use /plugin to manage marketplaces and install plugins.

Copilot users: Start with /plugin marketplace add rogerbriggen/ai_skills

## Install Manually

### GitHub Copilot

Copy the plugin folder or individual skill folders into one of these locations:

- `~/.copilot/skills/` for user-wide usage
- `.github/skills/` inside a repository for repo-specific usage

For example, copying `plugins/jira/skills/jira-read/` into `.github/skills/` makes the
`jira-read` skill available in that repository.

See also: <https://code.visualstudio.com/docs/copilot/customization/agent-skills#_create-a-skill>

### Claude Code

Copy the plugin folder or individual skill folders into one of these locations:

- `~/.claude/skills/` for user-wide usage
- `.claude/skills/` inside a repository for repo-specific usage

See also: <https://code.claude.com/docs/en/skills#where-skills-live>

## Plugin Details

### Jira

The Jira plugin uses the Jira REST API when `JIRA_API_TOKEN` is set and falls
back to the Atlassian CLI (`acli`) otherwise. Setup, examples, and
troubleshooting live in `plugins/jira/README.md`.

| Skill | Folder | Description |
| --- | --- | --- |
| `jira-read` | `plugins/jira/skills/jira-read/` | Read a Jira issue and summarize its key fields |
| `jira-update-state` | `plugins/jira/skills/jira-update-state/` | Transition a Jira issue to a new state |

### GitLab

The GitLab plugin uses the GitLab REST API when `GITLAB_TOKEN` is set and falls
back to the GitLab CLI (`glab`) otherwise. Setup, examples, and troubleshooting
live in `plugins/gitlab/README.md`.

| Skill | Folder | Description |
| --- | --- | --- |
| `gitlab-mr-list-open` | `plugins/gitlab/skills/gitlab-mr-list-open/` | List all open merge requests for the current project |
| `gitlab-mr-create` | `plugins/gitlab/skills/gitlab-mr-create/` | Create a merge request |
| `gitlab-mr-diff` | `plugins/gitlab/skills/gitlab-mr-diff/` | Show the diff of a merge request |
| `gitlab-mr-checkout` | `plugins/gitlab/skills/gitlab-mr-checkout/` | Check out a merge request branch locally |
| `gitlab-mr-assign-reviewer` | `plugins/gitlab/skills/gitlab-mr-assign-reviewer/` | Add yourself as a reviewer on a merge request |
| `gitlab-mr-line-note` | `plugins/gitlab/skills/gitlab-mr-line-note/` | Create an inline review note on a specific diff line |
| `gitlab-mr-summary` | `plugins/gitlab/skills/gitlab-mr-summary/` | Generate and post an AI summary of a merge request |

## GitHub Actions: Claude Code Integration

The workflow in `.github/workflows/claude.yml` enables Claude Code to respond to
`@claude` mentions in issues and pull requests.

See `.github/copilot-instructions.md` for repository-specific guidance.
