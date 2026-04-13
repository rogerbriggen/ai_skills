Create a new merge request in a self-hosted GitLab instance.

## Arguments

`$ARGUMENTS` — space-separated options:
- `--source <branch>` — source branch (default: current git branch)
- `--target <branch>` — target branch (default: repository default branch)
- `--title <title>` — MR title (default: last commit message)
- `--description <text>` — MR description (optional)

Example: `--source feature/my-feature --target main --title "Add my feature" --description "Adds new functionality"`

## Instructions

1. Parse `$ARGUMENTS` for the options above.
   - If `--source` is omitted, run `git branch --show-current` to get the current branch.
   - If `--title` is omitted, run `git log -1 --format=%s` to use the last commit message.
   - If `--target` is omitted, leave it unspecified and let GitLab use the project default branch.

2. Determine which method to use:
   - If `GITLAB_TOKEN` environment variable is **not** set → use **Option A: GitLab CLI**.
   - If `GITLAB_TOKEN` environment variable **is** set → use **Option B: REST API with curl**.

---

### Option A: GitLab CLI

```bash
glab mr create \
  --source-branch <source> \
  --target-branch <target> \
  --title "<title>" \
  --description "<description>"
```

Omit `--target-branch` if `--target` was not specified (glab uses the project default).
Omit `--description` if no description was provided.

Skip to step 5.

---

### Option B: REST API with curl

3. Require the following environment variables (fail with a clear error message if missing):
   - `GITLAB_HOST` — base URL of your GitLab instance, e.g. `https://gitlab.example.com`
   - `GITLAB_TOKEN` — personal access token with `api` scope

4. Determine the project path from the git remote:
   ```bash
   git remote get-url origin
   ```
   Strip the host prefix and `.git` suffix to get `namespace/project`.
   URL-encode it by replacing `/` with `%2F` (e.g. `my-group%2Fmy-project`).

   Create the merge request via the GitLab API:
   ```
   POST {GITLAB_HOST}/api/v4/projects/{encoded_project_path}/merge_requests
   Headers: PRIVATE-TOKEN: {GITLAB_TOKEN}
            Content-Type: application/json
   Body: {
     "source_branch": "<source>",
     "target_branch": "<target>",
     "title": "<title>",
     "description": "<description>"
   }
   ```
   Omit `target_branch` from the body if it was not specified.

---

5. On success (HTTP 201 for curl, or CLI confirmation for glab), display:
   - **MR IID**: `iid` from the response
   - **Title**: `title`
   - **URL**: `web_url`
   - **State**: `state`
   - **Source → Target**: `source_branch` → `target_branch`

6. On error, display the HTTP status and error message.

## Example curl command (for reference, Option B)

```bash
SOURCE_BRANCH=$(git branch --show-current)
TITLE=$(git log -1 --format=%s)
PROJECT_PATH=$(git remote get-url origin | sed 's|.*gitlab[^/]*/||; s|\.git$||' | sed 's|/|%2F|g')

curl -s --request POST \
  --header "PRIVATE-TOKEN: $GITLAB_TOKEN" \
  --header "Content-Type: application/json" \
  --data "{
    \"source_branch\": \"$SOURCE_BRANCH\",
    \"target_branch\": \"main\",
    \"title\": \"$TITLE\",
    \"description\": \"\"
  }" \
  "$GITLAB_HOST/api/v4/projects/$PROJECT_PATH/merge_requests"
```
