Create a review note on a specific line in a GitLab merge request diff.

## Arguments

`$ARGUMENTS` — space-separated values in this order:
1. MR IID — e.g. `42`
2. File path — e.g. `src/main.go`
3. Line number — new-file line number, e.g. `15`
4. Note text — the review comment (quote if it contains spaces), e.g. `"This should be refactored."`

Example: `42 src/main.go 15 "This logic should be extracted into a helper function."`

## Instructions

1. Parse `$ARGUMENTS`:
   - First token: MR IID
   - Second token: file path
   - Third token: line number (integer — refers to the line in the new version of the file)
   - Remaining tokens: the note text (joined back into a single string)

2. Determine which method to use:
   - If `GITLAB_TOKEN` environment variable is **not** set → use **Option A: GitLab CLI** (`glab api`).
   - If `GITLAB_TOKEN` environment variable **is** set → use **Option B: REST API with curl**.

3. Determine the project path from the git remote:
   ```bash
   git remote get-url origin
   ```
   Strip the host prefix and `.git` suffix. URL-encode `/` as `%2F`.

---

### Option A: GitLab CLI

4. Fetch the MR diff versions to get the diff refs:
   ```bash
   glab api "projects/<encoded_project_path>/merge_requests/<iid>/versions"
   ```
   Use the first entry (most recent version) and extract:
   - `base_commit_sha`
   - `start_commit_sha`
   - `head_commit_sha`

5. Create the inline discussion note:
   ```bash
   echo '{
     "body": "<note text>",
     "position": {
       "base_sha": "<base_commit_sha>",
       "start_sha": "<start_commit_sha>",
       "head_sha": "<head_commit_sha>",
       "position_type": "text",
       "new_path": "<file_path>",
       "old_path": "<file_path>",
       "new_line": <line_number>
     }
   }' | glab api "projects/<encoded_project_path>/merge_requests/<iid>/discussions" \
     --method POST \
     --input -
   ```

Skip to step 9.

---

### Option B: REST API with curl

6. Require the following environment variables (fail with a clear error message if missing):
   - `GITLAB_HOST` — base URL of your GitLab instance, e.g. `https://gitlab.example.com`
   - `GITLAB_TOKEN` — personal access token with `api` scope

7. Fetch the MR diff versions to get the diff refs:
   ```
   GET {GITLAB_HOST}/api/v4/projects/{encoded_project_path}/merge_requests/{iid}/versions
   Headers: PRIVATE-TOKEN: {GITLAB_TOKEN}
   ```
   Use the first entry (most recent version) and extract:
   - `base_commit_sha`
   - `start_commit_sha`
   - `head_commit_sha`

8. Create the inline discussion note:
   ```
   POST {GITLAB_HOST}/api/v4/projects/{encoded_project_path}/merge_requests/{iid}/discussions
   Headers: PRIVATE-TOKEN: {GITLAB_TOKEN}
            Content-Type: application/json
   Body: {
     "body": "<note text>",
     "position": {
       "base_sha": "<base_commit_sha>",
       "start_sha": "<start_commit_sha>",
       "head_sha": "<head_commit_sha>",
       "position_type": "text",
       "new_path": "<file_path>",
       "old_path": "<file_path>",
       "new_line": <line_number>
     }
   }
   ```
   - Use `new_line` for comments on added or unchanged lines in the new file.
   - For comments on deleted lines (only present in the old file), use `old_line` instead of `new_line`.

---

9. On success (HTTP 201), display the discussion ID and the note URL (`notes[0].id` in the response).
10. On error, display the HTTP status and error message.
    - If the error indicates the line is not part of the diff, suggest using a general MR note (`gitlab-mr-summary`) instead.

## Example curl commands (for reference, Option B)

```bash
PROJECT_PATH=$(git remote get-url origin | sed 's|.*gitlab[^/]*/||; s|\.git$||' | sed 's|/|%2F|g')

# Step 1: Get diff versions (SHAs)
curl -s --header "PRIVATE-TOKEN: $GITLAB_TOKEN" \
  "$GITLAB_HOST/api/v4/projects/$PROJECT_PATH/merge_requests/42/versions"

# Step 2: Create inline discussion note (replace SHAs with values from step 1)
curl -s --request POST \
  --header "PRIVATE-TOKEN: $GITLAB_TOKEN" \
  --header "Content-Type: application/json" \
  --data '{
    "body": "This logic should be extracted into a helper function.",
    "position": {
      "base_sha": "abc123",
      "start_sha": "def456",
      "head_sha": "ghi789",
      "position_type": "text",
      "new_path": "src/main.go",
      "old_path": "src/main.go",
      "new_line": 15
    }
  }' \
  "$GITLAB_HOST/api/v4/projects/$PROJECT_PATH/merge_requests/42/discussions"
```
