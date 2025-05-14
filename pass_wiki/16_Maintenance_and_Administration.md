# Maintenance and Administration

## Introduction

A password management system requires ongoing maintenance to ensure it remains secure, efficient, and reliable. This guide covers long-term administration tasks including key rotation, repository maintenance, backup and recovery procedures, user management for shared systems, and auditing practices.

## Key Rotation Procedures

Regularly rotating your GPG keys is an important security practice that limits the impact of potential key compromises.

### When to Rotate Keys

Consider rotating your GPG keys:
- According to a regular schedule (annually or bi-annually)
- After a suspected security breach
- When a key's expiration date is approaching
- When moving to a new primary device

### Creating a New GPG Key

```bash
# Generate a new GPG key
gpg --full-generate-key
```

Follow the prompts to create a new key with strong parameters:
- RSA and RSA (default)
- 4096 bits
- Appropriate expiration (1-2 years recommended)
- Your name and email
- A strong passphrase

### Exporting the New Key

```bash
# Get the ID of your new key
gpg --list-secret-keys --keyid-format LONG

# Export the public key
gpg --armor --export NEW_KEY_ID > new-public-key.asc

# Export the private key
gpg --armor --export-secret-key NEW_KEY_ID > new-private-key.asc

# Export the trust database
gpg --export-ownertrust > new-trust.txt
```

### Re-encrypting Your Password Store

Re-encrypt your password store with both the old and new keys to ensure a smooth transition:

```bash
# Initialize pass with both keys
pass init OLD_KEY_ID NEW_KEY_ID
```

This command re-encrypts all your passwords so they can be decrypted with either key.

### Testing the New Key

Verify that you can access your passwords with the new key:

```bash
# Try accessing a password
pass show Email/example
```

If successful, you can now remove the old key from your password store:

```bash
# Re-initialize with only the new key
pass init NEW_KEY_ID
```

This re-encrypts all passwords to be only accessible with the new key.

### Distributing the New Key

Transfer the new key to all your devices using the secure methods described in earlier guides:

```bash
# Create an encrypted archive
tar -czf new-keys.tar.gz new-public-key.asc new-private-key.asc new-trust.txt
gpg -c new-keys.tar.gz
```

Transfer this encrypted archive to each device, then import the keys:

```bash
# Import the new keys
gpg --import new-public-key.asc
gpg --import new-private-key.asc
gpg --import-ownertrust new-trust.txt
```

On each device, re-initialize Pass with the new key:

```bash
pass init NEW_KEY_ID
```

### Revoking the Old Key (Optional)

If you're rotating keys due to a security concern, consider revoking the old key:

```bash
# Generate a revocation certificate
gpg --gen-revoke OLD_KEY_ID > revocation.asc

# Import the revocation certificate
gpg --import revocation.asc
```

If you've published your public key to key servers, upload the revoked key:

```bash
gpg --keyserver hkps://keys.openpgp.org --send-keys OLD_KEY_ID
```

## Repository Maintenance

Regular maintenance of your Git repository ensures it remains efficient and reliable.

### Cleaning Up the Repository

Over time, your Git repository can accumulate unnecessary data. Periodically clean it up:

```bash
# Navigate to your password store
cd ~/.password-store

# Check the repository size
du -sh .git

# Run garbage collection
git gc

# Check the new size
du -sh .git
```

The `git gc` command collects garbage by removing unnecessary files and optimizing the repository.

### Pruning Old Branches

If you use branches in your password repository, prune old ones:

```bash
# List all branches
git branch -a

# Delete a local branch
git branch -d old-branch-name

# Delete a remote branch
git push origin --delete old-branch-name
```

### Optimizing Repository Performance

For large password stores, optimize performance:

```bash
# Optimize the repository
git gc --aggressive --prune=now
```

This performs a more aggressive optimization, which can significantly reduce repository size.

### Checking Repository Integrity

Periodically check your repository for corruption:

```bash
# Check repository integrity
git fsck
```

This command verifies the connectivity and validity of objects in the database.

## Backup and Recovery

While Git provides version control, additional backups are essential for disaster recovery.

### Creating Comprehensive Backups

Set up a regular backup routine that includes:

1. Your GPG keys
2. Your password store
3. Your Git repository (including the `.git` directory)

Create a backup script:

```bash
nano ~/pass-full-backup.sh
```

Add the following content:

```bash
#!/bin/bash
# Comprehensive backup script for Pass

# Set backup directory
BACKUP_DIR="$HOME/pass-backups"
mkdir -p "$BACKUP_DIR"

# Create timestamp
TIMESTAMP=$(date +"%Y%m%d-%H%M%S")

# Backup GPG keys
gpg --export --armor YOUR_KEY_ID > "$BACKUP_DIR/public-key-$TIMESTAMP.asc"
gpg --export-secret-keys --armor YOUR_KEY_ID > "$BACKUP_DIR/private-key-$TIMESTAMP.asc"
gpg --export-ownertrust > "$BACKUP_DIR/trust-$TIMESTAMP.txt"

# Backup password store including Git repository
tar -czf "$BACKUP_DIR/password-store-$TIMESTAMP.tar.gz" -C "$HOME" .password-store

# Encrypt the backup
gpg -c "$BACKUP_DIR/password-store-$TIMESTAMP.tar.gz"
gpg -c "$BACKUP_DIR/public-key-$TIMESTAMP.asc"
gpg -c "$BACKUP_DIR/private-key-$TIMESTAMP.asc"
gpg -c "$BACKUP_DIR/trust-$TIMESTAMP.txt"

# Remove unencrypted files
rm "$BACKUP_DIR/password-store-$TIMESTAMP.tar.gz"
rm "$BACKUP_DIR/public-key-$TIMESTAMP.asc"
rm "$BACKUP_DIR/private-key-$TIMESTAMP.asc"
rm "$BACKUP_DIR/trust-$TIMESTAMP.txt"

echo "Backup completed: $BACKUP_DIR"
```

Make the script executable:

```bash
chmod +x ~/pass-full-backup.sh
```

### Automating Backups

Schedule regular backups:

On macOS:
```bash
# Create a LaunchAgent
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
        <string>~/pass-full-backup.sh</string>
    </array>
    <key>StartCalendarInterval</key>
    <dict>
        <key>Day</key>
        <integer>1</integer>
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

On Linux/WSL:
```bash
# Add to crontab
crontab -e
```

Add the following line for weekly backups:
```
0 0 * * 0 ~/pass-full-backup.sh
```

### Storing Backups Securely

Store your backups in multiple secure locations:
- External encrypted hard drive
- Secure cloud storage
- Physical safe or secure location

Regularly test your backups by performing recovery drills.

### Recovery Procedures

Document the recovery process:

1. Restore GPG keys:
   ```bash
   # Decrypt the key backups
   gpg -d private-key-backup.asc.gpg > private-key.asc
   gpg -d public-key-backup.asc.gpg > public-key.asc
   gpg -d trust-backup.txt.gpg > trust.txt

   # Import the keys
   gpg --import public-key.asc
   gpg --import private-key.asc
   gpg --import-ownertrust trust.txt
   ```

2. Restore password store:
   ```bash
   # Decrypt the password store backup
   gpg -d password-store-backup.tar.gz.gpg > password-store-backup.tar.gz

   # Extract the backup
   tar -xzf password-store-backup.tar.gz -C $HOME
   ```

3. Verify the restoration:
   ```bash
   # List passwords
   pass

   # Try accessing a password
   pass show Email/example
   ```

## User Management for Shared Systems

If you're sharing your password management system with trusted individuals, proper user management is essential.

### Setting Up a Shared Password Store

Create a separate password store for shared passwords:

```bash
# Create a directory for the shared store
mkdir -p ~/shared-password-store

# Initialize with multiple GPG keys
PASSWORD_STORE_DIR=~/shared-password-store pass init YOUR_KEY_ID TEAM_MEMBER_1_KEY_ID TEAM_MEMBER_2_KEY_ID
```

### Managing Access

When team members change, update the encryption keys:

```bash
# Add a new team member
PASSWORD_STORE_DIR=~/shared-password-store pass init YOUR_KEY_ID EXISTING_MEMBER_KEY_ID NEW_MEMBER_KEY_ID

# Remove a team member
PASSWORD_STORE_DIR=~/shared-password-store pass init YOUR_KEY_ID REMAINING_MEMBER_KEY_ID
```

Each command re-encrypts all passwords with the specified keys.

### Using the Shared Store

Create an alias for the shared store:

```bash
echo 'alias team-pass="PASSWORD_STORE_DIR=~/shared-password-store pass"' >> ~/.bashrc
source ~/.bashrc
```

Now you can use `team-pass` to interact with the shared store:

```bash
# Add a shared password
team-pass insert Servers/production

# View a shared password
team-pass show Servers/production
```

### Access Control Policies

Establish clear policies for the shared password store:

1. Document who has access
2. Define procedures for adding/removing users
3. Establish password rotation schedules
4. Create guidelines for what passwords should be stored
5. Set up audit procedures

## Auditing and Compliance

Regular auditing helps ensure the security and integrity of your password management system.

### Password Audit

Periodically audit your passwords:

```bash
# Create a password audit script
nano ~/password-audit.sh
```

Add the following content:

```bash
#!/bin/bash
# Password audit script

echo "=== Password Store Audit ==="
echo "Date: $(date)"
echo

echo "=== Password Count ==="
pass | grep -v "Password Store" | wc -l
echo

echo "=== Password Categories ==="
pass | grep -v "Password Store" | grep "└──\|├──" | sed 's/└── \|├── //'
echo

echo "=== Recently Modified Passwords ==="
find ~/.password-store -name "*.gpg" -type f -mtime -30 | sed 's/.*\.password-store\///; s/\.gpg$//'
echo

echo "=== Passwords Not Modified in Over a Year ==="
find ~/.password-store -name "*.gpg" -type f -mtime +365 | sed 's/.*\.password-store\///; s/\.gpg$//'
echo

echo "=== Git Repository Status ==="
cd ~/.password-store && git status
echo

echo "=== GPG Key Information ==="
gpg --list-secret-keys --keyid-format LONG
echo

echo "=== Audit Complete ==="
```

Make the script executable:

```bash
chmod +x ~/password-audit.sh
```

Run the audit:

```bash
~/password-audit.sh > password-audit-$(date +"%Y%m%d").txt
```

Review the audit report for:
- Outdated passwords that need rotation
- Unused passwords that can be removed
- Missing categories or organizational issues
- GPG key expiration dates

### Security Compliance

If your password management system needs to comply with specific security standards (e.g., GDPR, HIPAA, SOC2), document how your implementation meets these requirements:

1. Encryption standards (GPG with 4096-bit keys)
2. Access controls (GPG key management)
3. Audit trails (Git history)
4. Backup procedures
5. Key rotation policies

Create a compliance document that maps your practices to specific requirements.

### Activity Logging

Set up logging for sensitive password operations:

```bash
# Create a post-show hook
mkdir -p ~/.password-store/.extensions
nano ~/.password-store/.extensions/post-show.bash
```

Add the following content:

```bash
#!/bin/bash
# Log password access

PASSWORD_NAME="$1"
LOG_FILE="$HOME/.password-access.log"

echo "$(date '+%Y-%m-%d %H:%M:%S') - Password accessed: $PASSWORD_NAME" >> "$LOG_FILE"
```

Make the hook executable:

```bash
chmod +x ~/.password-store/.extensions/post-show.bash
```

Enable extensions in your `.bashrc` or `.zshrc`:

```bash
export PASSWORD_STORE_ENABLE_EXTENSIONS=true
```

Periodically review the log file:

```bash
tail -n 50 ~/.password-access.log
```

Consider rotating log files to prevent them from growing too large.

## Long-Term Maintenance Checklist

Create a maintenance schedule with these regular tasks:

### Monthly Tasks

- Pull and push changes on all devices
- Check for uncommitted changes
- Review recent password additions and changes
- Verify synchronization across devices

### Quarterly Tasks

- Run a password audit
- Identify and rotate weak or old passwords
- Clean up unused passwords
- Check GPG key expiration dates
- Test backups with a recovery drill

### Annual Tasks

- Rotate GPG keys
- Review and update access for shared password stores
- Perform a comprehensive security review
- Update documentation and procedures
- Clean and optimize Git repositories

## Conclusion

Proper maintenance and administration of your password management system ensures its long-term security, reliability, and usability. By establishing regular procedures for key rotation, repository maintenance, backups, user management, and auditing, you create a robust system that will serve you well for years to come.

Key takeaways:
- Regularly rotate your GPG keys to limit the impact of potential compromises
- Maintain your Git repository to keep it efficient and reliable
- Implement comprehensive backup and recovery procedures
- Carefully manage access for shared password stores
- Conduct regular audits to ensure security and compliance

With these practices in place, your cross-platform password management system will remain a secure and effective tool for managing your digital credentials across all your devices and platforms.

## Navigation

- [README](README.md) - Wiki Home
- Previous: [Extending to Additional Platforms](14_Extending_to_Additional_Platforms.md)
- Next: [Real-Time Password Synchronization](16_Real_Time_Password_Synchronization.md)
- Related:
  - [Security Best Practices](10_Security_Best_Practices.md) - For security considerations
  - [Advanced Features and Extensions](12_Advanced_Features_and_Extensions.md) - For backup and extension details
  - [Troubleshooting Guide](13_Troubleshooting_Guide.md) - For help with maintenance issues
