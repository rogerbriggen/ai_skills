---
name: gitlab-mr-line-note
description: Add an inline discussion note to a GitLab merge request diff. Use when a user points at a file, line, and comment text.
version: 1.0.0
author: Roger Briggen
tags:
  - gitlab
  - merge-request
  - code-review
---

# GitLab MR Line Note

## Overview

Create an inline review discussion on a specific line of a GitLab merge request
diff. The skill uses the GitLab CLI when `GITLAB_TOKEN` is not set and the REST
API when the token is available.

## When to Use

- The user wants to comment on a specific diff line in an MR
- The user provides an MR IID, a file path, a line number, and note text
- The user wants an inline note instead of a general discussion comment

## Instructions

### Inputs

`$ARGUMENTS` is expected in this order:

1. MR IID
2. File path
3. Line number in the new file version
4. Note text

### Step 1: Parse the input

Treat the first token as the MR IID, the second as the file path, the third as
an integer line number, and the remaining text as the review note.

### Step 2: Select the access path

Check whether `GITLAB_TOKEN` is set.

- If it is not set, use the GitLab CLI path.
- If it is set, use the REST API path.

### Step 3: GitLab CLI path

Fetch the diff version SHAs:

```bash
glab api projects/:fullpath/merge_requests/{iid}/versions
```

Use the most recent entry to extract `base_commit_sha`, `start_commit_sha`, and
`head_commit_sha`.

Create the inline discussion:

```bash
glab api projects/:fullpath/merge_requests/{iid}/discussions \
  --method POST \
  -f body="{note text}" \
  -f "position[base_sha]={base_commit_sha}" \
  -f "position[start_sha]={start_commit_sha}" \
  -f "position[head_sha]={head_commit_sha}" \
  -f "position[position_type]=text" \
  -f "position[new_path]={file_path}" \
  -f "position[old_path]={file_path}" \
  -f "position[new_line]={line_number}"
```

### Step 4: REST API path

Require these environment variables and fail clearly if any are missing:

- `GITLAB_HOST`
- `GITLAB_TOKEN`

Determine the project path from `git remote get-url origin`, strip the host
prefix and `.git` suffix, then URL-encode `/` as `%2F`.

Fetch the merge request versions:

```text
GET {GITLAB_HOST}/api/v4/projects/{encoded_project_path}/merge_requests/{iid}/versions
Headers: PRIVATE-TOKEN: {GITLAB_TOKEN}
```

Use the most recent version to extract `base_commit_sha`, `start_commit_sha`,
and `head_commit_sha`.

Create the inline discussion:

```text
POST {GITLAB_HOST}/api/v4/projects/{encoded_project_path}/merge_requests/{iid}/discussions
Headers: PRIVATE-TOKEN: {GITLAB_TOKEN}
         Content-Type: application/json
Body: {
  "body": "{note text}",
  "position": {
    "base_sha": "{base_commit_sha}",
    "start_sha": "{start_commit_sha}",
    "head_sha": "{head_commit_sha}",
    "position_type": "text",
    "new_path": "{file_path}",
    "old_path": "{file_path}",
    "new_line": {line_number}
  }
}
```

Use `old_line` instead of `new_line` only when the comment targets a deleted
line in the old file.

## Output Format

Return the discussion ID and note URL on success. On failure, report the HTTP
status or CLI error output and mention when the target line is not part of the
diff.

## Examples

```text
/gitlab-mr-line-note 42 src/auth.go 87 "Consider using a constant here."
add a note on MR 42 src/main.go 15 "Extract this helper"
```

## Notes

- Inline notes require the exact diff version SHAs.
- Use the current file path for both `new_path` and `old_path` unless the file
  was renamed and the old path is known separately.
- Suggest a general MR note when the target line cannot be anchored to the diff.
