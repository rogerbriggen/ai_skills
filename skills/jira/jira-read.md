Use the Jira REST API to read a work item and display its details.

## Arguments

`$ARGUMENTS` — the Jira issue key, e.g. `PROJECT-123`

## Instructions

1. Read the issue key from `$ARGUMENTS`.
2. Require the following environment variables to be set (fail with a clear message if missing):
   - `JIRA_BASE_URL` — e.g. `https://yourcompany.atlassian.net`
   - `JIRA_EMAIL` — your Atlassian account email
   - `JIRA_API_TOKEN` — your Atlassian API token
3. Call the Jira REST API v3 to fetch the issue:
   ```
   GET {JIRA_BASE_URL}/rest/api/3/issue/{issueKey}?fields=summary,description,status,issuetype,priority,assignee,parent,subtasks
   ```
   Use HTTP Basic auth with `{JIRA_EMAIL}:{JIRA_API_TOKEN}` (Base64-encoded).
4. Parse the JSON response and display the following in a readable format:
   - **Key**: issue key (e.g. `PROJECT-123`)
   - **Title**: `fields.summary`
   - **Type**: `fields.issuetype.name`
   - **Status**: `fields.status.name`
   - **Priority**: `fields.priority.name` (if present)
   - **Assignee**: `fields.assignee.displayName` (or "Unassigned" if null)
   - **Parent**: `fields.parent.key — fields.parent.fields.summary` (if present)
   - **Description**: convert ADF (Atlassian Document Format) `fields.description` to plain text — walk the `content` nodes and extract `text` values
   - **Subtasks**: for each entry in `fields.subtasks`, show `key — fields.summary (fields.status.name)`
5. If the API returns an error, display the HTTP status and error message clearly.

## Example curl command (for reference)

```bash
curl -u "$JIRA_EMAIL:$JIRA_API_TOKEN" \
  -H "Accept: application/json" \
  "$JIRA_BASE_URL/rest/api/3/issue/PROJECT-123?fields=summary,description,status,issuetype,priority,assignee,parent,subtasks"
```
