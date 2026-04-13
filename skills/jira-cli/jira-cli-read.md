Read a Jira issue and display its details.

Automatically selects the access method at runtime:
- If `JIRA_API_TOKEN` is set in the environment → uses the **Jira Cloud REST API v3**
- If `JIRA_API_TOKEN` is not set → uses the **Jira CLI** (`jira` from [ankitpokhrel/jira-cli](https://github.com/ankitpokhrel/jira-cli))

## Arguments

`$ARGUMENTS` — the Jira issue key, e.g. `PROJECT-123`

## Instructions

1. Read the issue key from `$ARGUMENTS`.

2. **Choose the access method** based on environment variables:
   - If `JIRA_API_TOKEN` is present in the environment → follow the [REST API path](#rest-api-path).
   - If `JIRA_API_TOKEN` is not set → follow the [Jira CLI path](#jira-cli-path).

---

## REST API path

Requires the following environment variables (fail with a clear message if any are missing):
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

### Example curl command (REST API)

```bash
curl -u "$JIRA_EMAIL:$JIRA_API_TOKEN" \
  -H "Accept: application/json" \
  "$JIRA_BASE_URL/rest/api/3/issue/PROJECT-123?fields=summary,description,status,issuetype,priority,assignee,parent,subtasks"
```

---

## Jira CLI path

Requires the `jira` CLI ([ankitpokhrel/jira-cli](https://github.com/ankitpokhrel/jira-cli)) to be installed and authenticated (run `jira init` once to configure your instance URL and credentials).

3. Verify the CLI is available:
   ```bash
   jira version
   ```
   If the command is not found, print a clear error message explaining that the `jira` CLI must be installed (see [Setup](#setup-jira-cli)) and stop.

4. Fetch and display the issue:
   ```bash
   jira issue view PROJECT-123
   ```

5. The CLI outputs a human-readable summary including key, title, type, status, priority, assignee, description, and subtasks. Display the output as-is.

6. If the command exits with a non-zero status, display the error message and stop.

### Example CLI command

```bash
jira issue view PROJECT-123
# Add --plain flag to suppress interactive pager if needed:
jira issue view PROJECT-123 --plain
```
