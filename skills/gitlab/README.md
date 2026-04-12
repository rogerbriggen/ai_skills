# GitLab Self-Hosted Skills

Claude Code and GitHub Copilot skills for interacting with a self-hosted GitLab instance via the [GitLab REST API v4](https://docs.gitlab.com/api/rest/) and the [GitLab CLI (`glab`)](https://docs.gitlab.com/cli/).

## Available Skills

| Skill | Command | Description |
|-------|---------|-------------|
| [gitlab-mr-create](./gitlab-mr-create.md) | `/gitlab-mr-create --source feature/x --target main --title "My MR"` | Create a new merge request |
| [gitlab-mr-diff](./gitlab-mr-diff.md) | `/gitlab-mr-diff 42` | Show the diff of a merge request for review |
| [gitlab-mr-checkout](./gitlab-mr-checkout.md) | `/gitlab-mr-checkout 42` | Check out a merge request branch locally |
| [gitlab-mr-assign-reviewer](./gitlab-mr-assign-reviewer.md) | `/gitlab-mr-assign-reviewer 42` | Assign yourself as a reviewer on a merge request |
| [gitlab-mr-line-note](./gitlab-mr-line-note.md) | `/gitlab-mr-line-note 42 src/main.go 15 "Comment"` | Create an inline review note on a specific line |
| [gitlab-mr-summary](./gitlab-mr-summary.md) | `/gitlab-mr-summary 42` | Generate an AI summary and post it as a general discussion note |

## Setup

### 1. Create a GitLab Personal Access Token

1. Go to your GitLab instance → **User Settings** → **Access Tokens**
2. Click **Add new token**
3. Give it a name (e.g. `claude-code`), set an expiry, and select the **`api`** scope
4. Click **Create personal access token** and copy the token

For read-only operations, the `read_api` scope is sufficient.

### 2. Set Environment Variables

Export the following in your shell profile (e.g. `~/.bashrc`, `~/.zshrc`) or set them in your environment:

```bash
export GITLAB_HOST="https://gitlab.example.com"   # your self-hosted GitLab URL
export GITLAB_TOKEN="your_personal_access_token"
```

### 3. (Optional) Install and Configure the GitLab CLI

Several skills can use the `glab` CLI as a simpler alternative to raw API calls.

```bash
# Install (see https://gitlab.com/gitlab-org/cli for platform-specific instructions)
# macOS:
brew install glab

# Authenticate against your self-hosted instance:
glab auth login --hostname gitlab.example.com
```

### 4. Install the Skills

**Claude Code** — copy the `.md` files into your project's `.claude/commands/` directory so they are available as slash commands:

```bash
cp skills/gitlab/*.md /path/to/your/project/.claude/commands/
```

Or add them to your `.claude/settings.json`:

```json
{
  "skills": [
    "path/to/ai_skills/skills/gitlab/gitlab-mr-create.md",
    "path/to/ai_skills/skills/gitlab/gitlab-mr-diff.md",
    "path/to/ai_skills/skills/gitlab/gitlab-mr-checkout.md",
    "path/to/ai_skills/skills/gitlab/gitlab-mr-assign-reviewer.md",
    "path/to/ai_skills/skills/gitlab/gitlab-mr-line-note.md",
    "path/to/ai_skills/skills/gitlab/gitlab-mr-summary.md"
  ]
}
```

**GitHub Copilot** — reference the skill files in your Copilot chat session or add them to your workspace context.

## Usage Examples

```
/gitlab-mr-create --source feature/login --target main --title "Add login page"
/gitlab-mr-diff 42
/gitlab-mr-checkout 42
/gitlab-mr-assign-reviewer 42
/gitlab-mr-line-note 42 src/auth.go 87 "Consider using a constant here instead of a magic string."
/gitlab-mr-summary 42
```

## Notes on Project Path Resolution

All skills automatically derive the GitLab project path from the current git remote:

```bash
git remote get-url origin
# e.g. https://gitlab.example.com/my-group/my-project.git
# → project path: my-group/my-project
# → URL-encoded: my-group%2Fmy-project
```

Run the skills from inside the git repository you want to work with.

## Troubleshooting

| Problem | Solution |
|---------|----------|
| `401 Unauthorized` | Check that `GITLAB_TOKEN` is correct and has the `api` scope |
| `404 Not Found` | Verify `GITLAB_HOST` and that you are inside the correct git repository |
| `403 Forbidden` | Your token may lack the required scope; recreate it with `api` scope |
| Inline note fails | The line may not be part of the diff; use `/gitlab-mr-summary` for a general note instead |
| `glab` not found | Install `glab` or use the REST API path (both options are documented in each skill) |
