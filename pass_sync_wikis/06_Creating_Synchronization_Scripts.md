# Building Robust Synchronization Scripts

**Difficulty Level:** Intermediate  
**Time to Complete:** 45 minutes

## Introduction

While Git hooks provide basic automation, we need more robust scripts to handle synchronization reliably. In this guide, we'll create comprehensive synchronization scripts for both macOS and WSL that can handle various edge cases and provide better error handling.

## Why We Need Dedicated Sync Scripts

Git hooks are great for automating actions when you make changes, but they have limitations:

1. They only run when you perform Git operations
2. They don't handle all error cases gracefully
3. They can't be easily scheduled to run periodically

A dedicated synchronization script addresses these issues by:
- Providing a single command to sync passwords
- Including comprehensive error handling
- Being suitable for scheduled execution
- Logging sync activities for troubleshooting

## Creating the Sync Script

We'll create a script called `pass-sync.sh` that will:
1. Check for local changes
2. Commit any local changes
3. Pull remote changes
4. Push local commits
5. Handle errors and conflicts
6. Log all activities

### Step 1: Create a Directory for Scripts

First, let's create a directory to store our scripts:

```bash
# Create a bin directory in your home folder
mkdir -p ~/bin
```

This command creates a directory called `bin` in your home folder (if it doesn't already exist). This is a common location for user scripts.

### Step 2: Create the Sync Script

Now, let's create the synchronization script:

```bash
# Create the sync script
cat > ~/bin/pass-sync.sh << 'EOF'
#!/bin/bash

# pass-sync.sh - Synchronize password store between systems
# Created: $(date)

# Configuration
PASSWORD_STORE_DIR="${PASSWORD_STORE_DIR:-$HOME/.password-store}"
LOG_FILE="$HOME/.pass-sync.log"
MAX_LOG_SIZE=$((1024 * 1024))  # 1MB

# Function to log messages
log_message() {
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] $1" | tee -a "$LOG_FILE"
}

# Function to rotate log if it gets too large
rotate_log() {
    if [ -f "$LOG_FILE" ] && [ $(stat -c%s "$LOG_FILE" 2>/dev/null || stat -f%z "$LOG_FILE") -gt $MAX_LOG_SIZE ]; then
        mv "$LOG_FILE" "$LOG_FILE.old"
        log_message "Log file rotated"
    fi
}

# Rotate log if needed
rotate_log

# Start sync process
log_message "Starting password store synchronization"

# Check if password store directory exists
if [ ! -d "$PASSWORD_STORE_DIR" ]; then
    log_message "ERROR: Password store directory not found at $PASSWORD_STORE_DIR"
    exit 1
fi

# Navigate to password store
cd "$PASSWORD_STORE_DIR" || {
    log_message "ERROR: Could not change to directory $PASSWORD_STORE_DIR"
    exit 1
}

# Check if this is a Git repository
if [ ! -d ".git" ]; then
    log_message "ERROR: Not a Git repository: $PASSWORD_STORE_DIR"
    exit 1
fi

# Check for local changes
if [ -n "$(git status --porcelain)" ]; then
    log_message "Local changes detected, committing..."
    
    # Add all changes
    git add . || {
        log_message "ERROR: Failed to add changes to Git"
        exit 1
    }
    
    # Commit changes
    git commit -m "Automatic sync: $(date)" || {
        log_message "ERROR: Failed to commit changes"
        exit 1
    }
    
    log_message "Changes committed successfully"
else
    log_message "No local changes detected"
fi

# Try to pull changes
log_message "Pulling changes from remote repository..."

# Stash any uncommitted changes (just in case)
git stash -q

# Pull with rebase
if git pull --rebase origin master 2>/dev/null || git pull --rebase origin main 2>/dev/null; then
    log_message "Pull successful"
    
    # Apply stashed changes if any
    git stash pop -q 2>/dev/null || true
    
    # Count updated password files
    UPDATED_FILES=$(git diff-tree -r --name-only --diff-filter=AM ORIG_HEAD HEAD 2>/dev/null | grep "\.gpg$" | wc -l)
    if [ $UPDATED_FILES -gt 0 ]; then
        log_message "$UPDATED_FILES password(s) were updated from remote"
    fi
else
    # Pull failed, try without rebase
    log_message "Rebase pull failed, trying regular pull..."
    
    # Apply stashed changes
    git stash pop -q 2>/dev/null || true
    
    if git pull origin master 2>/dev/null || git pull origin main 2>/dev/null; then
        log_message "Regular pull successful"
    else
        log_message "ERROR: Pull failed. There might be conflicts to resolve manually."
        log_message "Run 'cd $PASSWORD_STORE_DIR && git status' to check the status."
        exit 1
    fi
fi

# Push changes if we have commits to push
if [ -n "$(git log @{push}.. 2>/dev/null)" ]; then
    log_message "Pushing changes to remote repository..."
    
    if git push origin master 2>/dev/null || git push origin main 2>/dev/null; then
        log_message "Push successful"
    else
        log_message "ERROR: Failed to push changes"
        exit 1
    fi
else
    log_message "No changes to push"
fi

# Record last sync time
echo "$(date '+%Y-%m-%d %H:%M:%S')" > "$PASSWORD_STORE_DIR/.last_sync"

log_message "Synchronization completed successfully"
exit 0
EOF

# Make the script executable
chmod +x ~/bin/pass-sync.sh
```

This creates a comprehensive synchronization script with the following features:

#### Script Components Explained:

1. **Configuration Section**:
   - Sets the password store directory location
   - Configures logging
   - Sets maximum log size for rotation

2. **Logging Functions**:
   - `log_message`: Writes timestamped messages to both the console and log file
   - `rotate_log`: Prevents the log file from growing too large

3. **Initial Checks**:
   - Verifies the password store directory exists
   - Confirms it's a Git repository

4. **Local Changes Handling**:
   - Detects if there are uncommitted changes
   - Adds and commits them with a timestamp

5. **Pulling Remote Changes**:
   - Stashes any uncommitted changes
   - Attempts to pull with rebase first (preserves your commit history)
   - Falls back to regular pull if rebase fails
   - Counts and reports updated password files

6. **Pushing Local Changes**:
   - Checks if there are commits to push
   - Pushes them to the remote repository

7. **Completion**:
   - Records the last sync time
   - Reports successful completion

### Step 3: Add the Script to Your PATH

To make the script easily accessible from anywhere, add your `bin` directory to your PATH:

#### On macOS:

```bash
# Add to your .zshrc or .bash_profile
echo 'export PATH="$HOME/bin:$PATH"' >> ~/.zshrc

# Reload your shell configuration
source ~/.zshrc
```

#### On WSL:

```bash
# Add to your .bashrc
echo 'export PATH="$HOME/bin:$PATH"' >> ~/.bashrc

# Reload your shell configuration
source ~/.bashrc
```

This allows you to run the script by simply typing `pass-sync.sh` from any directory.

## Testing the Sync Script

Let's test our synchronization script:

```bash
# Run the sync script
pass-sync.sh
```

You should see output showing the synchronization process, with messages about checking for changes, pulling, and pushing.

### Test Scenarios

Try these scenarios to verify the script works correctly:

1. **Make a change on one system**:
   ```bash
   # Add a new password
   pass generate test/sync-test 15
   
   # Run the sync script
   pass-sync.sh
   ```

2. **Check on the other system**:
   ```bash
   # Run the sync script
   pass-sync.sh
   
   # Verify the password exists
   pass test/sync-test
   ```

3. **Make conflicting changes** (to test conflict handling):
   - On System A: `pass edit test/sync-test` (change the password)
   - On System B (without syncing first): `pass edit test/sync-test` (change to something different)
   - On System B: Run `pass-sync.sh` (it should handle the conflict or notify you)

## Understanding the Script's Behavior

The script follows this workflow:

1. **Check Environment**: Ensures everything is set up correctly
2. **Handle Local Changes**: Commits any pending changes
3. **Pull Remote Changes**: Gets updates from the repository
4. **Push Local Changes**: Shares your changes with the repository
5. **Log Activities**: Records what happened for troubleshooting

This provides a more robust synchronization process than Git hooks alone, with better error handling and reporting.

## Customizing the Script

You can customize the script for your specific needs:

### Change the Log Location

Edit the script and modify this line:
```bash
LOG_FILE="$HOME/.pass-sync.log"
```

### Add Notifications

For desktop notifications when passwords are updated, add this function:

```bash
# Add this function to the script
notify() {
    if command -v osascript &>/dev/null; then
        # macOS notification
        osascript -e "display notification \"$1\" with title \"Password Sync\""
    elif command -v notify-send &>/dev/null; then
        # Linux notification
        notify-send "Password Sync" "$1"
    fi
}

# Then call it when passwords are updated
if [ $UPDATED_FILES -gt 0 ]; then
    log_message "$UPDATED_FILES password(s) were updated from remote"
    notify "$UPDATED_FILES password(s) were updated"
fi
```

### Add Email Notifications for Errors

For critical errors, you might want email notifications:

```bash
# Add this function to the script
send_email() {
    if command -v mail &>/dev/null; then
        echo "$1" | mail -s "Password Sync Error" your.email@example.com
    fi
}

# Then call it when errors occur
log_message "ERROR: Pull failed. There might be conflicts to resolve manually."
send_email "Password sync error: Pull failed. Manual intervention required."
```

## Troubleshooting

### Script Not Found

If you get "command not found" when trying to run the script:

```bash
# Check if the script is executable
ls -la ~/bin/pass-sync.sh

# Make sure your PATH includes ~/bin
echo $PATH

# If needed, run with the full path
~/bin/pass-sync.sh
```

### Git Authentication Issues

If the script fails with authentication errors:

```bash
# Test your GitHub SSH connection
ssh -T git@github.com

# Check your Git remote configuration
cd ~/.password-store
git remote -v
```

### Log File Analysis

The script creates a log file at `~/.pass-sync.log`. If you're having issues:

```bash
# View the last 20 lines of the log
tail -n 20 ~/.pass-sync.log

# Search for errors
grep ERROR ~/.pass-sync.log
```

## Next Steps

Now that you have a robust synchronization script, the next step is to set up scheduled execution so that synchronization happens automatically at regular intervals.

---

**Prerequisites**: 
- [Using Git Hooks to Automate Password Synchronization](05_Understanding_Git_Hooks.md)

**Next**: 
- [Setting Up Scheduled Password Synchronization on macOS](07_Scheduled_Synchronization_on_macOS.md)
- [Setting Up Scheduled Password Synchronization on WSL](08_Scheduled_Synchronization_on_WSL.md)

**Related**:
- [Implementing Real-time Password Synchronization](09_Real_Time_Synchronization.md)
- [Resolving Password Synchronization Conflicts](10_Handling_Synchronization_Conflicts.md)
