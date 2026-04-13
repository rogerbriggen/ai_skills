Transition a Jira issue to a new state.

Automatically selects the access method at runtime:
- If `JIRA_API_TOKEN` is set in the environment → uses the **Jira Cloud REST API v3**
- If `JIRA_API_TOKEN` is not set → uses the **Jira CLI** (`jira` from [ankitpokhrel/jira-cli](https://github.com/ankitpokhrel/jira-cli))

## Arguments

`$ARGUMENTS` — the Jira issue key and the desired new state, e.g. `PROJECT-123 In Progress`

## Instructions

1. Parse `$ARGUMENTS`: the first token is the issue key, the remainder is the desired state name.

2. **Choose the access method** based on environment variables:
   - If `JIRA_API_TOKEN` is present in the environment → follow the [REST API path](#rest-api-path).
   - If `JIRA_API_TOKEN` is not set → follow the [Jira CLI path](#jira-cli-path).

---

## REST API path

Requires the following environment variables (fail with a clear message if any are missing):
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

### Example curl commands (REST API)

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

---

## Jira CLI path

Requires the `jira` CLI ([ankitpokhrel/jira-cli](https://github.com/ankitpokhrel/jira-cli)) to be installed and authenticated (run `jira init` once to configure your instance URL and credentials).

3. Verify the CLI is available:
   ```bash
   jira version
   ```
   If the command is not found, print a clear error message explaining that the `jira` CLI must be installed (see [Setup](#setup-jira-cli)) and stop.

4. Transition the issue to the desired state:
   ```bash
   jira issue move PROJECT-123 "In Progress"
   ```

5. If the command succeeds (exit code 0), confirm: `Successfully transitioned {issueKey} to "{state}"`.

6. If the command exits with a non-zero status, display the error message and stop.
   - If the error mentions that the state is not available, suggest running `jira issue move PROJECT-123` without a state argument — the CLI will present an interactive list of valid transitions.

### Example CLI commands

```bash
# Transition to a specific state
jira issue move PROJECT-123 "In Progress"

# List available transitions interactively (no state argument)
jira issue move PROJECT-123
```
