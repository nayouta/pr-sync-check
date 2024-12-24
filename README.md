# pr-sync-check

This GitHub Actions workflow ensures that a pull request (PR) is up-to-date with its base branch before merging.

Below is the YAML configuration, followed by a step-by-step explanation of each part.

```yaml
name: PR Sync Check with Base Branch

on:
  pull_request:
    types: [opened, edited, synchronize, reopened, unlocked]

jobs:
  check-base-branch:
    runs-on: ubuntu-latest
    steps:
      - name: Check out PR branch
        uses: actions/checkout@v3
        with:
          fetch-depth: 0

      - name: Fetch base branch
        run: |
          git fetch origin ${{ github.event.pull_request.base.ref }} --depth=1

      - name: Compare with base branch
        run: |
          if [ "$(git rev-list HEAD..origin/${{ github.event.pull_request.base.ref }})" != "" ]; then
            echo "Warning: The PR has not incorporated the latest commit from the base branch."
            echo "You need to merge or rebase the base branch and then commit again."
            exit 1
          else
            echo "Notice: The PR is synchronized with the base branch."
          fi
```

## How it works

### 1. Workflow name

* `name: PR Sync Check with Base Branch`

  This is just a descriptive name for the workflow, making it easier to identify in the GitHub Actions tab.

### 2. Trigger (on: pull_request)

* This workflow is triggered whenever a pull request is:
  * opened
  * edited
  * synchronized (i.e., new commits are pushed to the PR branch)
  * reopened
  * unlocked
* The workflow runs automatically on those PR events.

### 3. Job: `check-base-branch`

* `runs-on: ubuntu-latest`

  The job executes on an Ubuntu runner provided by GitHub.

### 4. Steps

#### 1. Check out PR branch

```yaml
uses: actions/checkout@v3
with:
  fetch-depth: 0
```

* Clones the pull request branch.
* `fetch-depth: 0` is set to ensure the entire commit history is available. This is crucial for comparing commits across branches.

#### 2. Fetch base branch

```yaml
run: |
  git fetch origin ${{ github.event.pull_request.base.ref }} --depth=1
```

* Fetches the latest commit(s) from the base branch (e.g., `main` or `develop`), using `--depth=1` for efficiency.

#### 3. Compare with base branch

```yaml
run: |
  if [ "$(git rev-list HEAD..origin/${{ github.event.pull_request.base.ref }})" != "" ]; then
    echo "Warning: The PR has not incorporated the latest commit from the base branch."
    echo "You need to merge or rebase the base branch and then commit again."
    exit 1
  else
    echo "Notice: The PR is synchronized with the base branch."
  fi
```

* Uses `git rev-list` to check if the HEAD commit of the PR branch includes all the commits from the base branch.

* If commits are missing (the command returns a non-empty result), the script exits with an error, causing the workflow to fail.

* If there is no missing commit, the workflow considers it up to date and prints a confirmation message.

## Outcome

### If the PR branch is behind the base branch

* The workflow fails and instructs the contributor to merge or rebase the latest changes from the base branch.

### If the PR branch is up to date

* The workflow passes, indicating there are no additional merges or rebases needed.

### This ensures that a pull request is synchronized with the latest changes on the base branch before further review or merging
