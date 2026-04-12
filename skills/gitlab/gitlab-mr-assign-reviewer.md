Assign yourself as a reviewer on a GitLab merge request.

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

4. Get the current authenticated user's ID and username:
   ```
   GET {GITLAB_HOST}/api/v4/user
   Headers: PRIVATE-TOKEN: {GITLAB_TOKEN}
   ```
   Extract `id` (numeric) and `username` from the response.

5. Fetch the MR to get its existing reviewers (to avoid removing them):
   ```
   GET {GITLAB_HOST}/api/v4/projects/{encoded_project_path}/merge_requests/{iid}
   Headers: PRIVATE-TOKEN: {GITLAB_TOKEN}
   ```
   Extract the `reviewers` array (list of `{id, username}` objects).

6. Check if your user ID is already in the existing reviewer list.
   - If yes: display `@{username} is already a reviewer on MR !{iid}` and stop.
   - If no: build the new reviewer ID list by appending your ID to the existing ones.

7. Update the MR with the new reviewer list:
   ```
   PUT {GITLAB_HOST}/api/v4/projects/{encoded_project_path}/merge_requests/{iid}
   Headers: PRIVATE-TOKEN: {GITLAB_TOKEN}
            Content-Type: application/json
   Body: {"reviewer_ids": [<id1>, <id2>, ...]}
   ```

8. On success, display: `Successfully added @{username} as reviewer on MR !{iid}`.
9. On error, display the HTTP status and error message.

## Example curl commands (for reference)

```bash
PROJECT_PATH=$(git remote get-url origin | sed 's|.*gitlab[^/]*/||; s|\.git$||' | sed 's|/|%2F|g')

# Step 1: Get current user ID
curl -s --header "PRIVATE-TOKEN: $GITLAB_TOKEN" \
  "$GITLAB_HOST/api/v4/user"

# Step 2: Get existing MR reviewers
curl -s --header "PRIVATE-TOKEN: $GITLAB_TOKEN" \
  "$GITLAB_HOST/api/v4/projects/$PROJECT_PATH/merge_requests/42"

# Step 3: Add yourself as reviewer (replace USER_ID with your numeric ID)
curl -s --request PUT \
  --header "PRIVATE-TOKEN: $GITLAB_TOKEN" \
  --header "Content-Type: application/json" \
  --data '{"reviewer_ids": [USER_ID]}' \
  "$GITLAB_HOST/api/v4/projects/$PROJECT_PATH/merge_requests/42"
```

## Alternative: GitLab CLI

If `glab` is installed and authenticated:

```bash
glab mr update 42 --reviewer @me
```
