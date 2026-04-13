# Jira Update State

Transition a Jira work item to a new state.

Uses the Jira REST API when `JIRA_API_TOKEN` is set; falls back to the official
Atlassian CLI (`acli`) when it is not.

## Arguments

`$ARGUMENTS` — the Jira issue key and the desired new state, e.g. `PROJECT-123 In Progress`

## Instructions

1. Parse `$ARGUMENTS`: the first token is the issue key, the remainder is the desired state name.
2. Check whether `JIRA_API_TOKEN` is set in the environment.

### Path A — REST API (`JIRA_API_TOKEN` is set)

1. Require the following environment variables (fail with a clear message if any are missing):
   - `JIRA_BASE_URL` — e.g. `https://yourcompany.atlassian.net`
   - `JIRA_EMAIL` — your Atlassian account email
   - `JIRA_API_TOKEN` — your Atlassian API token
2. Fetch the available transitions for the issue:
   ```
   GET {JIRA_BASE_URL}/rest/api/3/issue/{issueKey}/transitions
   ```
   Use HTTP Basic auth with `{JIRA_EMAIL}:{JIRA_API_TOKEN}`.
3. Find the transition whose `name` matches the desired state (case-insensitive).
   - If no match is found, list all available transition names and stop.
4. Apply the matching transition:
   ```
   POST {JIRA_BASE_URL}/rest/api/3/issue/{issueKey}/transitions
   Content-Type: application/json
   Body: {"transition": {"id": "<transitionId>"}}
   ```
5. Confirm success: `Successfully transitioned {issueKey} to "{state}"`.
6. If the API returns an error, display the HTTP status and error message clearly.

### Path B — Atlassian CLI / `acli` (`JIRA_API_TOKEN` is not set)

1. Verify that `acli` is installed by running `acli --version`. If it is not found,
   print an installation hint and stop:

   ```
   acli is not installed. Install it with: npm install -g @atlassian/acli
   Then authenticate with: acli configure
   ```

2. Run the following command to transition the issue:

   ```bash
   acli jira workitem transition --id PROJECT-123 --state "In Progress"
   ```

3. Confirm success by displaying the output returned by `acli`.
4. If the command fails (e.g. unknown state), display the error output clearly.
   - If the error indicates an invalid state, suggest running `/jira-read PROJECT-123`
     to see the current status, then check valid transitions via the Jira UI.

## Example commands (for reference)

```bash
# REST API path — list transitions then apply one
curl -u "$JIRA_EMAIL:$JIRA_API_TOKEN" \
  -H "Accept: application/json" \
  "$JIRA_BASE_URL/rest/api/3/issue/PROJECT-123/transitions"

curl -u "$JIRA_EMAIL:$JIRA_API_TOKEN" \
  -X POST \
  -H "Content-Type: application/json" \
  -d '{"transition": {"id": "TRANSITION_ID"}}' \
  "$JIRA_BASE_URL/rest/api/3/issue/PROJECT-123/transitions"

# acli path
acli jira workitem transition --id PROJECT-123 --state "In Progress"
```

## References

- [acli jira workitem transition](https://developer.atlassian.com/cloud/acli/reference/commands/jira-workitem-transition/)
- [Jira REST API v3 — Transition issue](https://developer.atlassian.com/cloud/jira/platform/rest/v3/api-group-issues/#api-rest-api-3-issue-issueidorkey-transitions-post)
