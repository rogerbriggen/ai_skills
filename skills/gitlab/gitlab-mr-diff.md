Show the diff of an existing GitLab merge request for review.

## Arguments

`$ARGUMENTS` — the MR IID (internal ID), e.g. `42`

## Instructions

1. Read the MR IID from `$ARGUMENTS`.

2. Require the following environment variables (fail with a clear error message if missing):
   - `GITLAB_HOST` — base URL of your GitLab instance, e.g. `https://gitlab.example.com`
   - `GITLAB_TOKEN` — personal access token with `api` or `read_api` scope

3. Determine the project path from the git remote:
   ```bash
   git remote get-url origin
   ```
   Strip the host prefix and `.git` suffix to get `namespace/project`.
   URL-encode it by replacing `/` with `%2F`.

4. Fetch the MR details:
   ```
   GET {GITLAB_HOST}/api/v4/projects/{encoded_project_path}/merge_requests/{iid}
   Headers: PRIVATE-TOKEN: {GITLAB_TOKEN}
   ```
   Display a header with: title, author, `source_branch` → `target_branch`, state.

5. Fetch the MR changes (diff):
   ```
   GET {GITLAB_HOST}/api/v4/projects/{encoded_project_path}/merge_requests/{iid}/changes
   Headers: PRIVATE-TOKEN: {GITLAB_TOKEN}
   ```

6. For each file in the `changes` array:
   - Show the file path (`new_path`; if renamed, show `old_path` → `new_path`)
   - Indicate if it was a `new_file`, `deleted_file`, or `renamed_file`
   - Display the `diff` content in unified diff format

7. Show a summary at the end:
   - Total files changed
   - Total lines added (count `+` lines in diffs, excluding `+++` headers)
   - Total lines removed (count `-` lines in diffs, excluding `---` headers)

8. On error, display the HTTP status and error message.

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

## Alternative: GitLab CLI

If `glab` is installed and authenticated:

```bash
glab mr diff 42
```
