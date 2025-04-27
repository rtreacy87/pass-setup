# Security Best Practices

## Introduction

Your password management system is only as secure as your practices around it. This guide covers essential security best practices for maintaining the integrity and confidentiality of your Pass password store across multiple systems.

## Key Management Security

Your GPG keys are the foundation of your password management system's security. Protecting them is critical.

### Passphrase Protection

Always use a strong passphrase for your GPG key:

- **Length**: At least 12 characters, preferably longer
- **Complexity**: Mix of uppercase, lowercase, numbers, and special characters
- **Memorability**: Consider using a passphrase (multiple words) rather than a simple password
- **Uniqueness**: Don't reuse passphrases from other services

Remember that your passphrase is the last line of defense if someone obtains your private key.

### Key Backup

Create secure backups of your GPG keys:

```bash
# Export your public key
gpg --armor --export YOUR_GPG_KEY_ID > public-key-backup.asc

# Export your private key
gpg --armor --export-secret-key YOUR_GPG_KEY_ID > private-key-backup.asc

# Export your trust database
gpg --export-ownertrust > trust-backup.txt

# Create an encrypted archive of the backups
tar -czf gpg-backup.tar.gz public-key-backup.asc private-key-backup.asc trust-backup.txt
gpg -c gpg-backup.tar.gz

# Securely delete the unencrypted files
shred -u public-key-backup.asc private-key-backup.asc trust-backup.txt gpg-backup.tar.gz
```

Store the encrypted backup (`gpg-backup.tar.gz.gpg`) in a secure location, such as:
- An encrypted external drive kept in a safe place
- A secure cloud storage service with strong encryption
- A password-protected USB drive stored securely

### Key Rotation

Periodically rotate your GPG keys to limit the impact of potential compromises:

1. Generate a new GPG key pair:
   ```bash
   gpg --full-generate-key
   ```

2. Re-encrypt your password store with the new key:
   ```bash
   # Initialize pass with both old and new keys
   pass init OLD_KEY_ID NEW_KEY_ID
   
   # Remove the old key
   pass init NEW_KEY_ID
   ```

3. Update the key on all your devices

Consider rotating your keys annually or bi-annually.

### Secure Key Transfer

When transferring keys between systems:

- **Never** send private keys via email or unencrypted messaging
- **Always** encrypt private keys before transfer
- Use secure channels like encrypted USB drives or secure file transfer protocols
- Delete transferred key files after successful import

## Repository Security

Your password repository contains all your encrypted passwords, so it needs proper protection.

### Private Repository

If using a Git hosting service:

- **Always** use a private repository, never public
- Enable two-factor authentication on your Git hosting account
- Use SSH keys for authentication rather than passwords
- Regularly review access permissions to ensure only you have access

### No Plaintext

Never commit plaintext (unencrypted) passwords or sensitive information:

- Check files before committing to ensure they're encrypted (`.gpg` extension)
- If you accidentally commit plaintext, remove it immediately and purge it from Git history:
  ```bash
  pass git filter-branch --force --index-filter \
  "git rm --cached --ignore-unmatch path/to/plaintext/file" \
  --prune-empty --tag-name-filter cat -- --all
  ```

### Access Control

Limit access to your Git repository:

- Only grant access to trusted individuals if sharing is necessary
- Use separate repositories for personal and shared passwords
- Regularly audit who has access to your repository
- Revoke access immediately when no longer needed

## System Security

The security of your password management system depends on the security of the systems where it's installed.

### Disk Encryption

Enable full-disk encryption on all systems where you use Pass:

- **macOS**: Use FileVault (System Preferences → Security & Privacy → FileVault)
- **Windows**: Use BitLocker or VeraCrypt
- **WSL Ubuntu**: Your Windows disk encryption protects WSL files

Full-disk encryption ensures that if your device is lost or stolen, your passwords remain protected.

### Screen Lock

Configure automatic screen locking on all your devices:

- **macOS**: System Preferences → Security & Privacy → General → Require password after sleep or screen saver
- **Windows**: Settings → Accounts → Sign-in options → Require sign-in

Set a short timeout (e.g., 5 minutes) to minimize the risk if you step away from your computer.

### Update Regularly

Keep all software components updated:

```bash
# On macOS
brew update && brew upgrade

# On WSL Ubuntu
sudo apt update && sudo apt upgrade -y

# Update GPG
gpg --version  # Check current version
# Update via package manager if needed
```

Regular updates ensure you have the latest security patches.

## Password Store Security

Beyond the technical aspects, consider these practices for your password store itself.

### Strong Generated Passwords

When generating passwords with Pass, use sufficient length and complexity:

```bash
# Generate a 20-character password with symbols
pass generate -n Website/example 20
```

The `-n` flag includes symbols in the generated password.

For critical accounts, consider using even longer passwords (25+ characters).

### Regular Password Rotation

Periodically update important passwords:

```bash
# Generate a new password for an important account
pass generate -f Bank/important-account 20
```

The `-f` flag forces overwriting the existing password.

Consider setting up a schedule to rotate passwords for critical accounts every 3-6 months.

### Password Organization

Organize your passwords in a logical structure:

```
Password Store
├── Email
│   ├── personal
│   └── work
├── Finance
│   ├── banking
│   └── investments
├── Social
│   ├── facebook
│   └── twitter
└── Work
    ├── vpn
    └── wiki
```

Good organization makes it easier to manage and audit your passwords.

### Metadata Management

Be careful about what metadata you store with your passwords:

- **Do store**: Usernames, URLs, security questions (encrypted)
- **Don't store**: Unencrypted notes, personal identifying information

Example of good metadata in a password entry:
```
MySecurePassword123
username: user@example.com
url: https://example.com
recovery-email: backup@example.com
```

## Security Audit Procedures

Regularly audit your password management system to ensure it remains secure.

### Password Audit

Periodically review your passwords:

```bash
# List all passwords
pass

# Check a specific password's strength
pass Email/gmail | pwscore
```

Look for:
- Duplicate passwords
- Weak passwords
- Outdated passwords
- Passwords for services you no longer use

### GPG Key Audit

Verify your GPG key status:

```bash
# Check key expiration
gpg --list-secret-keys --keyid-format LONG

# Verify key integrity
gpg --check-sigs YOUR_GPG_KEY_ID
```

Ensure your keys haven't expired and are still secure.

### Git Repository Audit

Check your Git repository for issues:

```bash
# Navigate to your password store
cd ~/.password-store

# Check for uncommitted changes
git status

# Review recent commits
git log --name-status -10

# Check remote repository configuration
git remote -v
```

Verify that all changes are committed and pushed, and that your remote repository is correctly configured.

## Emergency Procedures

Prepare for potential security incidents.

### If Your Device is Lost or Stolen

1. If possible, remotely wipe the device
2. Change passwords for critical accounts from another device
3. Consider rotating your GPG key
4. Monitor accounts for suspicious activity

### If Your GPG Key is Compromised

1. Generate a new GPG key immediately
2. Re-encrypt your password store with the new key
3. Revoke the old key:
   ```bash
   gpg --gen-revoke OLD_KEY_ID > revocation.asc
   gpg --import revocation.asc
   ```
4. Update the key on all your devices
5. Change passwords for critical accounts

### If Your Git Repository is Compromised

1. Change your Git hosting service password and enable two-factor authentication
2. Rotate your GPG key
3. Re-encrypt your password store
4. Create a new private repository
5. Change passwords for critical accounts

## Conclusion

By following these security best practices, you'll significantly enhance the security of your cross-platform password management system. Remember that security is an ongoing process, not a one-time setup.

Key takeaways:
- Protect your GPG keys with strong passphrases and secure backups
- Keep your password repository private and properly controlled
- Secure all systems where you use Pass
- Organize and regularly audit your passwords
- Be prepared for potential security incidents

In the next guide, we'll explore GUI clients and browser integration options to make your password management system more convenient while maintaining security.
