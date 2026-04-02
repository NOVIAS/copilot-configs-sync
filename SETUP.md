# Setup Guide

This document explains how to configure the repository to resolve common GitHub Actions workflow issues.

## Issue 1: GitHub Actions is not permitted to create or approve pull requests

### Error Message
```
pull request create failed: GraphQL: GitHub Actions is not permitted to create or approve pull requests (createPullRequest)
```

### Solution

You need to enable GitHub Actions to create pull requests. Choose one of the following options:

#### Option 1: Enable in Repository Settings (Recommended)

1. Go to your repository on GitHub
2. Click **Settings** (top right)
3. Click **Actions** → **General** (left sidebar)
4. Scroll down to **Workflow permissions**
5. Check the box: **"Allow GitHub Actions to create and approve pull requests"**
6. Click **Save**

#### Option 2: Use a Personal Access Token (PAT)

If you prefer not to grant PR creation permissions to all workflows, you can use a PAT:

1. Create a Personal Access Token:
   - Go to https://github.com/settings/tokens
   - Click **Generate new token** → **Generate new token (classic)**
   - Give it a name like "Copilot Configs Sync"
   - Select scopes: `repo` (Full control of private repositories)
   - Click **Generate token**
   - **Copy the token** (you won't see it again)

2. Add the token as a repository secret:
   - Go to your repository **Settings** → **Secrets and variables** → **Actions**
   - Click **New repository secret**
   - Name: `PAT_TOKEN`
   - Value: paste the token you copied
   - Click **Add secret**

3. Update the workflow file (`.github/workflows/sync-copilot-configs.yml`):
   - Find the "Create or update pull request" step
   - Comment out: `GH_TOKEN: ${{ secrets.GITHUB_TOKEN }}`
   - Uncomment: `GH_TOKEN: ${{ secrets.PAT_TOKEN }}`

## Issue 2: Pull request Branch does not have required protected branch rules (enablePullRequestAutoMerge)

### Error Message
```
GraphQL: Pull request Branch does not have required protected branch rules (enablePullRequestAutoMerge)
Error: Process completed with exit code 1
```

### Root Cause
The repository does not have the auto-merge feature enabled. The workflow attempts to automatically merge pull requests using `gh pr merge --auto`, but the repository doesn't support this feature.

### Solution

Enable auto-merge in your repository settings:

1. Go to your repository on GitHub
2. Click **Settings** (top right)
3. Click **General** (left sidebar)
4. Scroll down to **Pull Requests** section
5. Check the box: **"Allow auto-merge"**
6. Select a default merge method (recommended: **Squash and merge**)
7. Click **Save**

After enabling this feature, the workflow will be able to automatically merge pull requests without errors.

### Alternative Solution (Disable Auto-merge in Workflow)

If you don't want to enable auto-merge, you can modify the workflow to remove the auto-merge step:

In `.github/workflows/sync-copilot-configs.yml`, comment out or remove the **"Enable auto-merge on pull request"** step. The PR will still be created but won't be merged automatically.

---

## Issue 3: Your local changes to the following files would be overwritten by checkout

### Error Message
```
error: Your local changes to the following files would be overwritten by checkout:
  SYNC_README.md
Please commit your changes or stash them before you switch branches.
```

### Root Cause
The workflow generates `SYNC_README.md` in the "Update sync readme" step, but then tries to switch branches in the "Commit synced changes" step without properly staging the changes first. This causes a git conflict.

### Solution

**This has been fixed in the latest version of the workflow.** The workflow now:

1. **Stages all changes first** using `git add` before switching branches
2. **Checks for staged changes** using `git diff --cached` instead of `git status`
3. **Avoids git conflicts** by ensuring no unstaged changes when switching branches

**If you're still facing this issue:**

1. Update your workflow file to the latest version
2. Go to **Actions** tab
3. Select **Sync Copilot Configs** workflow
4. Click **Run workflow**

---

## Issue 4: Node.js 20 deprecation warning

### Warning Message
```
Warning: Node.js 20 is deprecated. The following actions target Node.js 20 but are being forced to run on Node.js 24: actions/checkout@v4.2.1
```

### Solution

This has been fixed in the workflow by:
- Upgrading `actions/checkout@v4.2.1` to `actions/checkout@v4`
- The v4 version automatically uses the appropriate Node.js version for the runner

The warning should disappear in the next workflow run.

## Testing the Workflow

After making the changes:

1. Go to **Actions** tab in your repository
2. Select **Sync Copilot Configs** workflow
3. Click **Run workflow** → **Run workflow**
4. Wait for the workflow to complete
5. If successful, you should see a new pull request created

## Additional Notes

- The workflow runs automatically daily at 00:00 Beijing time (16:00 UTC)
- Weekly branches are created every Monday
- You can manually trigger the workflow at any time from the Actions tab
