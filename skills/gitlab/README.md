# GitLab Self-Hosted Skills

Claude Code and GitHub Copilot skills for interacting with a self-hosted GitLab instance via the [GitLab REST API v4](https://docs.gitlab.com/api/rest/) and the [GitLab CLI (`glab`)](https://docs.gitlab.com/cli/).

## Available Skills

| Skill                                                       | Command                                                              | Description                                                     |
| ----------------------------------------------------------- | -------------------------------------------------------------------- | --------------------------------------------------------------- |
| [gitlab-mr-create](./gitlab-mr-create.md)                   | `/gitlab-mr-create --source feature/x --target main --title "My MR"` | Create a new merge request                                      |
| [gitlab-mr-diff](./gitlab-mr-diff.md)                       | `/gitlab-mr-diff 42`                                                 | Show the diff of a merge request for review                     |
| [gitlab-mr-checkout](./gitlab-mr-checkout.md)               | `/gitlab-mr-checkout 42`                                             | Check out a merge request branch locally                        |
| [gitlab-mr-assign-reviewer](./gitlab-mr-assign-reviewer.md) | `/gitlab-mr-assign-reviewer 42`                                      | Assign yourself as a reviewer on a merge request                |
| [gitlab-mr-line-note](./gitlab-mr-line-note.md)             | `/gitlab-mr-line-note 42 src/main.go 15 "Comment"`                   | Create an inline review note on a specific line                 |
| [gitlab-mr-summary](./gitlab-mr-summary.md)                 | `/gitlab-mr-summary 42`                                              | Generate an AI summary and post it as a general discussion note |

## Tool Selection

Each skill automatically selects which tool to use based on the `GITLAB_TOKEN` environment variable:

| Condition                     | Tool Used               | Notes                                                               |
| ----------------------------- | ----------------------- | ------------------------------------------------------------------- |
| `GITLAB_TOKEN` is **not set** | **GitLab CLI (`glab`)** | Authenticate once with `glab auth login`; no plaintext token in env |
| `GITLAB_TOKEN` **is set**     | **REST API via `curl`** | Requires `GITLAB_HOST` and `GITLAB_TOKEN` in environment            |

> **Security tip:** Using `glab` avoids storing a plaintext personal access token in your environment — `glab` manages credentials in its own secure config store.

## Setup

Choose **one** of the two approaches below. You do not need both.

### Option A: GitLab CLI (`glab`) — recommended

No environment token needed. `glab` stores credentials securely in its own config.

1. Install `glab` (see the [platform-specific instructions](https://gitlab.com/gitlab-org/cli)):

   ```bash
   # macOS
   brew install glab

   # Windows (winget)
   winget install glab

   # Linux (various package managers)
   # See https://gitlab.com/gitlab-org/cli#installation
   ```

2. Authenticate against your self-hosted instance:

   ```bash
   glab auth login --hostname gitlab.example.com
   ```

3. Do **not** set `GITLAB_TOKEN` — skills detect the absence of this variable and use `glab` automatically.

### Option B: REST API with curl

1. Create a GitLab Personal Access Token:
   - Go to your GitLab instance → **User Settings** → **Access Tokens**
   - Click **Add new token**
   - Give it a name (e.g. `claude-code`), set an expiry, and select the **`api`** scope
   - Click **Create personal access token** and copy the token

   For read-only operations, the `read_api` scope is sufficient.

2. Set the following environment variables in your shell profile (e.g. `~/.bashrc`, `~/.zshrc`, or Windows System Environment Variables):

   ```bash
   export GITLAB_HOST="https://gitlab.example.com"   # your self-hosted GitLab URL
   export GITLAB_TOKEN="your_personal_access_token"
   ```

### Install the Skills

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

When using `glab`, the project path is resolved automatically via the `:fullpath` shorthand.

## Troubleshooting

| Problem             | Solution                                                                                  |
| ------------------- | ----------------------------------------------------------------------------------------- |
| `401 Unauthorized`  | Check that `GITLAB_TOKEN` is correct and has the `api` scope                              |
| `404 Not Found`     | Verify `GITLAB_HOST` and that you are inside the correct git repository                   |
| `403 Forbidden`     | Your token may lack the required scope; recreate it with `api` scope                      |
| Inline note fails   | The line may not be part of the diff; use `/gitlab-mr-summary` for a general note instead |
| `glab` not found    | Install `glab` (Option A) or set `GITLAB_TOKEN` to use the REST API (Option B)            |
| `glab` auth error   | Run `glab auth login --hostname <your-gitlab-host>` to authenticate                       |
| Wrong tool selected | If `GITLAB_TOKEN` is set, curl is used; unset it to switch to `glab`                      |
