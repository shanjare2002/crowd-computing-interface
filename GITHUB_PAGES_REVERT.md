# How to Revert to a Previous GitHub Pages Deployment

This guide explains how to revert your GitHub Pages site to a previous deployment when you need to roll back changes.

## Table of Contents
- [Understanding GitHub Pages Deployments](#understanding-github-pages-deployments)
- [Method 1: Using GitHub UI (Recommended for Beginners)](#method-1-using-github-ui-recommended-for-beginners)
- [Method 2: Using Git Commands](#method-2-using-git-commands)
- [Method 3: Using GitHub CLI](#method-3-using-github-cli)
- [Method 4: Reverting GitHub Actions Deployment](#method-4-reverting-github-actions-deployment)
- [Troubleshooting](#troubleshooting)

---

## Understanding GitHub Pages Deployments

GitHub Pages automatically deploys your site whenever changes are pushed to the configured branch (usually `main`, `master`, or `gh-pages`). Each deployment corresponds to a specific Git commit.

To revert a deployment, you need to:
1. Identify the commit you want to revert to
2. Update your branch to point to that commit
3. Push the changes to trigger a new deployment

---

## Method 1: Using GitHub UI (Recommended for Beginners)

### Step 1: View Deployment History
1. Navigate to your repository on GitHub: `https://github.com/USERNAME/REPOSITORY`
2. Click on the **"Environments"** link in the right sidebar
3. Click on **"github-pages"**
4. You'll see a list of all deployments with their commit messages and timestamps

### Step 2: Find the Commit to Revert To
1. In the deployments list, identify the deployment you want to revert to
2. Click on the deployment to see details
3. Note the **commit SHA** (a string like `abc1234`)

### Step 3: Revert Using GitHub UI
1. Go to the repository's **"Code"** tab
2. Click on the **commits history** (e.g., "42 commits")
3. Find the commit you want to revert to
4. Click the **"<>"** (Browse repository at this point in history) button on that commit
5. You have two options:

   **Option A: Create a new branch and PR**
   - Click the branch dropdown (shows a SHA)
   - Type a new branch name like `revert-to-working-version`
   - Click "Create branch"
   - Create a Pull Request to merge this into your main branch
   - Merge the PR to deploy

   **Option B: Direct revert (advanced)**
   - Copy the commit SHA
   - Go to your local repository and follow Method 2

---

## Method 2: Using Git Commands

This method requires Git installed on your computer and a local clone of the repository.

### Prerequisites
```bash
# Clone your repository if you haven't already
git clone https://github.com/USERNAME/REPOSITORY.git
cd REPOSITORY
```

### Option A: Revert to a Specific Commit (Creates New Commit)

This is the **safest method** as it preserves history:

```bash
# 1. Ensure you're on the correct branch
git checkout main  # or gh-pages, depending on your setup

# 2. Pull the latest changes
git pull origin main

# 3. Find the commit SHA you want to revert to
git log --oneline  # Shows recent commits

# 4. Revert to that commit (replace COMMIT_SHA with actual SHA)
git revert --no-commit COMMIT_SHA..HEAD

# 5. Commit the revert
git commit -m "Revert to commit COMMIT_SHA"

# 6. Push to GitHub (triggers new deployment)
git push origin main
```

### Option B: Reset to a Specific Commit (Rewrites History)

⚠️ **Warning**: This rewrites history. Only use if you're sure and working alone.

```bash
# 1. Find the commit to revert to
git log --oneline

# 2. Reset to that commit
git reset --hard COMMIT_SHA

# 3. Force push (safer option - checks if remote has changed)
git push --force-with-lease origin main

# Only use --force if you're absolutely certain and working alone:
# git push --force origin main
```

### Option C: Create a Revert Commit

For reverting a specific problematic commit:

```bash
# 1. Find the bad commit
git log --oneline

# 2. Revert that specific commit
git revert COMMIT_SHA

# 3. Push the revert commit
git push origin main
```

---

## Method 3: Using GitHub CLI

If you have [GitHub CLI](https://cli.github.com/) installed:

### Step 1: View Deployments
```bash
# List recent deployments (requires jq to be installed: apt-get install jq or brew install jq)
gh api repos/USERNAME/REPOSITORY/deployments | jq '.[0:5]'

# Alternative without jq:
gh api repos/USERNAME/REPOSITORY/deployments --jq '.[0:5]'
```

### Step 2: View Deployment Statuses
```bash
# Get deployment ID from the previous command output (look for "id" field)
# Then check its status (replace DEPLOYMENT_ID with the actual ID number)
gh api repos/USERNAME/REPOSITORY/deployments/DEPLOYMENT_ID/statuses

# Example: If the deployment ID is 123456789
gh api repos/USERNAME/REPOSITORY/deployments/123456789/statuses
```

### Step 3: Use Git Commands
The GitHub CLI doesn't directly revert deployments, so you'll need to use Method 2 (Git commands) to actually revert the code.

---

## Method 4: Reverting GitHub Actions Deployment

If your site uses GitHub Actions for deployment:

### Step 1: View Workflow Runs
1. Go to the **"Actions"** tab in your repository
2. Find the workflow run that corresponds to the working deployment
3. Note its commit SHA

### Step 2: Re-run a Previous Workflow
1. Click on the successful workflow run you want to redeploy
2. Click **"Re-run all jobs"** button (top right)
3. However, this might not work if the code has changed

### Step 3: Revert Code (Recommended)
Use Method 2 to revert your code to the working commit, which will trigger a new deployment automatically.

---

## Troubleshooting

### Issue: Changes Not Appearing After Push

**Solutions:**
1. **Wait a few minutes**: Deployments can take 1-5 minutes
2. **Check deployment status**: Go to Settings → Pages to see if there are any errors
3. **Clear browser cache**: Hard refresh (Ctrl+Shift+R or Cmd+Shift+R)
4. **Verify correct branch**: Ensure you pushed to the branch configured for GitHub Pages

### Issue: "Failed to Deploy" Error

**Solutions:**
1. Check the Actions tab for deployment logs
2. Ensure your HTML/files are valid
3. Check that the deployment branch is set correctly in Settings → Pages

### Issue: Can't Find Previous Deployments

**Solutions:**
1. Check the **Environments** section (only visible if you've had deployments)
2. Use `git log` to see commit history
3. Check the **Actions** tab if using workflows

### Issue: Force Push is Rejected

**Solutions:**
1. You may not have permission to force push (protected branch)
2. Ask repository admin to temporarily disable branch protection
3. Use Method 2, Option A instead (creating new commits)

### Issue: Site Shows Old Content

**Solutions:**
1. **Hard refresh** your browser (Ctrl+Shift+R)
2. **Clear browser cache completely**
3. Try accessing in **incognito/private mode**
4. Check if GitHub Pages CDN is cached (can take up to 10 minutes)

---

## Best Practices

1. ✅ **Always test locally first** before deploying
2. ✅ **Use version tags** for important releases: `git tag v1.0.0`
3. ✅ **Keep a changelog** of what each deployment changes
4. ✅ **Use pull requests** to review changes before deployment
5. ✅ **Avoid force pushing** to shared branches
6. ✅ **Document your deployment process** in your README

---

## Quick Reference Commands

```bash
# View recent commits
git log --oneline -10

# View detailed commit history
git log --graph --oneline --decorate

# Revert last commit
git revert HEAD
git push origin main

# Reset to specific commit (dangerous!)
git reset --hard COMMIT_SHA

# Use --force-with-lease instead of --force (safer - checks if remote has changed)
git push --force-with-lease origin main

# Only use plain --force if you're absolutely certain and working alone
# git push --force origin main

# Create a branch from old commit
git checkout -b revert-branch COMMIT_SHA
git push origin revert-branch

# View remote branches
git branch -r

# Check GitHub Pages deployment status
git log --oneline origin/gh-pages
```

---

## Additional Resources

- [GitHub Pages Documentation](https://docs.github.com/en/pages)
- [Git Revert Documentation](https://git-scm.com/docs/git-revert)
- [Git Reset Documentation](https://git-scm.com/docs/git-reset)
- [GitHub Actions for Pages](https://github.com/actions/deploy-pages)

---

## Need Help?

If you're still having issues:
1. Check the [GitHub Community Forum](https://github.community/)
2. Review your repository's Issues tab for similar problems
3. Contact the repository maintainer
4. Refer to [GitHub Support](https://support.github.com/)

