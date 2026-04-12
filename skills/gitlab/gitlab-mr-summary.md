Generate an AI summary of a GitLab merge request and post it as a general discussion note.

## Arguments

`$ARGUMENTS` — the MR IID (internal ID), e.g. `42`

## Instructions

1. Read the MR IID from `$ARGUMENTS`.

2. Require the following environment variables (fail with a clear error message if missing):
   - `GITLAB_HOST` — base URL of your GitLab instance, e.g. `https://gitlab.example.com`
   - `GITLAB_TOKEN` — personal access token with `api` scope

3. Determine the project path from the git remote:
   ```bash
   git remote get-url origin
   ```
   Strip the host prefix and `.git` suffix. URL-encode `/` as `%2F`.

4. Fetch MR metadata:
   ```
   GET {GITLAB_HOST}/api/v4/projects/{encoded_project_path}/merge_requests/{iid}
   Headers: PRIVATE-TOKEN: {GITLAB_TOKEN}
   ```
   Extract: `title`, `description`, `author.name`, `source_branch`, `target_branch`, `created_at`, `labels`.

5. Fetch the MR changes (diff):
   ```
   GET {GITLAB_HOST}/api/v4/projects/{encoded_project_path}/merge_requests/{iid}/changes
   Headers: PRIVATE-TOKEN: {GITLAB_TOKEN}
   ```
   Collect all changed files (`new_path`, `old_path`, `new_file`, `deleted_file`, `renamed_file`, `diff`).

6. Generate an AI summary of the merge request that includes:
   - **Purpose**: What is the goal of this MR? (inferred from title, description, and changes)
   - **Changes**: A concise bullet-point list of the key changes made
   - **Files changed**: Number of files modified, and the most significant ones
   - **Potential concerns**: Any notable risks, missing tests, or areas that need attention
   - Keep the summary concise (under ~300 words)

7. Post the summary as a general discussion note on the MR (no `position` field = general note):
   ```
   POST {GITLAB_HOST}/api/v4/projects/{encoded_project_path}/merge_requests/{iid}/discussions
   Headers: PRIVATE-TOKEN: {GITLAB_TOKEN}
            Content-Type: application/json
   Body: {
     "body": "## MR Summary (AI-generated)\n\n<summary text>"
   }
   ```

8. On success (HTTP 201), display the discussion URL.
9. On error, display the HTTP status and error message.

## Example curl commands (for reference)

```bash
PROJECT_PATH=$(git remote get-url origin | sed 's|.*gitlab[^/]*/||; s|\.git$||' | sed 's|/|%2F|g')

# Fetch MR details
curl -s --header "PRIVATE-TOKEN: $GITLAB_TOKEN" \
  "$GITLAB_HOST/api/v4/projects/$PROJECT_PATH/merge_requests/42"

# Fetch MR changes
curl -s --header "PRIVATE-TOKEN: $GITLAB_TOKEN" \
  "$GITLAB_HOST/api/v4/projects/$PROJECT_PATH/merge_requests/42/changes"

# Post summary as a general discussion note
curl -s --request POST \
  --header "PRIVATE-TOKEN: $GITLAB_TOKEN" \
  --header "Content-Type: application/json" \
  --data '{"body": "## MR Summary (AI-generated)\n\n..."}' \
  "$GITLAB_HOST/api/v4/projects/$PROJECT_PATH/merge_requests/42/discussions"
```
