# Synchronization Workflow Between Systems

## Introduction

With Pass set up on both your macOS and WSL Ubuntu systems, it's important to establish a consistent workflow for keeping your passwords synchronized. This guide will cover best practices for synchronization, handling conflicts, and automating the process.

## Understanding Git-Based Synchronization

Pass uses Git for version control and synchronization. When you make changes to your password store, those changes are committed to your local Git repository. To share these changes with your other devices, you need to push them to a remote repository and then pull them on your other devices.

## Daily Workflow for Password Management

Here's a recommended workflow for managing your passwords across multiple systems:

### Step 1: Pull Before Making Changes

Before adding, editing, or removing passwords, always pull the latest changes from your remote repository:

```bash
# On macOS or WSL Ubuntu
pass git pull
```

This command:
- Connects to your remote repository
- Downloads any changes made on other devices
- Merges those changes into your local password store

Making this a habit ensures you're always working with the most up-to-date version of your passwords.

### Step 2: Make Your Changes

Now you can make changes to your password store:

```bash
# Add a new password
pass insert Category/service-name

# Generate a new password
pass generate Category/service-name 15

# Edit an existing password
pass edit Category/service-name

# Remove a password
pass rm Category/service-name
```

### Step 3: Commit Your Changes

If you've enabled automatic Git integration (`PASSWORD_STORE_ENABLE_GIT=true`), Pass will automatically commit your changes. If not, you'll need to commit them manually:

```bash
pass git add .
pass git commit -m "Add new service password"
```

Use descriptive commit messages that explain what changes you made.

### Step 4: Push Your Changes

After committing, push your changes to the remote repository:

```bash
pass git push
```

This command uploads your local commits to the remote repository, making them available to your other devices.

### Step 5: Pull on Other Devices

On your other devices, pull the changes:

```bash
# On your other device (macOS or WSL Ubuntu)
pass git pull
```

This updates the password store on your other device with the changes you made.

## Synchronization Best Practices

Follow these best practices to ensure smooth synchronization:

### 1. Always Pull Before Making Changes

This reduces the chance of conflicts and ensures you're working with the latest data.

```bash
pass git pull
```

### 2. Push After Making Changes

Make it a habit to push your changes immediately after making them:

```bash
pass git push
```

### 3. Use Descriptive Commit Messages

Good commit messages help you understand what changes were made and when:

```bash
pass git commit -m "Add login for new work email account"
```

### 4. Check Git Status Regularly

To see if you have uncommitted changes or need to push/pull:

```bash
pass git status
```

This shows:
- Changes that haven't been committed
- Commits that haven't been pushed
- Whether your local repository is behind the remote

### 5. Review Git Log

To see the history of changes:

```bash
pass git log
```

This shows all commits, including:
- Who made the change
- When it was made
- The commit message

## Handling Conflicts

If you make changes to the same password on different devices without synchronizing, you might encounter conflicts when pulling or pushing.

### Identifying Conflicts

When you run `pass git pull` and there's a conflict, you'll see a message like:

```
CONFLICT (content): Merge conflict in Email/gmail.gpg
Automatic merge failed; fix conflicts and then commit the result.
```

### Resolving Conflicts

Since password files are encrypted, you can't resolve conflicts in the usual way. Instead:

#### Option 1: Keep Your Local Version

If you want to keep the version on your current system:

```bash
pass git checkout --ours Email/gmail.gpg
pass git add Email/gmail.gpg
pass git commit -m "Resolve conflict, keep local version"
```

These commands:
1. Select your local version of the file
2. Stage the file for commit
3. Commit the resolution

#### Option 2: Keep the Remote Version

If you want to keep the version from the remote repository:

```bash
pass git checkout --theirs Email/gmail.gpg
pass git add Email/gmail.gpg
pass git commit -m "Resolve conflict, keep remote version"
```

#### Option 3: Manually Resolve (Advanced)

For a more careful approach:

```bash
# Decrypt both versions
gpg -d ~/.password-store/Email/gmail.gpg > ~/local_version.txt
git show origin/master:Email/gmail.gpg | gpg -d > ~/remote_version.txt

# Compare them
diff ~/local_version.txt ~/remote_version.txt

# Edit the password manually
pass edit Email/gmail.gpg

# Mark as resolved
pass git add Email/gmail.gpg
pass git commit -m "Manually resolve conflict in gmail password"

# Clean up
shred -u ~/local_version.txt ~/remote_version.txt
```

This approach is more complex but allows you to see both versions and create a merged result.

## Automation Options

You can automate parts of the synchronization process to make it more convenient.

### Option 1: Git Hooks

Git hooks are scripts that run automatically at certain points in the Git workflow.

#### Creating a Post-Commit Hook to Automatically Push

```bash
# Navigate to your password store's Git hooks directory
cd ~/.password-store/.git/hooks

# Create a post-commit hook
nano post-commit
```

Add the following to the file:

```bash
#!/bin/bash
# Automatically push changes after commit
git push origin master
```

Make the hook executable:

```bash
chmod +x post-commit
```

Now, whenever you commit changes (or Pass auto-commits them), they'll be automatically pushed to your remote repository.

#### Creating a Post-Merge Hook to Notify You of Changes

```bash
# Navigate to your password store's Git hooks directory
cd ~/.password-store/.git/hooks

# Create a post-merge hook
nano post-merge
```

Add the following to the file:

```bash
#!/bin/bash
# Notify when changes are pulled
echo "Password store updated with changes from remote repository"
```

Make the hook executable:

```bash
chmod +x post-merge
```

### Option 2: Scheduled Synchronization

You can set up a cron job to regularly synchronize your password store:

```bash
# Edit your crontab
crontab -e
```

Add a line to pull changes every hour:

```
0 * * * * cd ~/.password-store && git pull
```

This runs `git pull` in your password store directory at the top of every hour.

### Option 3: Alias Commands

Create aliases for common synchronization tasks:

```bash
# Edit your .bashrc or .zshrc
nano ~/.bashrc
```

Add these aliases:

```bash
# Pass synchronization aliases
alias pass-sync='cd ~/.password-store && git pull && git push'
alias pass-status='cd ~/.password-store && git status'
alias pass-history='cd ~/.password-store && git log --pretty=format:"%h %ad | %s" --date=short -10'
```

After saving, run:

```bash
source ~/.bashrc
```

Now you can use:
- `pass-sync` to pull and push changes in one command
- `pass-status` to check the synchronization status
- `pass-history` to see the last 10 changes

## Troubleshooting Synchronization Issues

### Issue: Cannot Push Changes

If you see an error like:
```
error: failed to push some refs to 'git@github.com:yourusername/password-store.git'
```

Try pulling first:
```bash
pass git pull
```

Then try pushing again:
```bash
pass git push
```

### Issue: Changes Not Showing on Other Device

If you've pushed changes but don't see them on your other device:

1. Make sure you've pulled the changes:
   ```bash
   pass git pull
   ```

2. Check if you're on the correct branch:
   ```bash
   pass git branch
   ```

3. Verify the remote repository is correctly configured:
   ```bash
   pass git remote -v
   ```

### Issue: Merge Conflicts

If you frequently encounter merge conflicts:

1. Synchronize more often
2. Try to avoid editing the same password on different devices without synchronizing
3. Consider using the automation options described above

## Conclusion

Congratulations! You've learned:
- A daily workflow for password management across multiple systems
- Best practices for synchronization
- How to handle conflicts
- Options for automating the synchronization process
- Troubleshooting common synchronization issues

By following these practices, you'll ensure your passwords stay in sync across your devices, giving you secure and convenient access to your passwords wherever you are.

In the next guide, we'll cover security best practices to keep your password management system safe.

## Navigation

- [README](README.md) - Wiki Home
- Previous: [Setting Up Pass on WSL Ubuntu](08_Setting_Up_Pass_on_WSL_Ubuntu.md)
- Next: [Security Best Practices](10_Security_Best_Practices.md)
- Related:
  - [Git Integration for Pass](04_Git_Integration_for_Pass.md) - For Git setup details
  - [Troubleshooting Guide](13_Troubleshooting_Guide.md) - For help with synchronization issues
