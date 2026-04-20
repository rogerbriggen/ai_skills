---
name: gitlab-mr-list-open
description: List all open GitLab merge requests for the current project. Use when a user asks to show, list, or review open merge requests in the repository.
version: 1.0.0
author: Roger Briggen
tags:
  - gitlab
  - merge-request
  - review
---

# GitLab MR List Open

## Overview

List all open merge requests for the current GitLab project. The skill uses the
GitLab CLI when `GITLAB_TOKEN` is not set and the REST API when the token is
available.

## When to Use

- The user asks to list open merge requests
- The user wants to see the current review queue for the repository
- The user wants merge request IDs, titles, authors, or branch mappings for the current project

## Instructions

### Inputs

`$ARGUMENTS` is optional. Ignore extra arguments unless the user explicitly asks
for filtering that this skill supports in the future.

### Step 1: Select the access path

Check whether `GITLAB_TOKEN` is set.

- If it is not set, use the GitLab CLI path.
- If it is set, use the REST API path.

### Step 2: GitLab CLI path

Run:

```bash
glab mr list -F json -o updated_at -S desc
```

Display the command output as returned. If the command fails, show the CLI
error output clearly.

### Step 3: REST API path

Require these environment variables and fail clearly if any are missing:

- `GITLAB_HOST`
- `GITLAB_TOKEN`

Determine the project path from `git remote get-url origin`, strip the host
prefix and `.git` suffix, then URL-encode `/` as `%2F`.

Fetch open merge requests for the current project:

```text
GET {GITLAB_HOST}/api/v4/projects/{encoded_project_path}/merge_requests?state=opened&order_by=updated_at&sort=desc
Headers: PRIVATE-TOKEN: {GITLAB_TOKEN}
```

For each merge request, display:

- IID
- Title
- Author name or username
- State
- Source branch -> target branch
- Draft status if applicable
- Updated timestamp
- Web URL

If the response is empty, report that there are no open merge requests.
If the API returns an error, report the HTTP status and response message.

## Output Format

Return a concise list ordered by most recently updated merge request first. Each
entry should include the MR IID, title, author, branch mapping, and URL. On
failure, include the HTTP status or CLI error output.

## Examples

```text
/gitlab-mr-list-open
list open merge requests
show open MRs for this project
```

## Notes

- This skill operates on the current repository only.
- Prefer the REST API when `GITLAB_TOKEN` is available.
- `read_api` scope is sufficient for the API path.
