# Using Git Hooks to Automate Password Synchronization

**Difficulty Level:** Intermediate
**Time to Complete:** 30 minutes

## Introduction

In this guide, we'll set up Git hooks to automate the synchronization of your password store between your macOS and WSL systems. Git hooks are scripts that run automatically when certain Git events occur, making them perfect for our automation needs.

## What are Git Hooks?

Git hooks are scripts that Git executes before or after events such as commit, push, and receive. They allow you to customize Git's behavior and automate tasks.

Think of Git hooks as automatic triggers that run whenever you perform certain Git actions. For example, a pre-commit hook runs before a commit is created, and a post-commit hook runs after a commit is created.

## Types of Git Hooks Relevant for Password Synchronization

For our password synchronization system, we'll use these hooks:

1. **pre-commit**: Runs before a commit is created
   - We'll use this to pull the latest changes before committing
   - This helps prevent conflicts

2. **post-commit**: Runs after a commit is created
   - We'll use this to push changes immediately after committing
   - This ensures your changes are shared right away

3. **post-merge**: Runs after a merge operation (like after a pull)
   - We'll use this to notify you when changes are pulled from the repository
   - This helps you know when passwords have been updated

## Where Git Hooks Are Stored

Git hooks are stored in the `.git/hooks` directory of your Git repository. In our case, they'll be in:

- macOS: `~/.password-store/.git/hooks/`
- WSL: `~/.password-store/.git/hooks/`

Each hook is a separate file named after the hook type (e.g., `pre-commit`, `post-commit`).

## Creating the Git Hooks

We'll create these hooks on both your macOS and WSL systems. The process is almost identical for both.

### Step 1: Navigate to Your Password Store

```bash
# Navigate to your password store
cd ~/.password-store
```

### Step 2: Create the pre-commit Hook

This hook will pull the latest changes before creating a commit:

```bash
# Create the pre-commit hook file
cat > .git/hooks/pre-commit << 'EOF'
#!/bin/bash

echo "Running pre-commit hook: Pulling latest changes..."

# Save the current state of the index
git rev-parse HEAD > .git/ORIG_HEAD_PRECOMMIT

# Stash everything, including staged changes
echo "Stashing all changes..."
git stash save --include-untracked --quiet "pre-commit-stash"
STASH_RESULT=$?

if [ $STASH_RESULT -eq 0 ]; then
    # Pull the latest changes
    echo "Pulling latest changes..."
    git pull --rebase origin master || git pull --rebase origin main
    PULL_STATUS=$?

    # Restore the stashed changes
    echo "Restoring changes..."
    git stash pop --quiet

    # If pull failed, report it but allow commit to proceed
    if [ $PULL_STATUS -ne 0 ]; then
        echo "Warning: Pull failed, but proceeding with commit."
        echo "You may need to resolve conflicts manually later."
    fi
else
    echo "No changes to stash or stash failed. Proceeding without pull."
fi

echo "Pre-commit hook completed."
EOF

# Make the hook executable
chmod +x .git/hooks/pre-commit
```

What this script does:
- Saves the current HEAD commit for reference
- Stashes ALL changes, including both staged and unstaged changes
- Pulls the latest changes from the repository with rebase
- Tries both `master` and `main` branch names (to handle different GitHub defaults)
- Restores the stashed changes, including the staged ones
- Handles errors gracefully, allowing the commit to proceed even if the pull fails
- Provides detailed feedback about what's happening at each step

### Step 3: Create the post-commit Hook

This hook will push your changes immediately after committing:

```bash
# Create the post-commit hook file
cat > .git/hooks/post-commit << 'EOF'
#!/bin/bash

echo "Running post-commit hook: Pushing changes..."

# Push the changes
git push origin master || git push origin main

echo "Post-commit hook completed."
EOF

# Make the hook executable
chmod +x .git/hooks/post-commit
```

What this script does:
- Pushes your committed changes to the repository
- Tries both `master` and `main` branch names
- Provides feedback about what's happening

### Step 4: Create the post-merge Hook

This hook will notify you when changes are pulled from the repository:

```bash
# Create the post-merge hook file
cat > .git/hooks/post-merge << 'EOF'
#!/bin/bash

echo "Running post-merge hook: Password store updated from remote."

# Count how many password files were updated
UPDATED_FILES=$(git diff-tree -r --name-only --diff-filter=AM ORIG_HEAD HEAD | grep "\.gpg$" | wc -l)

if [ $UPDATED_FILES -gt 0 ]; then
    echo "$UPDATED_FILES password(s) were updated."
else
    echo "No passwords were updated."
fi

echo "Post-merge hook completed."
EOF

# Make the hook executable
chmod +x .git/hooks/post-merge
```

What this script does:
- Counts how many password files (`.gpg` files) were updated in the merge
- Displays a message showing how many passwords were updated
- Provides feedback about what's happening

## Testing the Git Hooks

Let's test our hooks to make sure they're working correctly:

### Test 1: Create a New Password

```bash
# Add a new password
pass generate test/hook-test 15
```

When you run this command:
1. Pass will generate a random 15-character password
2. It will automatically commit the change (because we enabled Git auto-commit)
3. The pre-commit hook will run and pull the latest changes
4. The post-commit hook will run and push your changes

### Test 2: Verify on the Other System

Now, go to your other system (if you set up the hooks on macOS, go to WSL, or vice versa):

```bash
# Pull the latest changes manually (just this once)
cd ~/.password-store
git pull

# Check if the new password exists
pass test/hook-test
```

You should see the password you created on the first system.

### Test 3: Make a Change on the Second System

```bash
# Edit the password
pass edit test/hook-test
```

Change the password to something else, save, and exit.

This will:
1. Automatically commit the change
2. The pre-commit hook will run and pull the latest changes
3. The post-commit hook will run and push your changes

### Test 4: Verify on the First System

Go back to the first system:

```bash
# Pull the latest changes manually (just this once)
cd ~/.password-store
git pull

# Check if the password was updated
pass test/hook-test
```

You should see the updated password.

## Understanding What's Happening

With these hooks in place, here's what happens when you make changes to your password store:

1. You add, edit, or delete a password using Pass
2. Pass automatically commits the change to Git
3. The pre-commit hook pulls the latest changes from the repository
4. Git creates the commit with your changes
5. The post-commit hook pushes your changes to the repository
6. On your other system, when you pull changes (either manually or through automation we'll set up later), the post-merge hook notifies you about updated passwords

This creates a workflow where:
- Changes are automatically shared between systems
- Conflicts are minimized by pulling before committing
- You get feedback about what's happening

## Troubleshooting

### "Permission denied" Errors

If you see "Permission denied" errors when the hooks try to run:

```bash
# Check the permissions on the hook files
ls -la .git/hooks/

# If needed, make them executable
chmod +x .git/hooks/pre-commit .git/hooks/post-commit .git/hooks/post-merge
```

### "Cannot pull with rebase: Your index contains uncommitted changes" Error

This common error occurs because of how Pass and Git hooks interact:

1. When you run a Pass command (like `pass insert`), Pass first creates and encrypts the password file
2. Pass then stages this file (adds it to the Git index)
3. At this point, your pre-commit hook runs
4. The hook tries to pull with rebase, but Git refuses because you have staged changes

#### Solution 1: Use the Improved Pre-commit Hook (Recommended)

The improved pre-commit hook in this guide handles this issue by stashing all changes, including staged ones, before pulling.

If you're still experiencing this error with the improved hook, make sure you're using the latest version of the hook and that it's properly executable:

```bash
chmod +x .git/hooks/pre-commit
```

#### Solution 2: Use a Pre-push Hook Instead

If you continue to have issues with the pre-commit hook, you can use a pre-push hook instead:

```bash
cat > .git/hooks/pre-push << 'EOF'
#!/bin/bash

echo "Running pre-push hook: Pulling latest changes..."

# Fetch the latest changes
git fetch origin

# Check if we need to pull
LOCAL=$(git rev-parse @)
REMOTE=$(git rev-parse @{u})

if [ "$LOCAL" != "$REMOTE" ]; then
    echo "Remote has changes. Pulling with rebase..."
    git pull --rebase origin master || git pull --rebase origin main

    if [ $? -ne 0 ]; then
        echo "Pull failed. Please resolve conflicts manually."
        exit 1
    fi
fi

echo "Pre-push hook completed."
EOF
chmod +x .git/hooks/pre-push
```

This hook runs after the commit is complete but before pushing, avoiding the staged changes issue.

#### Solution 3: Use a Post-commit Hook for Pulling

Another approach is to move the pull operation to a post-commit hook:

```bash
cat > .git/hooks/post-commit << 'EOF'
#!/bin/bash

echo "Running post-commit hook: Pulling and pushing changes..."

# Pull with rebase
git pull --rebase origin master || git pull --rebase origin main
PULL_STATUS=$?

# If pull succeeded, push changes
if [ $PULL_STATUS -eq 0 ]; then
    git push origin master || git push origin main
else
    echo "Warning: Pull failed. Please resolve conflicts manually and then push."
fi

echo "Post-commit hook completed."
EOF
chmod +x .git/hooks/post-commit
```

This hook pulls after the commit is complete, avoiding the staged changes issue.

#### Solution 4: Disable Automatic Git Integration in Pass

As a last resort, you can disable Pass's automatic Git integration and handle synchronization manually:

```bash
# Disable automatic Git integration
export PASSWORD_STORE_ENABLE_GIT=0

# Create a sync alias
echo 'alias pass-sync="cd ~/.password-store && git add . && git commit -m \"Password store update\" && git pull --rebase && git push"' >> ~/.bashrc
source ~/.bashrc
```

### Git Push/Pull Errors

If the hooks fail to push or pull:

```bash
# Check your Git remote configuration
git remote -v

# Verify your SSH connection to GitHub
ssh -T git@github.com
```

### Conflicts During Pull

If you get merge conflicts during the pre-commit hook:

```bash
# Abort the commit
git reset

# Manually pull and resolve conflicts
git pull
# (resolve conflicts)
git add .
git commit -m "Resolved conflicts"
```

## Ensuring Hooks Trigger for All Password Operations

By default, Pass only triggers Git operations (and thus Git hooks) when adding new passwords, but not when editing or removing existing ones. Let's fix this to ensure all password changes are properly synchronized.

### Configuring Pass to Use Git for All Operations

To make Pass use Git for all operations (add, edit, remove), you need to set the `PASSWORD_STORE_ENABLE_GIT` environment variable:

```bash
# Add to your .bashrc or .zshrc file
echo 'export PASSWORD_STORE_ENABLE_GIT=true' >> ~/.bashrc
source ~/.bashrc
```

This ensures that Pass will:
- Commit changes when you add a new password
- Commit changes when you edit an existing password
- Commit changes when you remove a password

Without this setting, only new passwords would trigger Git operations and hooks.

### Testing Git Integration for All Operations

Let's test that Git operations are triggered for all types of password changes:

```bash
# Create a test password
pass generate test/git-hook-test 15

# Edit the password
pass edit test/git-hook-test

# Remove the password
pass rm test/git-hook-test
```

For each operation, you should see:
1. The pre-commit hook running and pulling changes
2. The post-commit hook running and pushing changes

If you don't see the hooks running for edit or remove operations, check that:
- The `PASSWORD_STORE_ENABLE_GIT` environment variable is set
- You've restarted your terminal or sourced your `.bashrc`/`.zshrc` file

## Changing the Default Editor in Pass

By default, Pass uses the editor specified in your environment variables (usually `nano`). If you prefer a different editor, you can change it.

### Setting the Editor Environment Variable

Pass uses the `EDITOR` environment variable to determine which editor to use. To change it:

```bash
# For Vim
echo 'export EDITOR=vim' >> ~/.bashrc

# For VS Code
echo 'export EDITOR="code --wait"' >> ~/.bashrc

# For Emacs
echo 'export EDITOR=emacs' >> ~/.bashrc

# Apply the changes
source ~/.bashrc
```

### Testing Your Editor Change

To test that your editor change has taken effect:

```bash
# Create a new password to edit
pass generate test/editor-test 15

# Edit the password - should open in your chosen editor
pass edit test/editor-test
```

### Editor-Specific Tips

#### Vim

If you're using Vim, you can create a specific configuration for editing passwords:

```bash
# Create a Vim configuration for Pass
cat > ~/.vim/ftdetect/pass.vim << 'EOF'
autocmd BufNewFile,BufRead /dev/shm/pass.* setlocal noswapfile
EOF
```

This prevents Vim from creating swap files when editing passwords, which is more secure.

#### VS Code

When using VS Code, the `--wait` flag is important - it makes the command wait until the file is closed in VS Code before continuing. Without this, Pass might continue before you've finished editing.

#### Terminal-Based Editors in WSL

If you're using WSL and prefer terminal-based editors like Vim or Nano, no special configuration is needed.

If you want to use a Windows GUI editor from WSL, you'll need to create a wrapper script. For example, for Notepad++:

```bash
# Create a wrapper script
cat > ~/bin/notepadpp << 'EOF'
#!/bin/bash
FILE=$(wslpath -w "$1")
/mnt/c/Program\ Files/Notepad++/notepad++.exe "$FILE"
sleep 1  # Wait for Notepad++ to open
while tasklist.exe | grep -q notepad++.exe; do
    sleep 1
done
EOF
chmod +x ~/bin/notepadpp

# Set it as your editor
echo 'export EDITOR=~/bin/notepadpp' >> ~/.bashrc
source ~/.bashrc
```

## Next Steps

Now that you have Git hooks set up to automate the basic synchronization process, the next step is to create more robust synchronization scripts that can handle edge cases and provide better error handling.

---

**Prerequisites**:
- [Installing and Configuring Pass on macOS](03_Configuring_Pass_on_macOS.md)
- [Installing and Configuring Pass on WSL](04_Configuring_Pass_on_WSL.md)

**Next**: [Building Robust Synchronization Scripts](06_Creating_Synchronization_Scripts.md)

**Related**:
- [Setting Up Scheduled Password Synchronization on macOS](07_Scheduled_Synchronization_on_macOS.md)
- [Setting Up Scheduled Password Synchronization on WSL](08_Scheduled_Synchronization_on_WSL.md)
