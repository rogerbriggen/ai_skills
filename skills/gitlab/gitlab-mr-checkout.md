Check out a GitLab merge request branch locally for review.

## Arguments

`$ARGUMENTS` — the MR IID (internal ID), e.g. `42`

## Instructions

1. Read the MR IID from `$ARGUMENTS`.

2. Check whether `glab` is available:
   ```bash
   which glab
   ```
   - **If available**, use Option A (GitLab CLI — preferred).
   - **Otherwise**, use Option B (REST API + git).

---

### Option A: GitLab CLI

```bash
glab mr checkout <iid>
```

This fetches the MR's source branch and checks it out locally. Skip to step 6.

---

### Option B: REST API + git

3. Require the following environment variables (fail with a clear error message if missing):
   - `GITLAB_HOST` — base URL of your GitLab instance, e.g. `https://gitlab.example.com`
   - `GITLAB_TOKEN` — personal access token with `api` or `read_api` scope

4. Determine the project path from the git remote:
   ```bash
   git remote get-url origin
   ```
   Strip the host prefix and `.git` suffix. URL-encode `/` as `%2F`.

5. Fetch the MR details to get the source branch:
   ```
   GET {GITLAB_HOST}/api/v4/projects/{encoded_project_path}/merge_requests/{iid}
   Headers: PRIVATE-TOKEN: {GITLAB_TOKEN}
   ```
   Extract `source_branch` and `sha` from the response.

   Then fetch and check out the branch:
   ```bash
   git fetch origin <source_branch>
   git checkout <source_branch>
   ```
   If the branch doesn't exist locally yet:
   ```bash
   git checkout -b <source_branch> origin/<source_branch>
   ```

---

6. Confirm success by displaying:
   - MR title and IID
   - Source branch name
   - Latest commit SHA (`sha`)

## Example curl command (for reference)

```bash
PROJECT_PATH=$(git remote get-url origin | sed 's|.*gitlab[^/]*/||; s|\.git$||' | sed 's|/|%2F|g')

curl -s --header "PRIVATE-TOKEN: $GITLAB_TOKEN" \
  "$GITLAB_HOST/api/v4/projects/$PROJECT_PATH/merge_requests/42"
```
