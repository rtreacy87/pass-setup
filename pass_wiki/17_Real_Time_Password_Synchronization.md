# Implementing Real-Time Password Synchronization

**Difficulty Level:** Advanced  
**Time to Complete:** 60 minutes

## Introduction

While scheduled synchronization provides a reliable way to keep your password store updated across systems, real-time synchronization offers immediate updates whenever changes occur. This guide will show you how to implement real-time password synchronization between your macOS and WSL Ubuntu systems using file system monitoring tools.

## Understanding Real-Time Synchronization

Real-time synchronization differs from scheduled synchronization in several key ways:

| Scheduled Synchronization | Real-Time Synchronization |
|---------------------------|---------------------------|
| Runs at fixed intervals | Triggers immediately when changes occur |
| Consistent resource usage | Resource usage varies with activity |
| Simple to implement | More complex setup |
| Works even when file system events are missed | Requires reliable file system event monitoring |

Real-time synchronization uses file system monitoring tools to detect changes to your password store and trigger synchronization scripts immediately when changes occur.

## Prerequisites

Before implementing real-time synchronization, ensure you have:

1. A working Pass setup on both macOS and WSL Ubuntu
2. Git integration configured for your password store
3. The synchronization scripts from the [Building Robust Synchronization Scripts](06_Creating_Synchronization_Scripts.md) guide

## Setting Up Real-Time Synchronization on macOS

macOS provides a powerful tool called `fswatch` for monitoring file system changes.

### Step 1: Install fswatch

```bash
# Install fswatch using Homebrew
brew install fswatch
```

### Step 2: Create a Real-Time Sync Script

Create a new script that will use fswatch to monitor your password store and trigger synchronization when changes are detected:

```bash
# Create the script file
mkdir -p ~/bin
touch ~/bin/pass-realtime-sync.sh
chmod +x ~/bin/pass-realtime-sync.sh
```

Edit the script with your preferred text editor:

```bash
nano ~/bin/pass-realtime-sync.sh
```

Add the following content:

```bash
#!/bin/bash

# Configuration
PASSWORD_STORE_DIR="${PASSWORD_STORE_DIR:-$HOME/.password-store}"
SYNC_SCRIPT="$HOME/bin/pass-sync.sh"
LOG_FILE="$HOME/.pass-realtime-sync.log"
COOLDOWN_PERIOD=5  # seconds to wait after detecting changes

# Function to log messages
log_message() {
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] $1" >> "$LOG_FILE"
    echo "$1"
}

# Function to run sync after a cooldown period
# This prevents multiple syncs when many files change at once
run_sync_with_cooldown() {
    log_message "Changes detected in password store"
    
    # Wait for the cooldown period to allow all changes to complete
    sleep $COOLDOWN_PERIOD
    
    # Run the sync script
    log_message "Running sync script after cooldown"
    "$SYNC_SCRIPT"
    
    log_message "Sync completed"
}

# Check if the sync script exists
if [ ! -f "$SYNC_SCRIPT" ]; then
    log_message "Error: Sync script not found at $SYNC_SCRIPT"
    exit 1
fi

# Check if the password store directory exists
if [ ! -d "$PASSWORD_STORE_DIR" ]; then
    log_message "Error: Password store directory not found at $PASSWORD_STORE_DIR"
    exit 1
fi

# Log startup
log_message "Starting real-time password synchronization"
log_message "Monitoring directory: $PASSWORD_STORE_DIR"

# Start monitoring the password store directory
# Exclude the .git directory to avoid triggering on Git's internal changes
fswatch -0 -e "\.git/" "$PASSWORD_STORE_DIR" | while read -d "" event; do
    run_sync_with_cooldown
done
```

### Step 3: Create a Launch Agent for Real-Time Sync

Create a launch agent to start the real-time sync script when you log in:

```bash
# Create the launch agent file
mkdir -p ~/Library/LaunchAgents
touch ~/Library/LaunchAgents/com.password-store.realtime-sync.plist
```

Edit the file:

```bash
nano ~/Library/LaunchAgents/com.password-store.realtime-sync.plist
```

Add the following content:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>Label</key>
    <string>com.password-store.realtime-sync</string>
    <key>ProgramArguments</key>
    <array>
        <string>/bin/bash</string>
        <string>-c</string>
        <string>~/bin/pass-realtime-sync.sh</string>
    </array>
    <key>RunAtLoad</key>
    <true/>
    <key>KeepAlive</key>
    <true/>
    <key>StandardOutPath</key>
    <string>~/.pass-realtime-sync.stdout.log</string>
    <key>StandardErrorPath</key>
    <string>~/.pass-realtime-sync.stderr.log</string>
</dict>
</plist>
```

### Step 4: Load the Launch Agent

```bash
# Load the launch agent
launchctl load ~/Library/LaunchAgents/com.password-store.realtime-sync.plist
```

To verify it's running:

```bash
launchctl list | grep password-store
```

## Setting Up Real-Time Synchronization on WSL Ubuntu

For WSL Ubuntu, we'll use `inotifywait` from the `inotify-tools` package to monitor file system changes.

### Step 1: Install inotify-tools

```bash
# Install inotify-tools
sudo apt update
sudo apt install -y inotify-tools
```

### Step 2: Create a Real-Time Sync Script

```bash
# Create the script file
mkdir -p ~/bin
touch ~/bin/pass-realtime-sync.sh
chmod +x ~/bin/pass-realtime-sync.sh
```

Edit the script:

```bash
nano ~/bin/pass-realtime-sync.sh
```

Add the following content:

```bash
#!/bin/bash

# Configuration
PASSWORD_STORE_DIR="${PASSWORD_STORE_DIR:-$HOME/.password-store}"
SYNC_SCRIPT="$HOME/bin/pass-sync.sh"
LOG_FILE="$HOME/.pass-realtime-sync.log"
COOLDOWN_PERIOD=5  # seconds to wait after detecting changes
LAST_SYNC_TIME=0
COOLDOWN_ACTIVE=0

# Function to log messages
log_message() {
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] $1" >> "$LOG_FILE"
    echo "$1"
}

# Function to run sync after a cooldown period
run_sync_with_cooldown() {
    # If a sync was recently performed, don't trigger another one
    CURRENT_TIME=$(date +%s)
    TIME_DIFF=$((CURRENT_TIME - LAST_SYNC_TIME))
    
    if [ $TIME_DIFF -lt $COOLDOWN_PERIOD ]; then
        return
    fi
    
    # If cooldown is already active, don't start another one
    if [ $COOLDOWN_ACTIVE -eq 1 ]; then
        return
    fi
    
    COOLDOWN_ACTIVE=1
    log_message "Changes detected in password store"
    
    # Wait for the cooldown period to allow all changes to complete
    sleep $COOLDOWN_PERIOD
    
    # Run the sync script
    log_message "Running sync script after cooldown"
    "$SYNC_SCRIPT"
    
    LAST_SYNC_TIME=$(date +%s)
    COOLDOWN_ACTIVE=0
    log_message "Sync completed"
}

# Check if the sync script exists
if [ ! -f "$SYNC_SCRIPT" ]; then
    log_message "Error: Sync script not found at $SYNC_SCRIPT"
    exit 1
fi

# Check if the password store directory exists
if [ ! -d "$PASSWORD_STORE_DIR" ]; then
    log_message "Error: Password store directory not found at $PASSWORD_STORE_DIR"
    exit 1
fi

# Log startup
log_message "Starting real-time password synchronization"
log_message "Monitoring directory: $PASSWORD_STORE_DIR"

# Start monitoring the password store directory
# The -r flag makes it recursive, -m flag for monitoring multiple events
# --exclude prevents monitoring .git directory
inotifywait -m -r -e modify,create,delete,move --exclude "\.git" "$PASSWORD_STORE_DIR" | while read -r directory event filename; do
    # Only trigger on .gpg files or directories
    if [[ "$filename" == *.gpg || -z "$filename" ]]; then
        run_sync_with_cooldown
    fi
done
```

### Step 3: Set Up Automatic Startup

Add the real-time sync script to your `.bashrc` to start it when you open a WSL terminal:

```bash
# Edit your .bashrc file
nano ~/.bashrc
```

Add the following at the end of the file:

```bash
# Start real-time password synchronization if not already running
if ! pgrep -f "pass-realtime-sync.sh" > /dev/null; then
    nohup ~/bin/pass-realtime-sync.sh > /dev/null 2>&1 &
fi
```

### Step 4: Start the Real-Time Sync

To start the real-time sync immediately without restarting your terminal:

```bash
# Start the real-time sync script
nohup ~/bin/pass-realtime-sync.sh > /dev/null 2>&1 &
```

## Testing Real-Time Synchronization

Let's test our real-time synchronization setup:

### Test 1: Create a New Password on macOS

```bash
# On macOS
pass generate test/realtime-sync 15
```

### Test 2: Check for the Password on WSL

Wait a few seconds, then check if the password appears on your WSL system:

```bash
# On WSL Ubuntu
pass show test/realtime-sync
```

### Test 3: Edit the Password on WSL

```bash
# On WSL Ubuntu
pass edit test/realtime-sync
```

Change the password, save, and exit.

### Test 4: Check for the Updated Password on macOS

Wait a few seconds, then check if the password is updated on your macOS system:

```bash
# On macOS
pass show test/realtime-sync
```

## Advanced Configuration

### Adjusting the Cooldown Period

The cooldown period prevents multiple synchronizations when many files change at once. You can adjust it by changing the `COOLDOWN_PERIOD` variable in the script:

```bash
# For a shorter cooldown (more responsive but potentially more resource-intensive)
COOLDOWN_PERIOD=2

# For a longer cooldown (less responsive but more efficient)
COOLDOWN_PERIOD=10
```

### Excluding Specific Directories

You can exclude specific directories from triggering synchronization by modifying the monitoring command:

#### On macOS (fswatch):

```bash
# Exclude both .git and a specific category
fswatch -0 -e "\.git/" -e "Finance/" "$PASSWORD_STORE_DIR" | while read -d "" event; do
```

#### On WSL (inotifywait):

```bash
# Exclude both .git and a specific category
inotifywait -m -r -e modify,create,delete,move --exclude "\.git|Finance/" "$PASSWORD_STORE_DIR" | while read -r directory event filename; do
```

### Handling Network Disconnections

To make the real-time sync more robust against network disconnections, you can add network checks to the sync script:

```bash
# Function to check network connectivity
check_network() {
    # Try to ping the Git server
    ping -c 1 github.com > /dev/null 2>&1
    return $?
}

# Modify the run_sync_with_cooldown function
run_sync_with_cooldown() {
    log_message "Changes detected in password store"
    
    # Wait for the cooldown period
    sleep $COOLDOWN_PERIOD
    
    # Check network before syncing
    if check_network; then
        log_message "Running sync script after cooldown"
        "$SYNC_SCRIPT"
        log_message "Sync completed"
    else
        log_message "Network unavailable, skipping sync"
    fi
}
```

## Troubleshooting

### Issue: High CPU Usage

If you notice high CPU usage from the monitoring tools:

1. Increase the cooldown period
2. Be more specific about which events to monitor
3. Exclude more directories from monitoring

### Issue: Changes Not Syncing

If changes aren't being synchronized:

1. Check the log files:
   ```bash
   # On macOS
   cat ~/.pass-realtime-sync.log
   
   # On WSL Ubuntu
   cat ~/.pass-realtime-sync.log
   ```

2. Verify the monitoring process is running:
   ```bash
   # On macOS
   ps aux | grep fswatch
   
   # On WSL Ubuntu
   ps aux | grep inotifywait
   ```

3. Try running the sync script manually:
   ```bash
   ~/bin/pass-sync.sh
   ```

### Issue: WSL Monitoring Stops Working

If the monitoring in WSL stops working after the system has been idle:

1. WSL might be suspending background processes
2. Consider setting up a Windows scheduled task to periodically "wake up" WSL
3. Use the scheduled synchronization approach from the previous guide as a backup

## Understanding the Limitations

Real-time synchronization has some inherent limitations:

1. **Resource Usage**: Continuous monitoring consumes more resources than scheduled tasks
2. **Network Dependency**: Requires constant network connectivity for immediate syncing
3. **Complexity**: More moving parts means more potential points of failure
4. **WSL Limitations**: WSL may suspend background processes when idle

Consider using a combination of real-time and scheduled synchronization for the most robust setup.

## Conclusion

You've now implemented real-time password synchronization between your macOS and WSL Ubuntu systems! This provides immediate updates whenever you make changes to your password store, ensuring you always have the latest passwords available on all your systems.

Remember that real-time synchronization is more complex than scheduled synchronization, so keep an eye on the log files and be prepared to troubleshoot if issues arise.

## Navigation

- [README](README.md) - Wiki Home
- Previous: [Maintenance and Administration](15_Maintenance_and_Administration.md)
- Related:
  - [Building Robust Synchronization Scripts](06_Creating_Synchronization_Scripts.md)
  - [Setting Up Scheduled Password Synchronization on macOS](07_Scheduled_Synchronization_on_macOS.md)
  - [Setting Up Scheduled Password Synchronization on WSL](08_Scheduled_Synchronization_on_WSL.md)
  - [Troubleshooting Guide](13_Troubleshooting_Guide.md)
