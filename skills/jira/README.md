# Jira Skills

Claude Code and GitHub Copilot skills for interacting with Jira.

These skills automatically choose the best available access method:

| Condition                        | Method used                                             |
| -------------------------------- | ------------------------------------------------------- |
| `JIRA_API_TOKEN` is set          | Jira Cloud REST API v3                                  |
| `JIRA_API_TOKEN` is **not** set  | Official Atlassian CLI (`acli`) with stored credentials |

## Available Skills

| Skill                                        | Command                                       | Description                            |
| -------------------------------------------- | --------------------------------------------- | -------------------------------------- |
| [jira-read](./jira-read.md)                  | `/jira-read PROJECT-123`                      | Read a Jira issue                      |
| [jira-update-state](./jira-update-state.md)  | `/jira-update-state PROJECT-123 In Progress`  | Transition a Jira issue to a new state |

## Setup

### Option 1 — REST API (recommended for automation)

Set the following environment variables in your shell profile (`~/.bashrc`, `~/.zshrc`, etc.):

```bash
export JIRA_BASE_URL="https://yourcompany.atlassian.net"
export JIRA_EMAIL="your.email@example.com"
export JIRA_API_TOKEN="your_api_token_here"
```

Generate an API token at <https://id.atlassian.com/manage-profile/security/api-tokens>.

### Option 2 — Official Atlassian CLI (`acli`)

Used automatically when `JIRA_API_TOKEN` is **not** set in your environment. The `acli` path uses credentials saved by `acli configure`, so you do not need to export `JIRA_API_TOKEN` when using `acli`.

#### Install

```bash
npm install -g @atlassian/acli
```

#### Authenticate

```bash
acli configure
```

You will be prompted for your Atlassian site URL, account email, and an API token. `acli` stores these credentials in its own config for future use.

## Install the Skills

**Claude Code** — install the skills using either of these options:

**Option A:** copy the `.md` files into your project's `.claude/commands/` directory:

```bash
cp skills/jira/*.md /path/to/your/project/.claude/commands/
```

**Option B:** reference the skill files from `.claude/settings.json`:

```json
{
  "commands": {
    "jira-read": "skills/jira/jira-read.md",
    "jira-update-state": "skills/jira/jira-update-state.md"
  }
}
```

With either option, the skills are then available as `/jira-read` and `/jira-update-state`.

**GitHub Copilot** — reference the skill files in your Copilot chat session or add them to your workspace context.

## Usage Examples

```text
/jira-read PROJECT-42
/jira-update-state PROJECT-42 Done
/jira-update-state PROJECT-42 In Progress
```

## Troubleshooting

| Problem                   | Solution                                                         |
| ------------------------- | ---------------------------------------------------------------- |
| `401 Unauthorized` (API)  | Check `JIRA_EMAIL` and `JIRA_API_TOKEN`                          |
| `404 Not Found` (API)     | Verify `JIRA_BASE_URL` and the issue key                         |
| `acli: command not found` | Run `npm install -g @atlassian/acli`                             |
| `acli` auth error         | Run `acli configure` to re-authenticate                          |
| Invalid state             | Run `/jira-read PROJECT-123`; check valid transitions in Jira UI |
