---
name: gitlab-mr-checkout
description: Check out a GitLab merge request branch locally. Use when a user asks to switch to the branch behind an MR for local review.
version: 1.0.0
author: Roger Briggen
tags:
  - gitlab
  - merge-request
  - git
---

# GitLab MR Checkout

## Overview

Check out the branch behind a GitLab merge request so it can be reviewed or
modified locally. The skill uses the GitLab CLI when `GITLAB_TOKEN` is not set
and the REST API plus git when the token is available.

## When to Use

- The user asks to check out or switch to a merge request branch
- The user wants to review an MR locally
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

### Step 3: GitLab CLI path

Run:

```bash
glab mr checkout {iid}
```

### Step 4: REST API path

Require these environment variables and fail clearly if any are missing:

- `GITLAB_HOST`
- `GITLAB_TOKEN`

Determine the project path from `git remote get-url origin`, strip the host
prefix and `.git` suffix, then URL-encode `/` as `%2F`.

Fetch MR details:

```text
GET {GITLAB_HOST}/api/v4/projects/{encoded_project_path}/merge_requests/{iid}
Headers: PRIVATE-TOKEN: {GITLAB_TOKEN}
```

Extract `title`, `source_branch`, and `sha`.

Then fetch and check out the branch:

```bash
git fetch origin {source_branch}
git checkout {source_branch}
```

If the branch does not exist locally, create it from `origin/{source_branch}`.

## Output Format

Confirm the MR title, IID, source branch, and latest commit SHA. On failure,
include the HTTP status or git and CLI error output.

## Examples

```text
/gitlab-mr-checkout 42
check out MR 42
switch to merge request 105
```

## Notes

- Do not guess the branch name; fetch it from GitLab metadata.
- Keep the local checkout logic simple and explicit.
- Report git checkout failures directly so the user can resolve local conflicts.
