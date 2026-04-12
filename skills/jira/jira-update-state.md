Use the Jira REST API to transition a work item to a new state.

## Arguments

`$ARGUMENTS` — the Jira issue key and the desired new state, e.g. `PROJECT-123 In Progress`

## Instructions

1. Parse `$ARGUMENTS`: the first token is the issue key, the remainder is the desired state name.
2. Require the following environment variables to be set (fail with a clear message if missing):
   - `JIRA_BASE_URL` — e.g. `https://yourcompany.atlassian.net`
   - `JIRA_EMAIL` — your Atlassian account email
   - `JIRA_API_TOKEN` — your Atlassian API token
3. Fetch the available transitions for the issue:
   ```
   GET {JIRA_BASE_URL}/rest/api/3/issue/{issueKey}/transitions
   ```
   Use HTTP Basic auth with `{JIRA_EMAIL}:{JIRA_API_TOKEN}`.
4. Find the transition whose `name` matches the desired state (case-insensitive).
   - If no match is found, list all available transition names and stop.
5. Apply the matching transition:
   ```
   POST {JIRA_BASE_URL}/rest/api/3/issue/{issueKey}/transitions
   Content-Type: application/json
   Body: {"transition": {"id": "<transitionId>"}}
   ```
6. Confirm success by displaying: `Successfully transitioned {issueKey} to "{state}"`.
7. If the API returns an error, display the HTTP status and error message clearly.

## Example curl commands (for reference)

```bash
# List available transitions
curl -u "$JIRA_EMAIL:$JIRA_API_TOKEN" \
  -H "Accept: application/json" \
  "$JIRA_BASE_URL/rest/api/3/issue/PROJECT-123/transitions"

# Apply a transition (replace TRANSITION_ID with the actual ID)
curl -u "$JIRA_EMAIL:$JIRA_API_TOKEN" \
  -X POST \
  -H "Content-Type: application/json" \
  -d '{"transition": {"id": "TRANSITION_ID"}}' \
  "$JIRA_BASE_URL/rest/api/3/issue/PROJECT-123/transitions"
```
