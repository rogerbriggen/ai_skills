---
name: jira-read
description: Read a Jira issue and return its key fields. Use when a user asks to inspect, view, or summarize a Jira work item.
version: 1.0.0
author: Roger Briggen
tags:
  - jira
  - atlassian
  - issue-tracking
---

# Jira Read

## Overview

Read a Jira work item and present the important fields in a structured format.
The skill uses the Jira REST API when `JIRA_API_TOKEN` is available and falls
back to the Atlassian CLI (`acli`) otherwise.

## When to Use

- The user asks to read, view, show, or inspect a Jira issue
- The user needs a quick summary of status, assignee, subtasks, or description
- The user provides an issue key such as `PROJECT-123`

## Instructions

### Inputs

`$ARGUMENTS` contains the Jira issue key, for example `PROJECT-123`.

### Step 1: Read the issue key

Extract the issue key from `$ARGUMENTS` and validate that it looks like a Jira
issue identifier.

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

Call the Jira REST API v3:

```text
GET {JIRA_BASE_URL}/rest/api/3/issue/{issueKey}?fields=summary,description,status,issuetype,priority,assignee,parent,subtasks
```

Use HTTP Basic auth with `{JIRA_EMAIL}:{JIRA_API_TOKEN}`.

Parse the response and display:

- Key
- Title from `fields.summary`
- Type from `fields.issuetype.name`
- Status from `fields.status.name`
- Priority from `fields.priority.name` if present
- Assignee from `fields.assignee.displayName`, or `Unassigned`
- Parent from `fields.parent` if present
- Description converted from Atlassian Document Format to plain text by walking
  the content nodes and collecting `text` values
- Subtasks from `fields.subtasks`, including key, summary, and status

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
acli jira workitem view --id {issueKey}
```

Display the command output as returned. If the command fails, show the error
output clearly.

## Output Format

Return a readable issue summary with labeled fields. On errors, include the
HTTP status or command failure output and keep the failure reason explicit.

## Examples

```text
/jira-read PROJECT-123
show PROJECT-456
read MYTEAM-9
```

## Notes

- Prefer the REST API when the required environment variables are present.
- Preserve the issue key exactly as provided by the user.
- Do not silently ignore missing fields; mark them as unavailable when needed.

## References

- Atlassian CLI: https://developer.atlassian.com/cloud/acli/reference/commands/jira-workitem-view/
- Jira REST API: https://developer.atlassian.com/cloud/jira/platform/rest/v3/api-group-issues/#api-rest-api-3-issue-issueidorkey-get
