# Advanced Features and Extensions

## Introduction

Pass can be extended with additional functionality through extensions and custom configurations. This guide covers advanced features including two-factor authentication integration, custom Git hooks for automation, backup strategies, and password sharing options.

## Two-Factor Authentication Integration

### Installing pass-otp Extension

The `pass-otp` extension allows you to store and generate Time-based One-Time Passwords (TOTP) alongside your regular passwords.

#### Installation on macOS

```bash
brew install pass-otp
```

#### Installation on WSL Ubuntu

```bash
# Install dependencies
sudo apt install -y pass qrencode oathtool

# Clone the repository
git clone https://github.com/tadfisher/pass-otp
cd pass-otp

# Install the extension
sudo make install
```

### Adding TOTP Secrets

When setting up two-factor authentication on a website, you'll typically be shown a QR code and/or given a secret key. You can add this to your Pass entry:

#### Method 1: Add TOTP URI Manually

```bash
# Edit an existing password entry
pass edit Website/example
```

Add a line with the TOTP URI:
```
MySecretPassword123
username: user@example.com
otpauth://totp/Example:user@example.com?secret=JBSWY3DPEHPK3PXP&issuer=Example
```

The URI format is:
```
otpauth://totp/Provider:user@example.com?secret=YOUR_SECRET&issuer=Provider
```

#### Method 2: Use pass-otp Insert Command

```bash
pass otp insert Website/example
```

This will prompt you to enter the TOTP secret.

#### Method 3: Import from QR Code (macOS)

```bash
# Capture a screenshot of the QR code
# Then run:
zbarimg -q --raw screenshot.png | pass otp insert Website/example
```

You'll need to install `zbar` first:
```bash
brew install zbar
```

### Generating OTP Codes

To generate a TOTP code:

```bash
pass otp Website/example
```

This will display the current TOTP code, which typically changes every 30 seconds.

### Combining Password and OTP

You can retrieve both your password and TOTP code in one command:

```bash
pass show Website/example
pass otp Website/example
```

Or create a shell function in your `.bashrc` or `.zshrc`:

```bash
pass-full() {
    pass show "$1"
    pass otp "$1"
}
```

Then use:
```bash
pass-full Website/example
```

## Custom Git Hooks for Automation

Git hooks are scripts that run automatically when certain Git events occur. You can use them to automate tasks in your password store.

### Setting Up Git Hooks Directory

First, navigate to your password store's Git hooks directory:

```bash
cd ~/.password-store/.git/hooks
```

### Automatic Push After Commit

Create a `post-commit` hook to automatically push changes after each commit:

```bash
nano post-commit
```

Add the following content:

```bash
#!/bin/bash
# Automatically push changes after commit

# Get the current branch name
BRANCH=$(git symbolic-ref --short HEAD)

# Push changes
git push origin $BRANCH

echo "Changes automatically pushed to remote repository"
```

Make the hook executable:

```bash
chmod +x post-commit
```

### Automatic Pull Before Password Access

Create a `pre-command` hook for Pass to pull changes before accessing passwords:

```bash
mkdir -p ~/.password-store/.extensions
nano ~/.password-store/.extensions/pre-command.bash
```

Add the following content:

```bash
#!/bin/bash
# Pull changes before accessing passwords

# Only pull if the command is 'show', 'edit', or 'generate'
if [[ "$1" == "show" || "$1" == "edit" || "$1" == "generate" ]]; then
    cd "$PASSWORD_STORE_DIR" || exit 1
    git pull -q origin master
fi
```

Make the hook executable:

```bash
chmod +x ~/.password-store/.extensions/pre-command.bash
```

Enable the extension by adding to your `.bashrc` or `.zshrc`:

```bash
export PASSWORD_STORE_ENABLE_EXTENSIONS=true
```

### Notification Hook

Create a hook to notify you when passwords are accessed:

```bash
nano ~/.password-store/.extensions/post-show.bash
```

Add the following content:

```bash
#!/bin/bash
# Notify when a password is accessed

PASSWORD_NAME="$1"

# On macOS
if [[ "$(uname)" == "Darwin" ]]; then
    osascript -e "display notification \"Password for $PASSWORD_NAME was accessed\" with title \"Pass\""
# On Linux
elif [[ "$(uname)" == "Linux" ]]; then
    if command -v notify-send &> /dev/null; then
        notify-send "Pass" "Password for $PASSWORD_NAME was accessed"
    fi
fi
```

Make the hook executable:

```bash
chmod +x ~/.password-store/.extensions/post-show.bash
```

## Backup Strategies

While Git provides version control and synchronization, it's important to have additional backups of your password store.

### Creating Encrypted Backups

Create a script to generate encrypted backups:

```bash
nano ~/pass-backup.sh
```

Add the following content:

```bash
#!/bin/bash
# Create an encrypted backup of the password store

# Set backup directory
BACKUP_DIR="$HOME/pass-backups"
mkdir -p "$BACKUP_DIR"

# Create timestamp
TIMESTAMP=$(date +"%Y%m%d-%H%M%S")

# Create archive of the password store
tar -czf "$BACKUP_DIR/pass-backup-$TIMESTAMP.tar.gz" -C "$HOME" .password-store

# Encrypt the archive
gpg -c "$BACKUP_DIR/pass-backup-$TIMESTAMP.tar.gz"

# Remove the unencrypted archive
rm "$BACKUP_DIR/pass-backup-$TIMESTAMP.tar.gz"

echo "Backup created: $BACKUP_DIR/pass-backup-$TIMESTAMP.tar.gz.gpg"
```

Make the script executable:

```bash
chmod +x ~/pass-backup.sh
```

### Scheduling Regular Backups

#### On macOS

Create a LaunchAgent to run the backup script regularly:

```bash
mkdir -p ~/Library/LaunchAgents
nano ~/Library/LaunchAgents/com.user.pass-backup.plist
```

Add the following content:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>Label</key>
    <string>com.user.pass-backup</string>
    <key>ProgramArguments</key>
    <array>
        <string>/bin/bash</string>
        <string>-c</string>
        <string>~/pass-backup.sh</string>
    </array>
    <key>StartCalendarInterval</key>
    <dict>
        <key>Hour</key>
        <integer>0</integer>
        <key>Minute</key>
        <integer>0</integer>
    </dict>
</dict>
</plist>
```

Load the LaunchAgent:

```bash
launchctl load ~/Library/LaunchAgents/com.user.pass-backup.plist
```

#### On WSL Ubuntu

Use cron to schedule regular backups:

```bash
crontab -e
```

Add the following line to run the backup daily at midnight:

```
0 0 * * * ~/pass-backup.sh
```

### Backup Rotation

To prevent accumulating too many backups, create a script to rotate them:

```bash
nano ~/pass-backup-rotate.sh
```

Add the following content:

```bash
#!/bin/bash
# Rotate password store backups, keeping the 7 most recent

# Set backup directory
BACKUP_DIR="$HOME/pass-backups"

# Keep only the 7 most recent backups
ls -t "$BACKUP_DIR"/*.gpg | tail -n +8 | xargs rm -f

echo "Backup rotation completed"
```

Make the script executable:

```bash
chmod +x ~/pass-backup-rotate.sh
```

Add it to your crontab or LaunchAgent to run after the backup script.

## Password Sharing Options

There are several ways to share passwords with trusted individuals.

### Multi-Key Encryption

You can encrypt your password store with multiple GPG keys, allowing multiple people to access it:

```bash
# Initialize pass with multiple keys
pass init "YOUR_KEY_ID" "RECIPIENT_KEY_ID"
```

This re-encrypts all passwords so they can be decrypted by either key.

### Sharing Individual Passwords

To share a specific password with someone:

```bash
# Encrypt a password for a specific recipient
pass show Website/example | gpg --encrypt --armor --recipient recipient@example.com > shared_password.gpg
```

You can then send the `shared_password.gpg` file to the recipient, who can decrypt it with:

```bash
gpg --decrypt shared_password.gpg
```

### Creating a Shared Password Store

For team password management, you can create a separate password store:

```bash
# Create a directory for the shared store
mkdir -p ~/shared-password-store

# Initialize a new password store with multiple keys
PASSWORD_STORE_DIR=~/shared-password-store pass init "YOUR_KEY_ID" "TEAM_MEMBER_1_KEY_ID" "TEAM_MEMBER_2_KEY_ID"

# Set up Git for the shared store
cd ~/shared-password-store
git init
git remote add origin git@github.com:team/shared-password-store.git
```

To use the shared store, you can either:

1. Set the `PASSWORD_STORE_DIR` environment variable temporarily:
   ```bash
   PASSWORD_STORE_DIR=~/shared-password-store pass show Website/example
   ```

2. Create an alias in your `.bashrc` or `.zshrc`:
   ```bash
   alias team-pass='PASSWORD_STORE_DIR=~/shared-password-store pass'
   ```

   Then use:
   ```bash
   team-pass show Website/example
   ```

## Custom Extensions

You can create your own Pass extensions to add custom functionality.

### Creating a Simple Extension

Let's create an extension that generates pronounceable passwords:

```bash
mkdir -p ~/.password-store/.extensions
nano ~/.password-store/.extensions/pronounceable.bash
```

Add the following content:

```bash
#!/bin/bash
# Pass extension to generate pronounceable passwords

pronounceable() {
    local length="${1:-15}"
    local consonants="bcdfghjklmnpqrstvwxyz"
    local vowels="aeiou"
    local password=""

    for ((i=0; i<length; i+=2)); do
        if [ $i -lt $length ]; then
            consonant=${consonants:$(( RANDOM % ${#consonants} )):1}
            password="${password}${consonant}"
        fi

        if [ $((i+1)) -lt $length ]; then
            vowel=${vowels:$(( RANDOM % ${#vowels} )):1}
            password="${password}${vowel}"
        fi
    done

    echo "$password"
}

case "$1" in
    help|--help|-h)
        echo "Usage: pass pronounceable [length]"
        echo "    Generate a pronounceable password of the specified length (default: 15)"
        ;;
    *)
        pronounceable "$1"
        ;;
esac
```

Make the extension executable:

```bash
chmod +x ~/.password-store/.extensions/pronounceable.bash
```

Enable extensions in your `.bashrc` or `.zshrc`:

```bash
export PASSWORD_STORE_ENABLE_EXTENSIONS=true
```

Use the extension:

```bash
# Generate a pronounceable password
pass pronounceable 12

# Store a pronounceable password
pass generate -p Website/example $(pass pronounceable 12)
```

## Conclusion

These advanced features and extensions significantly enhance the functionality of Pass, making it a more powerful and flexible password management solution. By implementing two-factor authentication, custom Git hooks, backup strategies, and password sharing options, you can tailor Pass to meet your specific needs while maintaining security.

Remember that each extension or customization should be evaluated for its security implications before implementation. Always prioritize the security of your password store over convenience features.

In the next guide, we'll cover troubleshooting common issues that you might encounter when using Pass across multiple systems.

## Navigation

- [README](README.md) - Wiki Home
- Previous: [GUI Clients and Browser Integration](11_GUI_Clients_and_Browser_Integration.md)
- Next: [Troubleshooting Guide](13_Troubleshooting_Guide.md)
- Related:
  - [Security Best Practices](10_Security_Best_Practices.md) - For security considerations
  - [Maintenance and Administration](15_Maintenance_and_Administration.md) - For long-term management
