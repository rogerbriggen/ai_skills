---
description: Read a Jira work item — displays title, description, state, and subtasks with their state. Usage: /jira-read <ISSUE-KEY>
---

Read the Jira work item **$ARGUMENTS** using the Jira Cloud REST API and display its details in a clear, readable format.

## Required environment variables

- `JIRA_BASE_URL` — Your Atlassian instance URL, e.g. `https://yourcompany.atlassian.net`
- `JIRA_EMAIL` — Your Atlassian account email address
- `JIRA_API_TOKEN` — Your Atlassian API token (generate at https://id.atlassian.com/manage-profile/security/api-tokens)

If any of these are not set, stop and inform the user which variables are missing.

## Step 1 — Fetch the issue

Run this command using the Bash tool:

```bash
curl -s -w "\n%{http_code}" \
  -u "$JIRA_EMAIL:$JIRA_API_TOKEN" \
  -H "Accept: application/json" \
  "$JIRA_BASE_URL/rest/api/3/issue/$ARGUMENTS?fields=summary,description,status,subtasks,issuetype,assignee,priority,parent"
```

The output contains the JSON response body followed by the HTTP status code on the last line.

## Step 2 — Handle errors

If the HTTP status code is not `200`, display a helpful error:

- `401` — Authentication failed. Check `JIRA_EMAIL` and `JIRA_API_TOKEN`.
- `403` — Permission denied. Your account may not have access to this issue.
- `404` — Issue not found. Check the issue key (`$ARGUMENTS`) and `JIRA_BASE_URL`.
- Other — Show the status code and any error message from the response body.

## Step 3 — Display the work item

Parse the JSON and display the following information:

---

### [issue key] — [fields.summary]

| Field | Value |
|---|---|
| **Type** | `fields.issuetype.name` |
| **Status** | `fields.status.name` |
| **Priority** | `fields.priority.name` (omit row if null) |
| **Assignee** | `fields.assignee.displayName` (show "Unassigned" if null) |
| **Parent** | `fields.parent.key — fields.parent.fields.summary` (omit row if null) |

**Description:**

Convert `fields.description` from Atlassian Document Format (ADF) to readable plain text:
- ADF is a JSON structure with `type: "doc"` and a `content` array.
- Recursively extract all `text` nodes and join them, preserving paragraph breaks.
- If `fields.description` is null or empty, display *(No description)*.

**Subtasks:**

For each entry in `fields.subtasks`, display one line:
```
- [subtask.key] subtask.fields.summary — Status: subtask.fields.status.name
```

If `fields.subtasks` is empty or null, display *(No subtasks)*.

---

## Notes

- The Jira Cloud REST API v3 uses Atlassian Document Format (ADF) for rich text fields. When displaying description content, convert ADF to plain text rather than showing raw JSON.
- Issue keys are case-insensitive in the API but conventionally uppercase (e.g., `PROJ-123`).
