---
name: gitlab-mr-diff
description: Show the diff for a GitLab merge request. Use when a user asks to inspect, review, or summarize the changes in an MR.
version: 1.0.0
author: Roger Briggen
tags:
  - gitlab
  - merge-request
  - code-review
---

# GitLab MR Diff

## Overview

Show the unified diff for an existing GitLab merge request. The skill uses the
GitLab CLI when `GITLAB_TOKEN` is not set and the REST API when the token is
available.

## When to Use

- The user asks to show or review an MR diff
- The user provides a merge request IID such as `42`
- The user needs a changed-files summary with additions and removals

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
glab mr diff {iid}
```

Display the unified diff output as returned.

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

Display a header with the title, author, branch mapping, and state.

Fetch MR changes:

```text
GET {GITLAB_HOST}/api/v4/projects/{encoded_project_path}/merge_requests/{iid}/changes
Headers: PRIVATE-TOKEN: {GITLAB_TOKEN}
```

For each entry in `changes`, display:

- The file path, or `old_path -> new_path` for renames
- Whether it is a new, deleted, or renamed file
- The unified diff content

At the end, summarize:

- Total files changed
- Total lines added, excluding `+++` headers
- Total lines removed, excluding `---` headers

## Output Format

Return a readable MR header, the per-file diffs, and the summary counts. On
failure, include the HTTP status or CLI error output.

## Examples

```text
/gitlab-mr-diff 42
show the diff for MR 42
review merge request 105
```

## Notes

- Preserve unified diff formatting so reviewers can read the patch directly.
- Count additions and removals from the diff body, not file header markers.
- Stop on API errors instead of returning partial results.
