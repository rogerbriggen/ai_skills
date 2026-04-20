---
name: jira-update-state
description: Transition a Jira issue to a new workflow state. Use when a user asks to move, update, or transition a Jira work item.
version: 1.0.0
author: Roger Briggen
tags:
  - jira
  - atlassian
  - workflow
---

# Jira Update State

## Overview

Transition a Jira work item to a new workflow state. The skill uses the Jira
REST API when `JIRA_API_TOKEN` is available and falls back to the Atlassian CLI
(`acli`) otherwise.

## When to Use

- The user asks to move an issue to a new state
- The user wants to transition a ticket to `In Progress`, `Done`, or another
  workflow state
- The user provides an issue key together with a target state

## Instructions

### Inputs

`$ARGUMENTS` contains the Jira issue key followed by the target state, for
example `PROJECT-123 In Progress`.

### Step 1: Parse the input

Treat the first token as the issue key and the remaining text as the desired
state name.

### Step 2: Select the access path

Check whether `JIRA_API_TOKEN` is set.

- If it is set, use the REST API path.
- If it is not set, use the Atlassian CLI path.

### Step 3: REST API path

Require the following environment variables and fail with a clear message if any
are missing:

- `JIRA_BASE_URL`
- `JIRA_EMAIL`
- `JIRA_API_TOKEN`

Fetch the available transitions:

```text
GET {JIRA_BASE_URL}/rest/api/3/issue/{issueKey}/transitions
```

Use HTTP Basic auth with `{JIRA_EMAIL}:{JIRA_API_TOKEN}`.

Find the transition whose `name` matches the requested state case-insensitively.
If no match exists, list the available transition names and stop.

Apply the matching transition:

```text
POST {JIRA_BASE_URL}/rest/api/3/issue/{issueKey}/transitions
Content-Type: application/json
Body: {"transition": {"id": "<transitionId>"}}
```

Confirm success with `Successfully transitioned {issueKey} to "{state}"`.
If the API returns an error, report the HTTP status and response message.

### Step 4: Atlassian CLI path

Verify that `acli` is installed with `acli --version`.

If it is missing, stop and print:

```text
acli is not installed. Install it with: npm install -g @atlassian/acli
Then authenticate with: acli configure
```

Run:

```bash
acli jira workitem transition --id {issueKey} --state "{state}"
```

Display the command output on success. If the command fails, show the error and
suggest checking valid transitions in Jira when the state name is invalid.

## Output Format

Return a short success confirmation or a clear failure message. When the state
cannot be applied, include the available transitions or the CLI error output.

## Examples

```text
/jira-update-state PROJECT-123 In Progress
move PROJECT-456 to Done
transition MYTEAM-9 to Ready for Test
```

## Notes

- Match transition names case-insensitively, but preserve the user-facing state
  name in confirmations.
- Stop instead of guessing when multiple transitions are ambiguous.
- Keep the output concise and action-oriented.

## References

- Atlassian CLI: https://developer.atlassian.com/cloud/acli/reference/commands/jira-workitem-transition/
- Jira REST API: https://developer.atlassian.com/cloud/jira/platform/rest/v3/api-group-issues/#api-rest-api-3-issue-issueidorkey-transitions-post
