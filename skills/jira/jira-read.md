# Jira Read

Read a Jira work item and display its details.

Uses the Jira REST API when `JIRA_API_TOKEN` is set; falls back to the official
Atlassian CLI (`acli`) when it is not.

## Arguments

`$ARGUMENTS` — the Jira issue key, e.g. `PROJECT-123`

## Instructions

1. Read the issue key from `$ARGUMENTS`.
2. Check whether `JIRA_API_TOKEN` is set in the environment.

### Path A — REST API (`JIRA_API_TOKEN` is set)

1. Require the following environment variables (fail with a clear message if any are missing):
   - `JIRA_BASE_URL` — e.g. `https://yourcompany.atlassian.net`
   - `JIRA_EMAIL` — your Atlassian account email
   - `JIRA_API_TOKEN` — your Atlassian API token
2. Call the Jira REST API v3:
   ```
   GET {JIRA_BASE_URL}/rest/api/3/issue/{issueKey}?fields=summary,description,status,issuetype,priority,assignee,parent,subtasks
   ```
   Use HTTP Basic auth with `{JIRA_EMAIL}:{JIRA_API_TOKEN}` (Base64-encoded).
3. Parse the JSON response and display in a readable format:
   - **Key**: issue key (e.g. `PROJECT-123`)
   - **Title**: `fields.summary`
   - **Type**: `fields.issuetype.name`
   - **Status**: `fields.status.name`
   - **Priority**: `fields.priority.name` (if present)
   - **Assignee**: `fields.assignee.displayName` (or "Unassigned" if null)
   - **Parent**: `fields.parent.key — fields.parent.fields.summary` (if present)
   - **Description**: convert ADF (Atlassian Document Format) `fields.description` to plain text — walk the `content` nodes and extract `text` values
   - **Subtasks**: for each entry in `fields.subtasks`, show `key — fields.summary (fields.status.name)`
4. If the API returns an error, display the HTTP status and error message clearly.

### Path B — Atlassian CLI / `acli` (`JIRA_API_TOKEN` is not set)

1. Verify that `acli` is installed by running `acli --version`. If it is not found,
   print an installation hint and stop:

   ```
   acli is not installed. Install it with: npm install -g @atlassian/acli
   Then authenticate with: acli configure
   ```

2. Run the following command and capture its output:

   ```bash
   acli jira workitem view --id PROJECT-123
   ```

3. Display the output as returned by `acli`, which includes the issue key, summary,
   status, type, priority, assignee, description, and subtasks.
4. If the command fails, display the error output clearly.

## Example commands (for reference)

```bash
# REST API path
curl -u "$JIRA_EMAIL:$JIRA_API_TOKEN" \
  -H "Accept: application/json" \
  "$JIRA_BASE_URL/rest/api/3/issue/PROJECT-123?fields=summary,description,status,issuetype,priority,assignee,parent,subtasks"

# acli path
acli jira workitem view --id PROJECT-123
```

## References

- [acli jira workitem view](https://developer.atlassian.com/cloud/acli/reference/commands/jira-workitem-view/)
- [Jira REST API v3 — Get issue](https://developer.atlassian.com/cloud/jira/platform/rest/v3/api-group-issues/#api-rest-api-3-issue-issueidorkey-get)
