Show the diff of an existing GitLab merge request for review.

## Arguments

`$ARGUMENTS` — the MR IID (internal ID), e.g. `42`

## Instructions

1. Read the MR IID from `$ARGUMENTS`.

2. Detect which tool to use based on the `GITLAB_TOKEN` environment variable:
   - If `GITLAB_TOKEN` is **not set** → use **Option A** (GitLab CLI — no token needed, cross-platform).
   - If `GITLAB_TOKEN` **is set** → use **Option B** (REST API with curl).

---

### Option A: GitLab CLI

```bash
glab mr diff <iid>
```

This prints the full unified diff for the MR. Skip to step 7.

---

### Option B: REST API with curl

3. Require the following environment variables (fail with a clear error message if missing):
   - `GITLAB_HOST` — base URL of your GitLab instance, e.g. `https://gitlab.example.com`
   - `GITLAB_TOKEN` — personal access token with `api` or `read_api` scope

4. Determine the project path from the git remote:
   ```bash
   git remote get-url origin
   ```
   Strip the host prefix and `.git` suffix to get `namespace/project`.
   URL-encode it by replacing `/` with `%2F`.

5. Fetch the MR details:
   ```
   GET {GITLAB_HOST}/api/v4/projects/{encoded_project_path}/merge_requests/{iid}
   Headers: PRIVATE-TOKEN: {GITLAB_TOKEN}
   ```
   Display a header with: title, author, `source_branch` → `target_branch`, state.

6. Fetch the MR changes (diff):
   ```
   GET {GITLAB_HOST}/api/v4/projects/{encoded_project_path}/merge_requests/{iid}/changes
   Headers: PRIVATE-TOKEN: {GITLAB_TOKEN}
   ```

---

7. For each file in the `changes` array:
   - Show the file path (`new_path`; if renamed, show `old_path` → `new_path`)
   - Indicate if it was a `new_file`, `deleted_file`, or `renamed_file`
   - Display the `diff` content in unified diff format

8. Show a summary at the end:
   - Total files changed
   - Total lines added (count `+` lines in diffs, excluding `+++` headers)
   - Total lines removed (count `-` lines in diffs, excluding `---` headers)

9. On error, display the HTTP status and error message.

## Example curl commands (for reference)

```bash
PROJECT_PATH=$(git remote get-url origin | sed 's|.*gitlab[^/]*/||; s|\.git$||' | sed 's|/|%2F|g')

# Fetch MR details
curl -s --header "PRIVATE-TOKEN: $GITLAB_TOKEN" \
  "$GITLAB_HOST/api/v4/projects/$PROJECT_PATH/merge_requests/42"

# Fetch MR diff/changes
curl -s --header "PRIVATE-TOKEN: $GITLAB_TOKEN" \
  "$GITLAB_HOST/api/v4/projects/$PROJECT_PATH/merge_requests/42/changes"
```
