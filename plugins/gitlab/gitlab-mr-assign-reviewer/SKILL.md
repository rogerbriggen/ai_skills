---
name: gitlab-mr-assign-reviewer
description: Add the current user as a reviewer on a GitLab merge request. Use when a user asks to assign themselves to an MR review.
version: 1.0.0
author: Roger Briggen
tags:
  - gitlab
  - merge-request
  - review
---

# GitLab MR Assign Reviewer

## Overview

Add the current user as a reviewer on a GitLab merge request without removing
existing reviewers. The skill uses the GitLab CLI when `GITLAB_TOKEN` is not
set and the REST API when the token is available.

## When to Use

- The user asks to add themselves as a reviewer
- The user provides a merge request IID
- The user wants a review assignment without changing the rest of the reviewer list

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
glab mr update {iid} --reviewer @me
```

### Step 4: REST API path

Require these environment variables and fail clearly if any are missing:

- `GITLAB_HOST`
- `GITLAB_TOKEN`

Determine the project path from `git remote get-url origin`, strip the host
prefix and `.git` suffix, then URL-encode `/` as `%2F`.

Fetch the current user:

```text
GET {GITLAB_HOST}/api/v4/user
Headers: PRIVATE-TOKEN: {GITLAB_TOKEN}
```

Extract the user `id` and `username`.

Fetch the merge request:

```text
GET {GITLAB_HOST}/api/v4/projects/{encoded_project_path}/merge_requests/{iid}
Headers: PRIVATE-TOKEN: {GITLAB_TOKEN}
```

Extract the current reviewers. If the authenticated user is already a reviewer,
stop and report that no change is needed.

Otherwise, append the user ID to the existing reviewer IDs and update the merge
request:

```text
PUT {GITLAB_HOST}/api/v4/projects/{encoded_project_path}/merge_requests/{iid}
Headers: PRIVATE-TOKEN: {GITLAB_TOKEN}
         Content-Type: application/json
Body: {"reviewer_ids": [{id1}, {id2}, ...]}
```

## Output Format

Return a short confirmation such as `Successfully added @username as reviewer on
MR !42`. On failure, include the HTTP status or CLI error output.

## Examples

```text
/gitlab-mr-assign-reviewer 42
add me as reviewer on MR 42
assign myself to merge request 105
```

## Notes

- Preserve the existing reviewer list.
- Avoid duplicate reviewer IDs.
- Stop with a no-op message when the user is already assigned.
