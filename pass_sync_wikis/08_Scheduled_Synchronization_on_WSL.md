# Setting Up Scheduled Password Synchronization on WSL

**Difficulty Level:** Intermediate  
**Time to Complete:** 30 minutes

## Introduction

In this guide, we'll set up scheduled synchronization of your password store on Windows Subsystem for Linux (WSL). This complements the macOS scheduling we set up in the previous guide, ensuring your passwords stay in sync across both systems automatically.

## What is Cron?

Linux systems, including WSL, use a scheduler called `cron` to run tasks at specified times. Cron is a time-based job scheduler that allows you to run commands or scripts automatically at specified intervals.

Think of cron as a scheduler that can run programs:
- At specific times of day
- On specific days of the week
- At regular intervals (every 5 minutes, hourly, daily, etc.)
- Without requiring user interaction

## Setting Up Cron in WSL

Let's configure cron to run our synchronization script at regular intervals:

### Step 1: Install Cron (if not already installed)

First, make sure cron is installed on your WSL system:

```bash
# Update package lists
sudo apt update

# Install cron
sudo apt install -y cron

# Start the cron service
sudo service cron start
```

These commands:
- Update your package lists to ensure you get the latest version
- Install the cron package if it's not already installed
- Start the cron service so it can run scheduled tasks

### Step 2: Configure Cron to Start Automatically

By default, the cron service doesn't start automatically when you launch WSL. Let's configure it to start automatically:

```bash
# Create a startup script
cat > ~/start-cron.sh << 'EOF'
#!/bin/bash
# Check if cron is running
if ! service cron status > /dev/null; then
    # Start cron if it's not running
    sudo service cron start
    echo "Cron service started"
else
    echo "Cron service is already running"
fi
EOF

# Make the script executable
chmod +x ~/start-cron.sh

# Add it to your .bashrc to run at startup
echo "~/start-cron.sh" >> ~/.bashrc
```

These commands:
- Create a script that checks if cron is running and starts it if needed
- Make the script executable
- Add the script to your `.bashrc` file so it runs when you open a WSL terminal

### Step 3: Create a Cron Job for Password Synchronization

Now, let's set up a cron job to run our synchronization script every 5 minutes:

```bash
# Open the crontab editor
crontab -e
```

This will open a text editor (usually nano or vi). If this is your first time using crontab, you might be asked to choose an editor. Nano is generally easier for beginners.

Add the following line to the file:

```
*/5 * * * * ~/bin/pass-sync.sh >> ~/.pass-sync.log 2>&1
```

Save the file and exit the editor:
- In nano: Press Ctrl+O to save, then Enter, then Ctrl+X to exit
- In vi: Press Esc, then type `:wq` and press Enter

Let's break down what this cron job does:

#### Cron Job Components Explained:

1. **Time Specification**: `*/5 * * * *`
   - The five fields represent: minute, hour, day of month, month, day of week
   - `*/5` in the first position means "every 5 minutes"
   - The asterisks in the other positions mean "every hour, every day, every month, every day of week"

2. **Command**: `~/bin/pass-sync.sh`
   - The path to our synchronization script

3. **Output Redirection**: `>> ~/.pass-sync.log 2>&1`
   - `>>` appends output to the specified file
   - `~/.pass-sync.log` is the log file where output will be saved
   - `2>&1` redirects error messages (stderr) to the same file as standard output

### Step 4: Verify the Cron Job

Let's verify that our cron job has been set up correctly:

```bash
# List your cron jobs
crontab -l
```

You should see the line we added in the previous step.

## Testing the Scheduled Synchronization

To test that our scheduled synchronization is working:

1. Make a change to your password store on macOS
2. Wait at least 5 minutes for both systems to sync
3. Check if the change appears on your WSL system

Alternatively, you can check the log file to see if the sync script is running:

```bash
# Wait a few minutes, then check the log
cat ~/.pass-sync.log
```

You should see entries showing that the sync script has run.

## Managing Your Cron Job

Here are some useful commands for managing your cron job:

### Editing Your Cron Jobs

If you need to make changes to your cron jobs:

```bash
# Open the crontab editor
crontab -e
```

### Removing All Cron Jobs

If you want to remove all your cron jobs:

```bash
# Remove all cron jobs
crontab -r
```

Be careful with this command, as it removes all your cron jobs without confirmation.

### Changing the Sync Interval

If you want to change how often synchronization occurs:

1. Open the crontab editor:
   ```bash
   crontab -e
   ```

2. Modify the time specification:
   - `*/5 * * * *` = Every 5 minutes
   - `*/10 * * * *` = Every 10 minutes
   - `*/30 * * * *` = Every 30 minutes
   - `0 * * * *` = Every hour (at minute 0)

3. Save and exit the editor

### Running the Sync Manually

Even with scheduled synchronization, you can still run the sync script manually:

```bash
# Run the sync script manually
~/bin/pass-sync.sh
```

## Troubleshooting

### Cron Not Running

If cron doesn't seem to be running:

```bash
# Check cron status
sudo service cron status

# If it's not running, start it
sudo service cron start
```

### Cron Job Not Running

If your cron job doesn't seem to be running:

```bash
# Check if the script is executable
ls -la ~/bin/pass-sync.sh

# Make it executable if needed
chmod +x ~/bin/pass-sync.sh

# Try running the script manually to see if it works
~/bin/pass-sync.sh
```

### Path Issues

Cron runs with a limited environment, which can cause path-related issues. If your script can't find commands it needs:

1. Edit your crontab:
   ```bash
   crontab -e
   ```

2. Add a PATH definition at the top:
   ```
   PATH=/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin:$HOME/bin
   */5 * * * * ~/bin/pass-sync.sh >> ~/.pass-sync.log 2>&1
   ```

3. Save and exit the editor

### Log File Getting Too Large

If your log file grows too large:

```bash
# Rotate the log file manually
mv ~/.pass-sync.log ~/.pass-sync.log.old
```

Our sync script already includes log rotation, but you can do it manually if needed.

## Understanding What's Happening

With cron set up, here's what happens:

1. Every 5 minutes (or your chosen interval), cron runs the sync script
2. The script:
   - Checks for local changes and commits them
   - Pulls changes from the remote repository
   - Pushes any local commits
   - Logs all activities

This creates a system where:
- Your passwords are regularly synchronized
- Changes made on your macOS system are pulled to your WSL system
- Changes made on your WSL system are pushed to your macOS system
- Everything happens automatically in the background

## WSL-Specific Considerations

### WSL Shutdown Behavior

WSL doesn't always stay running in the background like a traditional Linux system. When you close all WSL terminals, WSL might shut down, which means cron will stop running.

To mitigate this:

1. Keep at least one WSL terminal open
2. Use the Windows Task Scheduler to periodically wake up WSL (advanced)
3. Consider using Windows Terminal's "quake mode" to keep a terminal readily available

### Windows Startup

To ensure cron starts when Windows boots:

1. Create a Windows shortcut that opens WSL
2. Add this shortcut to your Windows startup folder

## Next Steps

Now that you have scheduled synchronization set up on both macOS and WSL, you have a fully automated password synchronization system! The next guide will show you how to implement real-time synchronization for even faster updates.

---

**Prerequisites**: 
- [Building Robust Synchronization Scripts](06_Creating_Synchronization_Scripts.md)
- [Setting Up Scheduled Password Synchronization on macOS](07_Scheduled_Synchronization_on_macOS.md)

**Next**: [Implementing Real-time Password Synchronization](09_Real_Time_Synchronization.md)

**Related**:
- [Resolving Password Synchronization Conflicts](10_Handling_Synchronization_Conflicts.md)
- [Securing Your Automated Password Synchronization](11_Security_Best_Practices.md)
