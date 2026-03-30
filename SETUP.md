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

## Issue 2: Node.js 20 deprecation warning

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
