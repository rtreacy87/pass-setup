# Troubleshooting GPG Subkeys with Pass

## Introduction

When using GPG subkeys with `pass`, you might encounter various issues related to key configuration, expiration, or permissions. This guide will help you diagnose and fix common problems that can occur when using GPG subkeys with the `pass` password manager.

We'll cover how to identify issues, diagnose their causes, and apply the appropriate fixes, all using command-line tools with clear explanations.

## Common Subkey Issues

### Issue 1: "gpg: decryption failed: No secret key"

This error occurs when trying to decrypt a password, but the corresponding encryption subkey is not available or not properly configured.

#### Diagnosis

Check if your encryption subkey is properly imported:

```bash
gpg --list-secret-keys --keyid-format LONG
```

Look for your key and check if there's an encryption subkey (marked with `[E]`) listed. If it's missing or has expired, that's the source of the problem.

#### Solutions

1. **If the subkey is missing**, import it:
   ```bash
   gpg --import path/to/secret-subkeys.asc
   ```

2. **If the subkey has expired**, you need to extend its expiration date using your master key (which requires access to your offline master key) or create a new subkey.

3. **If the subkey is present but not working**, try refreshing your GPG agent:
   ```bash
   gpgconf --kill gpg-agent
   ```

### Issue 2: "error: gpg failed to sign the data"

This error occurs when trying to sign Git commits but your signing subkey is not properly configured.

#### Diagnosis

Verify your signing subkey:

```bash
gpg --list-secret-keys --keyid-format LONG
```

Look for a signing subkey (marked with `[S]`). Also check your Git configuration:

```bash
pass git config user.signingkey
```

#### Solutions

1. **If the signing subkey is missing**, import it:
   ```bash
   gpg --import path/to/secret-subkeys.asc
   ```

2. **If Git is configured with the wrong key ID**, update it:
   ```bash
   pass git config user.signingkey YOUR_SIGNING_SUBKEY_ID
   ```

3. **If the signing subkey has expired**, extend its expiration or create a new one.

### Issue 3: "gpg: WARNING: unsafe ownership on homedir"

This warning indicates that your GPG home directory has incorrect permissions.

#### Diagnosis

Check the permissions on your GPG directory:

```bash
ls -la ~/.gnupg
```

The directory should be owned by your user and have permissions set to 700 (rwx------).

#### Solution

Fix the permissions:

```bash
chmod 700 ~/.gnupg
chmod 600 ~/.gnupg/*
```

### Issue 4: "gpg: WARNING: unsafe permissions on homedir"

Similar to the previous issue, this warning indicates permission problems with your GPG configuration.

#### Solution

Fix the permissions and ensure the directory is owned by your user:

```bash
chown -R $(whoami) ~/.gnupg
chmod 700 ~/.gnupg
chmod 600 ~/.gnupg/*
```

## Diagnosing Subkey Problems

When troubleshooting issues with GPG subkeys, it's helpful to have a systematic approach to diagnosis.

### Checking Key Status and Capabilities

To get detailed information about your keys:

```bash
# List all keys with capabilities
gpg --list-keys --keyid-format LONG --with-colons

# Check a specific key in detail
gpg --edit-key YOUR_KEY_ID
```

In the GPG edit mode, you can use these commands:
```
gpg> check
gpg> list
gpg> quit
```

### Verifying Key Associations

To check which keys are associated with your email:

```bash
gpg --list-keys your.email@example.com
```

To check which keys `pass` is using:

```bash
cat ~/.password-store/.gpg-id
```

### Testing Encryption/Decryption

To test if your encryption subkey is working properly:

```bash
# Create a test file
echo "This is a test" > test.txt

# Encrypt it
gpg --encrypt --recipient YOUR_KEY_ID test.txt

# Try to decrypt it
gpg --decrypt test.txt.gpg
```

If decryption works, your encryption subkey is functioning correctly.

### Testing Signing

To test if your signing subkey is working properly:

```bash
# Sign a test file
echo "This is a test" > test.txt
gpg --sign test.txt

# Verify the signature
gpg --verify test.txt.gpg
```

If verification succeeds, your signing subkey is functioning correctly.

## Fixing Subkey Configuration Issues

### Correcting Trust Settings

If you're having trust-related issues:

```bash
# Edit your key
gpg --edit-key YOUR_KEY_ID

# Set ultimate trust for your own key
gpg> trust
# Select option 5 (I trust ultimately)
gpg> save
```

### Updating Expired Subkeys

If your subkeys have expired and you have access to your master key:

```bash
# Edit your key
gpg --edit-key YOUR_KEY_ID

# Select the expired subkey (replace N with the subkey number)
gpg> key N

# Change the expiration date
gpg> expire
# Follow the prompts to set a new expiration date

gpg> save
```

### Re-importing Keys Correctly

If you need to start fresh with your key import:

```bash
# First, make a backup of your current keyring
cp -r ~/.gnupg ~/.gnupg-backup

# Remove the problematic key
gpg --delete-secret-and-public-key YOUR_KEY_ID

# Import the public key
gpg --import public-key.asc

# Import the secret subkeys
gpg --import secret-subkeys.asc

# Import the trust database
gpg --import-ownertrust trust.txt
```

## Pass-Specific Troubleshooting

### Key ID Mismatch Issues

If `pass` is using a different key than expected:

```bash
# Check which key pass is using
cat ~/.password-store/.gpg-id

# Reinitialize pass with the correct key
pass init YOUR_KEY_ID
```

### Git Integration Problems

If you're having issues with Git integration:

```bash
# Check Git configuration
pass git config --list

# Ensure signing is properly configured
pass git config user.signingkey YOUR_SIGNING_SUBKEY_ID
pass git config commit.gpgsign true

# Test a commit
pass insert test/git-test
pass git add .
pass git commit -m "Test commit"
```

### Cross-Platform Compatibility

If you're having issues using your subkeys across different platforms:

```bash
# On Linux/macOS, check key format
gpg --list-secret-keys --keyid-format LONG

# Ensure consistent configuration across platforms
# Compare the output of:
gpg --version
```

Different GPG versions may handle subkeys differently. If possible, try to use similar versions across your devices.

## Recovery Procedures

### Restoring from Backups

If you need to restore your GPG setup from backups:

```bash
# Create a temporary directory
mkdir -p ~/gpg-restore
chmod 700 ~/gpg-restore
cd ~/gpg-restore

# Decrypt your backup
gpg -d ~/path/to/gpg-full-backup.tar.gz.gpg > gpg-full-backup.tar.gz

# Extract the backup
tar -xzf gpg-full-backup.tar.gz

# Import the keys
gpg --import */public-key*.asc
gpg --import */secret-key*.asc
gpg --import-ownertrust */trust*.txt

# Clean up
cd ~
rm -rf ~/gpg-restore
```

### Emergency Access Options

If you need emergency access to your passwords but are having issues with your subkeys:

1. **If you have access to your master key**:
   ```bash
   # Import your full key (including master key)
   gpg --import full-secret-key.asc
   
   # Try to access your passwords
   pass show important/password
   ```

2. **If you don't have access to any keys but have a backup of your password store**:
   You'll need to set up a new GPG key and re-encrypt your passwords. This is a complex process that should be done carefully.

### When to Regenerate Keys

Consider regenerating your GPG keys if:

- Your master key has been compromised
- You've lost access to your master key and all backups
- Your key is very old (5+ years) and uses outdated algorithms
- You want to switch to a stronger key size or algorithm

## Diagnostic Commands Reference

Here's a reference of useful diagnostic commands for troubleshooting GPG subkeys:

### GPG Key Information

```bash
# List all keys
gpg --list-keys --keyid-format LONG

# List secret keys
gpg --list-secret-keys --keyid-format LONG

# Show detailed key information
gpg --list-keys --keyid-format LONG --with-colons

# Check key fingerprints
gpg --fingerprint YOUR_KEY_ID

# Check signatures on your key
gpg --check-sigs YOUR_KEY_ID
```

### GPG Configuration

```bash
# Show GPG version
gpg --version

# Show GPG configuration
cat ~/.gnupg/gpg.conf

# Show GPG agent configuration
cat ~/.gnupg/gpg-agent.conf

# Check environment variables
env | grep GPG
```

### Pass Configuration

```bash
# Show pass version
pass --version

# Check which key pass is using
cat ~/.password-store/.gpg-id

# Check pass environment variables
env | grep PASSWORD_STORE
```

### Git Configuration

```bash
# Check Git configuration in pass store
cd ~/.password-store && git config --list

# Check global Git configuration
git config --global --list
```

## Common Error Messages and Solutions

Here's a quick reference for common error messages and their solutions:

| Error Message | Likely Cause | Solution |
|---------------|--------------|----------|
| `gpg: decryption failed: No secret key` | Missing encryption subkey | Import your secret subkeys |
| `error: gpg failed to sign the data` | Missing or misconfigured signing subkey | Import signing subkey or fix Git config |
| `gpg: WARNING: unsafe ownership on homedir` | Incorrect permissions | Fix permissions with `chmod 700 ~/.gnupg` |
| `gpg: keyblock resource '/home/user/.gnupg/pubring.kbx': resource limit` | Corrupted keyring | Rebuild keyring from backups |
| `gpg-agent: command get_passphrase failed: No pinentry` | Missing pinentry program | Install pinentry (`apt install pinentry-curses`) |
| `pass: gpg: signing failed: Inappropriate ioctl for device` | GPG can't request passphrase | Set `export GPG_TTY=$(tty)` |

## Conclusion

Troubleshooting GPG subkeys with `pass` can be complex, but with the systematic approach outlined in this guide, you should be able to diagnose and fix most common issues. Remember to:

1. **Back up your keys** before making any changes
2. **Check permissions** as they're a common source of problems
3. **Verify key expiration dates** regularly
4. **Test encryption and signing** to confirm functionality
5. **Keep your GPG software updated** for best compatibility and security

By maintaining your GPG subkeys properly and knowing how to troubleshoot issues, you can ensure your `pass` password management system remains secure and functional over time.

## Navigation

- [README](../README.md) - Wiki Home
- Previous: [Subkey Rotation and Maintenance](20_Subkey_Rotation_and_Maintenance.md)
- Related: [Troubleshooting Guide](../13_Troubleshooting_Guide.md)
