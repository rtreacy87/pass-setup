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

# Stash any changes not being committed
git stash -q --keep-index

# Pull the latest changes
git pull --rebase origin master || git pull --rebase origin main

# Restore stashed changes
git stash pop -q

echo "Pre-commit hook completed."
EOF

# Make the hook executable
chmod +x .git/hooks/pre-commit
```

What this script does:
- Temporarily stashes any changes that aren't part of the current commit
- Pulls the latest changes from the repository with rebase
- Tries both `master` and `main` branch names (to handle different GitHub defaults)
- Restores the stashed changes
- Provides feedback about what's happening

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
