---
name: gitlab-mr-summary
description: Generate and post an AI summary for a GitLab merge request. Use when a user asks for a concise review summary or overview of an MR.
version: 1.0.0
author: Roger Briggen
tags:
  - gitlab
  - merge-request
  - review
---

# GitLab MR Summary

## Overview

Generate an AI summary of a GitLab merge request and post it as a general
comment. The skill uses the GitLab CLI when `GITLAB_TOKEN` is not set and the
REST API when the token is available.

## When to Use

- The user asks for a merge request summary
- The user wants a concise overview of purpose, major changes, and risks
- The user provides a merge request IID

## Instructions

### Inputs

`$ARGUMENTS` contains the merge request IID.

### Step 1: Read the IID

Extract the merge request IID from `$ARGUMENTS`.

### Step 2: Select the access path

Check whether `GITLAB_TOKEN` is set.

- If it is not set, use the GitLab CLI path.
- If it is set, use the REST API path.

### Step 3: Collect MR metadata

For either path, gather:

- Title
- Description
- Author name
- Source branch
- Target branch
- Created timestamp
- Labels

GitLab CLI commands:

```bash
glab api projects/:fullpath/merge_requests/{iid}
glab api projects/:fullpath/merge_requests/{iid}/changes
```

REST API endpoints:

```text
GET {GITLAB_HOST}/api/v4/projects/{encoded_project_path}/merge_requests/{iid}
GET {GITLAB_HOST}/api/v4/projects/{encoded_project_path}/merge_requests/{iid}/changes
```

### Step 4: Generate the summary

Create a concise summary under 300 words that covers:

- Purpose of the MR
- Key changes as short bullet points
- Files changed count and the most significant files
- Potential concerns such as risks, missing tests, or follow-up work

### Step 5: Post the summary

GitLab CLI:

```bash
glab mr note {iid} --message "## MR Summary (AI-generated)

{summary}"
```

REST API:

```text
POST {GITLAB_HOST}/api/v4/projects/{encoded_project_path}/merge_requests/{iid}/discussions
Headers: PRIVATE-TOKEN: {GITLAB_TOKEN}
         Content-Type: application/json
Body: {"body": "## MR Summary (AI-generated)\n\n{summary}"}
```

## Output Format

Return the discussion URL on success. On failure, include the HTTP status or
CLI error output.

## Examples

```text
/gitlab-mr-summary 42
summarize MR 42
post an AI summary for merge request 105
```

## Notes

- Keep the summary concise and review-oriented.
- Base the summary on metadata and actual changed files, not only on the title.
- Prefer concrete risks over generic review boilerplate.
