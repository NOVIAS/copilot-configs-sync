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

The workflow attempts to enable auto-merge on pull requests using `gh pr merge --auto`, but:

1. **Auto-merge is not enabled** in your repository settings, OR
2. **Branch protection rules** are conflicting with auto-merge settings

### Quick Diagnosis: Check Your Repository Settings

#### Step 1: Verify Auto-merge is Enabled

1. Go to your GitHub repository
2. Click **Settings** (top right)
3. Click **General** (left sidebar)
4. Scroll down to **Pull Requests** section
5. **Look for**: `✓ Allow auto-merge` checkbox
   - ✅ If **checked**: Auto-merge is enabled
   - ❌ If **unchecked**: Enable it now

#### Step 2: Check Branch Protection Rules (if applicable)

1. Go to **Settings** → **Branches**
2. Look for **Branch protection rules** 
3. Check if there are rules for:
   - `main` / default branch
   - `automation/*` pattern
   - Verify they don't conflict with auto-merge

#### Step 3: Verify Workflow Permissions

1. Go to **Settings** → **Actions** → **General**
2. Find **Workflow permissions**
3. **Confirm**: `✓ Allow GitHub Actions to create and approve pull requests` is checked

### Solution

Choose **one** of the following approaches:

#### ✅ Recommended Solution: Enable Auto-merge in Repository Settings

1. Go to **Settings** → **General**
2. Find **Pull Requests** section
3. **Check** the box: `✓ Allow auto-merge`
4. **Select merge method**: Squash and merge (recommended)
5. Click **Save**
6. Run the workflow again

**Status**: After this, the workflow will automatically merge PRs when the `automation/sync-copilot-configs` branch is synced.

#### ✅ Alternative Solution: Workflow Now Handles Auto-merge Gracefully

The workflow has been updated to:

- ✅ **Attempt auto-merge** when it's available
- ✅ **Continue gracefully** if auto-merge is not supported (doesn't fail the workflow)
- ✅ **Display helpful messages** instructing users to enable auto-merge

If auto-merge isn't available, the PR will still be created successfully, and you can merge it manually.

**To see if this is working**:
1. Run the workflow
2. Check the logs for "✓ Auto-merge enabled successfully" or "⚠ Auto-merge not available"
3. If auto-merge is unavailable, check the settings as described above

### Why This Matters

When the `automation/sync-copilot-configs` branch already exists from a previous sync attempt, subsequent runs may encounter:
- Stale branch protection rules
- Existing PR configurations
- Rebase conflicts

**Enabling auto-merge ensures clean, automated merges** without manual intervention.

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
