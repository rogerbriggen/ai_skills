Generate an AI summary of a GitLab merge request and post it as a general discussion note.

## Arguments

`$ARGUMENTS` — the MR IID (internal ID), e.g. `42`

## Instructions

1. Read the MR IID from `$ARGUMENTS`.

2. Detect which tool to use based on the `GITLAB_TOKEN` environment variable:
   - If `GITLAB_TOKEN` is **not set** → use **Option A** (GitLab CLI — no token needed, cross-platform).
   - If `GITLAB_TOKEN` **is set** → use **Option B** (REST API with curl).

---

### Option A: GitLab CLI

3. Fetch MR metadata:
   ```bash
   glab api projects/:fullpath/merge_requests/<iid>
   ```
   Extract: `title`, `description`, `author.name`, `source_branch`, `target_branch`, `created_at`, `labels`.

4. Fetch the MR changes (diff):
   ```bash
   glab api projects/:fullpath/merge_requests/<iid>/changes
   ```
   Collect all changed files (`new_path`, `old_path`, `new_file`, `deleted_file`, `renamed_file`, `diff`).

5. Generate the AI summary (same as step 6 below).

6. Post the summary as a general note:
   ```bash
   glab mr note <iid> --message "## MR Summary (AI-generated)

<summary text>"
   ```

Skip to the final display step.

---

### Option B: REST API with curl

3. Require the following environment variables (fail with a clear error message if missing):
   - `GITLAB_HOST` — base URL of your GitLab instance, e.g. `https://gitlab.example.com`
   - `GITLAB_TOKEN` — personal access token with `api` scope

4. Determine the project path from the git remote:
   ```bash
   git remote get-url origin
   ```
   Strip the host prefix and `.git` suffix. URL-encode `/` as `%2F`.

5. Fetch MR metadata:
   ```
   GET {GITLAB_HOST}/api/v4/projects/{encoded_project_path}/merge_requests/{iid}
   Headers: PRIVATE-TOKEN: {GITLAB_TOKEN}
   ```
   Extract: `title`, `description`, `author.name`, `source_branch`, `target_branch`, `created_at`, `labels`.

6. Fetch the MR changes (diff):
   ```
   GET {GITLAB_HOST}/api/v4/projects/{encoded_project_path}/merge_requests/{iid}/changes
   Headers: PRIVATE-TOKEN: {GITLAB_TOKEN}
   ```
   Collect all changed files (`new_path`, `old_path`, `new_file`, `deleted_file`, `renamed_file`, `diff`).

7. Post the summary as a general discussion note on the MR (no `position` field = general note):
   ```
   POST {GITLAB_HOST}/api/v4/projects/{encoded_project_path}/merge_requests/{iid}/discussions
   Headers: PRIVATE-TOKEN: {GITLAB_TOKEN}
            Content-Type: application/json
   Body: {
     "body": "## MR Summary (AI-generated)\n\n<summary text>"
   }
   ```

---

Generate an AI summary of the merge request that includes:
- **Purpose**: What is the goal of this MR? (inferred from title, description, and changes)
- **Changes**: A concise bullet-point list of the key changes made
- **Files changed**: Number of files modified, and the most significant ones
- **Potential concerns**: Any notable risks, missing tests, or areas that need attention
- Keep the summary concise (under ~300 words)

On success, display the discussion URL.
On error, display the HTTP status and error message.

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
