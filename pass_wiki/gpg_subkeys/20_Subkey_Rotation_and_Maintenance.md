# Subkey Rotation and Maintenance

## Introduction

GPG subkeys should be rotated periodically to maintain security. This guide will walk you through the process of planning for subkey expiration, creating new subkeys, distributing them to your devices, and handling potential subkey compromise.

Regular maintenance of your GPG subkeys is an important security practice that helps limit the damage if a key is compromised and ensures your encryption remains strong over time.

## Planning for Subkey Expiration

When you created your subkeys, you likely set an expiration date (we recommended 1 year in the previous guides). As this date approaches, you'll need to plan for rotation.

### Setting Up Calendar Reminders

It's a good idea to set reminders well before your subkeys expire:

```bash
# Check when your subkeys expire
gpg --list-secret-keys --keyid-format LONG
```

Look for the expiration dates in the output, which will look something like `[expires: 2024-06-01]`.

Set calendar reminders for:
- 1 month before expiration: Start planning the rotation
- 2 weeks before expiration: Perform the rotation
- 1 week before expiration: Verify all systems are updated

### Preparing for Rotation in Advance

Before your subkeys expire, make sure you:

1. Have access to your master key (which you'll need to create new subkeys)
2. Have a current backup of your entire GPG keyring
3. Have documented which devices have your current subkeys
4. Have time set aside for the rotation process (about 1 hour)

## Creating New Subkeys

When it's time to rotate your subkeys, you'll need to access your master key, which you may have stored offline for security.

### Accessing Your Master Key Securely

If your master key is stored on an offline medium (like an encrypted USB drive):

```bash
# Mount your encrypted storage
# This will vary depending on your setup

# If your master key is in a backup archive
gpg -d gpg-full-backup.tar.gz.gpg > gpg-full-backup.tar.gz
tar -xzf gpg-full-backup.tar.gz -C ~/
```

If you're using a completely separate offline system for key management, you'll need to perform the following steps on that system.

### Generating Replacement Subkeys

Once you have access to your master key, you can create new subkeys:

```bash
# Edit your key
gpg --expert --edit-key YOUR_KEY_ID
```

Replace `YOUR_KEY_ID` with your actual GPG key ID.

At the GPG prompt, add a new encryption subkey:

```
gpg> addkey
```

Choose RSA (encrypt only) and follow the prompts as you did when creating your original subkeys. Set an appropriate expiration date (1 year is recommended).

Repeat the process to create a new signing subkey if needed.

When finished, save your changes:

```
gpg> save
```

### Setting Appropriate Expiration Dates

When setting expiration dates for your new subkeys:

1. **Stagger expiration dates**: If you have multiple subkeys, consider setting slightly different expiration dates (e.g., 11 months, 12 months, 13 months) to avoid having to rotate all keys at once.

2. **Consider your usage patterns**: If you travel frequently or have periods when key rotation would be difficult, plan around these.

3. **Balance security and convenience**: Shorter expiration periods (e.g., 6 months) provide better security but require more frequent rotation.

## Distributing Updated Subkeys

After creating new subkeys, you need to distribute them to all your devices.

### Exporting New Subkeys

Export your updated subkeys following the same process as before:

```bash
# Export your public key
gpg --armor --export YOUR_KEY_ID > public-key-new.asc

# Export your secret subkeys
gpg --armor --export-secret-subkeys YOUR_KEY_ID > secret-subkeys-new.asc

# Export your trust database
gpg --export-ownertrust > trust-new.txt

# Create and encrypt an archive
mkdir -p ~/subkey-export-new
chmod 700 ~/subkey-export-new
mv public-key-new.asc secret-subkeys-new.asc trust-new.txt ~/subkey-export-new/
tar -czf ~/subkey-export-new.tar.gz -C ~ subkey-export-new
gpg -c ~/subkey-export-new.tar.gz
```

### Importing on Secondary Devices

On each of your secondary devices:

```bash
# Decrypt the archive
gpg -d subkey-export-new.tar.gz.gpg > subkey-export-new.tar.gz

# Extract the files
tar -xzf subkey-export-new.tar.gz -C ~/

# Import the keys
gpg --import ~/subkey-export-new/public-key-new.asc
gpg --import ~/subkey-export-new/secret-subkeys-new.asc
gpg --import-ownertrust ~/subkey-export-new/trust-new.txt
```

### Verifying Successful Updates

After importing the new subkeys on each device, verify that they're working correctly:

```bash
# Check that the new subkeys are available
gpg --list-secret-keys --keyid-format LONG

# Test encryption with the new subkey
echo "test" | gpg -e -r YOUR_KEY_ID | gpg -d

# If using Git, test signing
echo "test" | gpg --sign -u YOUR_SIGNING_SUBKEY_ID
```

Also test that `pass` still works correctly:

```bash
# Test creating a new password
pass generate test/rotation-test 15

# Test viewing a password
pass test/rotation-test
```

## Handling Subkey Compromise

If you suspect one of your subkeys has been compromised (e.g., a device with the subkey was stolen), you need to take immediate action.

### Identifying Potential Compromise

Signs that might indicate a subkey compromise:

- A device with your subkeys has been lost or stolen
- Unauthorized access to your password store
- Unexpected changes to your passwords
- Suspicious Git commits in your password repository

### Emergency Revocation Procedures

If you believe a subkey has been compromised:

1. **Access your master key** (which should be stored securely offline)

2. **Revoke the compromised subkey**:
   ```bash
   # Edit your key
   gpg --edit-key YOUR_KEY_ID
   
   # Select the compromised subkey (replace N with the number of the subkey)
   gpg> key N
   
   # Revoke the subkey
   gpg> revkey
   
   # Confirm the revocation
   # Save your changes
   gpg> save
   ```

3. **Create a new subkey** to replace the revoked one (follow the steps in "Generating Replacement Subkeys" above)

4. **Export your updated public key**:
   ```bash
   gpg --armor --export YOUR_KEY_ID > public-key-revoked.asc
   ```

5. **Distribute the updated public key** to all your devices and any key servers you use:
   ```bash
   # On each device
   gpg --import public-key-revoked.asc
   
   # If you use key servers
   gpg --keyserver hkps://keys.openpgp.org --send-keys YOUR_KEY_ID
   ```

### Recovery Steps

After revoking a compromised subkey:

1. **Change sensitive passwords** that might have been exposed

2. **Review your password store** for any unauthorized changes:
   ```bash
   # Check Git history
   pass git log
   ```

3. **Export and distribute new subkeys** to replace the revoked ones (follow the steps in "Exporting New Subkeys" above)

4. **Update your documentation** to reflect the revocation and new subkeys

## Long-term Maintenance Strategy

To maintain your GPG subkey setup over the long term, establish a regular maintenance routine.

### Documentation of Key History

Keep a secure record of your key management activities:

```bash
# Create or update your key management log
nano ~/key-management-log.txt

# Example content:
# 2023-06-01: Created initial subkeys (IDs: 2B1A8H7G6F5E4D3C, 6F5E4D3C2B1A8H7G)
# 2024-05-15: Rotated subkeys, created new ones (IDs: 3C2B1A8H7G6F5E4D, 7G6F5E4D3C2B1A8H)
# 2024-05-16: Distributed new subkeys to laptop and work computer

# Encrypt the log
gpg -c ~/key-management-log.txt
shred -u ~/key-management-log.txt  # Securely delete the unencrypted file
```

### Backup Verification Procedures

Regularly verify that your backups are accessible and functional:

```bash
# Create a temporary directory for testing
mkdir -p ~/backup-test
chmod 700 ~/backup-test
cd ~/backup-test

# Decrypt your backup
gpg -d ~/path/to/gpg-full-backup.tar.gz.gpg > gpg-test.tar.gz

# Extract a small portion to verify integrity
tar -tzf gpg-test.tar.gz | head -5

# Clean up
cd ~
rm -rf ~/backup-test
```

Perform this verification at least once a year or whenever you make significant changes to your key setup.

### Security Audit Checklist

Periodically perform a security audit of your GPG and `pass` setup:

```bash
# Check for expired or soon-to-expire keys
gpg --list-secret-keys --keyid-format LONG

# Verify key permissions
ls -la ~/.gnupg/

# Check for unauthorized Git commits
cd ~/.password-store && git log --author="not-your-name"

# Verify that your master key is not present on everyday devices
# This should show sec# (with the #) for your master key
gpg --list-secret-keys --keyid-format LONG

# Check that pass is using the correct key
pass grep -n ""
```

Create a reminder to perform this audit every 3-6 months.

## Conclusion

Regular rotation and maintenance of your GPG subkeys is essential for maintaining the security of your `pass` password management system. By following the procedures in this guide, you can:

- Ensure your subkeys are rotated before they expire
- Respond effectively to potential key compromise
- Maintain good documentation of your key history
- Verify the integrity of your backups
- Regularly audit your security setup

With these practices in place, your password management system will remain secure and functional over the long term.

In the next guide, we'll cover troubleshooting common issues that can arise when using GPG subkeys with `pass`.

## Navigation

- [README](../README.md) - Wiki Home
- Previous: [Setting Up Pass with GPG Subkeys](19_Setting_Up_Pass_with_GPG_Subkeys.md)
- Next: [Troubleshooting GPG Subkeys with Pass](21_Troubleshooting_GPG_Subkeys_with_Pass.md)
- Related: [Security Best Practices](../10_Security_Best_Practices.md)
