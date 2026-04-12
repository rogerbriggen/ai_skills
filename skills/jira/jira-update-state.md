---
description: Update the state/status of a Jira work item. Usage: /jira-update-state <ISSUE-KEY> <NEW-STATE>
---

Update the status of a Jira work item using the Jira Cloud REST API.

Arguments: **$ARGUMENTS**

Expected format: `<ISSUE-KEY> <NEW-STATE>`

Examples:
- `PROJ-123 Done`
- `PROJ-123 "In Progress"`
- `PROJ-123 "To Do"`

Parse `$ARGUMENTS` to extract:
- `ISSUE_KEY` — the first token (e.g., `PROJ-123`)
- `NEW_STATE` — everything after the first token, with surrounding quotes stripped (e.g., `In Progress`)

## Required environment variables

- `JIRA_BASE_URL` — Your Atlassian instance URL, e.g. `https://yourcompany.atlassian.net`
- `JIRA_EMAIL` — Your Atlassian account email address
- `JIRA_API_TOKEN` — Your Atlassian API token (generate at https://id.atlassian.com/manage-profile/security/api-tokens)

If any of these are not set, stop and inform the user which variables are missing.

## Step 1 — Fetch available transitions

Run this command using the Bash tool (replace `ISSUE_KEY` with the parsed value):

```bash
curl -s -w "\n%{http_code}" \
  -u "$JIRA_EMAIL:$JIRA_API_TOKEN" \
  -H "Accept: application/json" \
  "$JIRA_BASE_URL/rest/api/3/issue/ISSUE_KEY/transitions"
```

If the HTTP status code is not `200`, display an error:
- `401` — Authentication failed. Check `JIRA_EMAIL` and `JIRA_API_TOKEN`.
- `403` — Permission denied.
- `404` — Issue not found. Check the issue key and `JIRA_BASE_URL`.
- Other — Show the status code and error details.

## Step 2 — Find matching transition

From the `transitions` array in the response, find the entry whose `name` matches `NEW_STATE` (case-insensitive).

If no matching transition is found:
- List all available transition names to the user.
- Stop and ask the user to choose one of the available transitions.

## Step 3 — Apply the transition

Using the matched transition `id`, run:

```bash
curl -s -w "\n%{http_code}" \
  -u "$JIRA_EMAIL:$JIRA_API_TOKEN" \
  -H "Accept: application/json" \
  -H "Content-Type: application/json" \
  -X POST \
  -d "{\"transition\":{\"id\":\"TRANSITION_ID\"}}" \
  "$JIRA_BASE_URL/rest/api/3/issue/ISSUE_KEY/transitions"
```

Replace `TRANSITION_ID` with the matched transition id and `ISSUE_KEY` with the parsed issue key.

## Step 4 — Confirm result

- A `204 No Content` response means the transition was applied successfully.
- Display a success message: **ISSUE_KEY** has been moved to **NEW_STATE**.
- For any other status code, display the error details.

## Notes

- Transitions in Jira are workflow-specific. The available transitions depend on the issue's current status and the project's workflow configuration.
- To see the current state of an issue and its subtasks, use `/jira-read ISSUE_KEY` first.
