# Setting Up Scheduled Password Synchronization on macOS

**Difficulty Level:** Intermediate
**Time to Complete:** 30 minutes

## Introduction

In this guide, we'll set up scheduled synchronization of your password store on macOS. This ensures your passwords stay in sync even when you're not actively using the `pass` command.

## What is launchd?

macOS uses a system called `launchd` to manage scheduled tasks and services. It's similar to cron on Linux but more powerful and integrated with macOS. We'll use `launchd` to run our synchronization script at regular intervals.

Think of `launchd` as a system that can start programs:
- At specific times
- At regular intervals
- When certain events occur
- When you log in
- When the system boots

## Creating a Launch Agent

A Launch Agent is a special configuration file that tells `launchd` what to run and when. We'll create one for our password synchronization.

### Step 1: Create the Launch Agent Directory

First, make sure the Launch Agents directory exists:

```bash
# Create the LaunchAgents directory if it doesn't exist
mkdir -p ~/Library/LaunchAgents
```

This command creates a directory called `LaunchAgents` in your Library folder (if it doesn't already exist). This is where user-specific launch agents are stored.

### Step 2: Create the Launch Agent Configuration

Now, let's create the configuration file:

```bash
# Create the launch agent plist file
cat > ~/Library/LaunchAgents/com.user.pass-sync.plist << 'EOF'
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>Label</key>
    <string>com.user.pass-sync</string>
    <key>ProgramArguments</key>
    <array>
        <string>/bin/bash</string>
        <string>-c</string>
        <string>~/bin/pass-sync.sh</string>
    </array>
    <key>StartInterval</key>
    <integer>300</integer>
    <key>RunAtLoad</key>
    <true/>
    <key>StandardErrorPath</key>
    <string>~/.pass-sync-error.log</string>
    <key>StandardOutPath</key>
    <string>~/.pass-sync-output.log</string>
    <key>EnvironmentVariables</key>
    <dict>
        <key>PATH</key>
        <string>/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin:/opt/homebrew/bin</string>
    </dict>
</dict>
</plist>
EOF
```

Let's break down what this configuration does:

#### Key Components Explained:

1. **Label**: `com.user.pass-sync`
   - A unique identifier for this launch agent
   - Follows the reverse domain name convention

2. **ProgramArguments**:
   - Specifies the command to run
   - Uses `/bin/bash -c` to ensure the script runs in a proper shell environment
   - Points to our sync script at `~/bin/pass-sync.sh`

3. **StartInterval**: `300`
   - Runs the script every 300 seconds (5 minutes)
   - You can adjust this value based on your needs

4. **RunAtLoad**: `true`
   - Runs the script when the launch agent is loaded
   - This means it will run when you log in

5. **StandardErrorPath** and **StandardOutPath**:
   - Redirects error and output messages to log files
   - Helps with troubleshooting

6. **EnvironmentVariables**:
   - Sets the PATH environment variable
   - Ensures the script can find all necessary commands

### Step 3: Fix the Path in the Launch Agent

The tilde (`~`) in the paths won't be expanded by launchd. Let's replace them with your actual home directory:

```bash
# Replace ~ with your actual home directory
sed -i '' "s|~/bin/pass-sync.sh|$HOME/bin/pass-sync.sh|g" ~/Library/LaunchAgents/com.user.pass-sync.plist
sed -i '' "s|~/.pass-sync-error.log|$HOME/.pass-sync-error.log|g" ~/Library/LaunchAgents/com.user.pass-sync.plist
sed -i '' "s|~/.pass-sync-output.log|$HOME/.pass-sync-output.log|g" ~/Library/LaunchAgents/com.user.pass-sync.plist
```

These commands replace the tilde (`~`) with your actual home directory path in the launch agent configuration file.

### Step 4: Load the Launch Agent

Now, let's load the launch agent to start the scheduled synchronization:

```bash
# Load the launch agent
launchctl load ~/Library/LaunchAgents/com.user.pass-sync.plist
```

This command tells `launchd` to load our configuration and start the scheduled task. The sync script will now run:
- Immediately (because of `RunAtLoad`)
- Every 5 minutes thereafter (because of `StartInterval`)

## Verifying the Setup

Let's verify that our launch agent is properly loaded and running:

```bash
# Check if the launch agent is loaded
launchctl list | grep pass-sync
```

You should see output that includes `com.user.pass-sync`, indicating that the launch agent is loaded.

To check if it's actually running:

```bash
# Wait a few minutes, then check the output log
cat ~/.pass-sync-output.log
```

You should see log entries showing that the sync script has run.

## Managing the Launch Agent

Here are some useful commands for managing your launch agent:

### Stopping the Scheduled Synchronization

If you need to temporarily stop the synchronization:

```bash
# Unload the launch agent
launchctl unload ~/Library/LaunchAgents/com.user.pass-sync.plist
```

### Starting it Again

To start it again after stopping:

```bash
# Load the launch agent
launchctl load ~/Library/LaunchAgents/com.user.pass-sync.plist
```

### Running the Sync Manually

Even with scheduled synchronization, you can still run the sync script manually:

```bash
# Run the sync script manually
~/bin/pass-sync.sh
```

### Changing the Sync Interval

If you want to change how often synchronization occurs:

1. Edit the launch agent configuration:
   ```bash
   # Open the file in a text editor
   nano ~/Library/LaunchAgents/com.user.pass-sync.plist
   ```

2. Change the `StartInterval` value:
   - 300 = 5 minutes
   - 600 = 10 minutes
   - 1800 = 30 minutes
   - 3600 = 1 hour

3. Save the file and reload the launch agent:
   ```bash
   launchctl unload ~/Library/LaunchAgents/com.user.pass-sync.plist
   launchctl load ~/Library/LaunchAgents/com.user.pass-sync.plist
   ```

## Troubleshooting

### Launch Agent Not Running

If the launch agent doesn't seem to be running:

```bash
# Check for errors in the error log
cat ~/.pass-sync-error.log

# Try running the script manually to see if it works
~/bin/pass-sync.sh

# Make sure the script is executable
chmod +x ~/bin/pass-sync.sh
```

### Path Issues

If the script can't find commands it needs:

```bash
# Edit the launch agent to update the PATH
nano ~/Library/LaunchAgents/com.user.pass-sync.plist

# Find the PATH line and add any missing directories
# For example, if you installed Git with Homebrew, you might need:
# /opt/homebrew/bin
```

### Permission Issues

If there are permission problems:

```bash
# Check permissions on the script
ls -la ~/bin/pass-sync.sh

# Check permissions on the password store
ls -la ~/.password-store

# Fix if needed
chmod +x ~/bin/pass-sync.sh
chmod -R 700 ~/.password-store
```

## Understanding What's Happening

With the launch agent set up, here's what happens:

1. When you log in to your Mac, the launch agent is loaded
2. The sync script runs immediately
3. Every 5 minutes (or your chosen interval), the script runs again
4. The script:
   - Checks for local changes and commits them
   - Pulls changes from the remote repository
   - Pushes any local commits
   - Logs all activities

This creates a system where:
- Your passwords are regularly synchronized
- Changes made on your WSL system are pulled to your Mac
- Changes made on your Mac are pushed to your WSL system
- Everything happens automatically in the background

## Next Steps

Now that you have scheduled synchronization set up on macOS, the next step is to set up similar scheduling on your WSL system to complete the automation.

---

**Prerequisites**:
- [Building Robust Synchronization Scripts](06_Creating_Synchronization_Scripts.md)

**Next**: [Setting Up Scheduled Password Synchronization on WSL](08_Scheduled_Synchronization_on_WSL.md)

**Related**:
- [Implementing Real-time Password Synchronization](../pass_wiki/16_Real_Time_Password_Synchronization.md)
- [Resolving Password Synchronization Conflicts](10_Handling_Synchronization_Conflicts.md)
