---
name: gitlab-mr-create
description: Create a GitLab merge request. Use when a user asks to open an MR from the current branch or with explicit source and target branches.
version: 1.0.0
author: Roger Briggen
tags:
  - gitlab
  - merge-request
  - git
---

# GitLab MR Create

## Overview

Create a merge request in a self-hosted GitLab instance. The skill uses the
GitLab CLI when `GITLAB_TOKEN` is not set and switches to the REST API when the
API token is present.

## When to Use

- The user asks to create or open a merge request
- The user provides source and target branches, a title, or a description
- The user wants to open an MR from the current branch with sensible defaults

## Instructions

### Inputs

`$ARGUMENTS` may include these options:

- `--source <branch>`
- `--target <branch>`
- `--title <title>`
- `--description <text>`

### Step 1: Resolve defaults

Parse `$ARGUMENTS`.

- If `--source` is missing, use `git branch --show-current`.
- If `--title` is missing, use `git log -1 --format=%s`.
- If `--target` is missing, let GitLab use the repository default branch.

### Step 2: Select the access path

Check whether `GITLAB_TOKEN` is set.

- If it is not set, use the GitLab CLI path.
- If it is set, use the REST API path.

### Step 3: GitLab CLI path

Run:

```bash
glab mr create \
  --source-branch {source} \
  --target-branch {target} \
  --title "{title}" \
  --description "{description}"
```

Omit `--target-branch` when no target was supplied.

### Step 4: REST API path

Require these environment variables and fail clearly if any are missing:

- `GITLAB_HOST`
- `GITLAB_TOKEN`

Determine the project path from `git remote get-url origin`, strip the host
prefix and `.git` suffix, then URL-encode `/` as `%2F`.

Create the merge request:

```text
POST {GITLAB_HOST}/api/v4/projects/{encoded_project_path}/merge_requests
Headers: PRIVATE-TOKEN: {GITLAB_TOKEN}
         Content-Type: application/json
Body: {
  "source_branch": "{source}",
  "target_branch": "{target}",
  "title": "{title}",
  "description": "{description}"
}
```

Omit `target_branch` from the body when no explicit target was supplied.

## Output Format

On success, return the MR IID, title, URL, state, and source-to-target branch
mapping. On failure, report the HTTP status or CLI error output.

## Examples

```text
/gitlab-mr-create --source feature/login --target main --title "Add login page"
create a merge request from feature/x to main
open an MR from my current branch
```

## Notes

- Preserve explicit user inputs instead of recomputing them.
- Derive missing values from the current repository state.
- Stop with a clear error when the repository remote does not identify a GitLab project.
