# Command Reference for GPG Key Management

This appendix provides a comprehensive reference of GPG commands for key management, organized by task. Each command includes an explanation of what it does and how to use it.

## Key Generation and Management

### Generating Keys

```bash
# Generate a new key pair with interactive prompts
gpg --full-generate-key

# Generate a key non-interactively (batch mode)
gpg --batch --generate-key key_config.txt
```

Where `key_config.txt` contains:
```
Key-Type: RSA
Key-Length: 4096
Subkey-Type: RSA
Subkey-Length: 4096
Name-Real: Your Name
Name-Email: your.email@example.com
Expire-Date: 2y
Passphrase: your-secure-passphrase
```

### Listing Keys

```bash
# List all public keys in your keyring
gpg --list-keys

# List all private keys in your keyring
gpg --list-secret-keys

# List keys with fingerprints
gpg --fingerprint

# List keys with long key IDs
gpg --list-keys --keyid-format LONG

# List keys with signatures
gpg --list-sigs
```

### Editing Keys

```bash
# Edit a key (interactive)
gpg --edit-key KEY_ID

# Common commands at the gpg> prompt:
# adduid - Add a user ID
# deluid - Delete a user ID
# addkey - Add a subkey
# delkey - Delete a subkey
# expire - Change expiration date
# passwd - Change passphrase
# trust - Change trust level
# save - Save changes and exit
# quit - Exit without saving
```

### Changing Key Expiration

```bash
# Change expiration date (interactive)
gpg --edit-key KEY_ID
# At the gpg> prompt:
expire
# Follow prompts to set new expiration
save
```

### Changing Key Passphrase

```bash
# Change the passphrase for a key
gpg --passwd KEY_ID
```

## Key Export and Import

### Exporting Keys

```bash
# Export public key in ASCII format
gpg --export --armor KEY_ID > public_key.asc

# Export private key in ASCII format
gpg --export-secret-keys --armor KEY_ID > private_key.asc

# Export specific subkey
gpg --export-secret-subkeys --armor SUBKEY_ID! > subkey.asc

# Export all keys
gpg --export --armor > all_public_keys.asc
gpg --export-secret-keys --armor > all_private_keys.asc

# Export trust database
gpg --export-ownertrust > trust.txt
```

### Importing Keys

```bash
# Import a public key
gpg --import public_key.asc

# Import a private key
gpg --import private_key.asc

# Import trust database
gpg --import-ownertrust trust.txt

# Import from keyserver
gpg --keyserver hkps://keys.openpgp.org --recv-keys KEY_ID
```

### Refreshing Keys

```bash
# Refresh keys from keyserver
gpg --keyserver hkps://keys.openpgp.org --refresh-keys
```

## Key Verification and Trust

### Verifying Key Fingerprints

```bash
# Display key fingerprint
gpg --fingerprint KEY_ID

# Display key fingerprint in machine-readable format
gpg --with-colons --fingerprint KEY_ID
```

### Managing Trust

```bash
# Set trust level (interactive)
gpg --edit-key KEY_ID
# At the gpg> prompt:
trust
# Enter trust level (1-5)
save

# List trust database
gpg --export-ownertrust
```

### Signing Keys

```bash
# Sign someone's key (certify it's authentic)
gpg --sign-key KEY_ID

# Sign with a specific key
gpg --default-key YOUR_KEY_ID --sign-key THEIR_KEY_ID
```

## Key Revocation and Deletion

### Creating Revocation Certificates

```bash
# Generate a revocation certificate
gpg --gen-revoke KEY_ID > revocation_certificate.asc

# Import a revocation certificate
gpg --import revocation_certificate.asc
```

### Revoking Keys

```bash
# Revoke a key (after importing revocation certificate)
gpg --keyserver hkps://keys.openpgp.org --send-keys KEY_ID
```

### Deleting Keys

```bash
# Delete a public key from your keyring
gpg --delete-key KEY_ID

# Delete a private key from your keyring
gpg --delete-secret-key KEY_ID

# Delete both public and private keys
gpg --delete-secret-and-public-key KEY_ID
```

## Encryption and Decryption

### Encrypting Files

```bash
# Encrypt with recipient's public key
gpg --encrypt --recipient RECIPIENT_EMAIL file.txt

# Encrypt for multiple recipients
gpg --encrypt --recipient RECIPIENT1 --recipient RECIPIENT2 file.txt

# Encrypt and sign
gpg --encrypt --sign --recipient RECIPIENT_EMAIL file.txt

# Symmetric encryption (password-based)
gpg --symmetric --cipher-algo AES256 file.txt
```

### Decrypting Files

```bash
# Decrypt a file
gpg --decrypt file.txt.gpg > decrypted_file.txt

# Decrypt and verify signature
gpg --decrypt signed_encrypted_file.gpg > decrypted_file.txt
```

## Signing and Verification

### Signing Files

```bash
# Create a detached signature
gpg --detach-sign file.txt

# Create a detached signature in ASCII format
gpg --detach-sign --armor file.txt

# Sign a file (includes content)
gpg --sign file.txt

# Clear-sign a text file (readable but verified)
gpg --clearsign document.txt
```

### Verifying Signatures

```bash
# Verify a detached signature
gpg --verify file.txt.sig file.txt

# Verify a signed file
gpg --verify file.txt.gpg

# Verify and extract content
gpg --decrypt file.txt.gpg > original_file.txt
```

## GPG Agent Management

### Starting and Stopping the Agent

```bash
# Start the GPG agent
gpg-agent --daemon

# Kill the GPG agent
gpgconf --kill gpg-agent
```

### Configuring the Agent

```bash
# Edit agent configuration
nano ~/.gnupg/gpg-agent.conf

# Common settings:
# default-cache-ttl 3600
# max-cache-ttl 14400
# pinentry-program /usr/bin/pinentry-curses
```

### Testing the Agent

```bash
# Check if agent is running
gpg-connect-agent /bye

# Clear cached passphrases
echo RELOADAGENT | gpg-connect-agent
```

## Troubleshooting and Maintenance

### Checking GPG Version and Configuration

```bash
# Check GPG version
gpg --version

# Show configuration file locations
gpgconf --list-dirs

# Show all configuration options
gpgconf --list-options gpg
```

### Fixing Permissions

```bash
# Fix common permission issues
chmod 700 ~/.gnupg
chmod 600 ~/.gnupg/*
chmod 700 ~/.gnupg/private-keys-v1.d
```

### Debugging

```bash
# Run GPG with verbose output
gpg --verbose --decrypt file.gpg

# Run with even more verbose output
gpg --verbose --verbose --decrypt file.gpg

# Show packet information
gpg --list-packets file.gpg
```

### Checking Key Health

```bash
# Check for expired keys
gpg --list-keys --with-colons | grep "^pub" | grep "e$"

# Check for keys without expiration
gpg --list-keys --with-colons | grep "^pub" | grep ":0:$"
```

## Git Integration

### Configuring Git with GPG

```bash
# Set signing key
git config --global user.signingkey KEY_ID

# Enable commit signing by default
git config --global commit.gpgsign true

# Enable tag signing by default
git config --global tag.gpgsign true

# Specify GPG program path
git config --global gpg.program $(which gpg)
```

### Signing Git Operations

```bash
# Sign a commit
git commit -S -m "Signed commit message"

# Sign a tag
git tag -s v1.0 -m "Signed tag"

# Verify a signed commit
git verify-commit COMMIT_HASH

# Verify a signed tag
git verify-tag TAG_NAME
```

## Advanced Operations

### Working with Subkeys

```bash
# Add a new subkey (interactive)
gpg --expert --edit-key KEY_ID
# At the gpg> prompt:
addkey
# Follow prompts to create subkey
save

# Export only subkeys
gpg --export-secret-subkeys --armor KEY_ID > subkeys.asc

# List subkeys
gpg --list-keys --with-subkey-fingerprints KEY_ID
```

### Key Backup and Restoration

```bash
# Backup entire GPG directory
tar -czf gnupg-backup.tar.gz -C ~ .gnupg

# Restore from backup
tar -xzf gnupg-backup.tar.gz -C ~
chmod 700 ~/.gnupg
chmod 600 ~/.gnupg/*
```

### Paper Backup

```bash
# Create a paper-friendly backup (requires paperkey)
gpg --export-secret-keys KEY_ID | paperkey --output paper-backup.txt

# Restore from paper backup
paperkey --pubring ~/.gnupg/pubring.gpg --secrets paper-backup.txt --output restored_key.gpg
gpg --import restored_key.gpg
```

### Smart Card Operations

```bash
# Check if card is detected
gpg --card-status

# Edit card settings
gpg --card-edit

# Move key to card
gpg --edit-key KEY_ID
# At the gpg> prompt:
keytocard
# Follow prompts to move key to card
save
```

## Common Command Combinations

### Complete Key Transfer Process

```bash
# Export keys
gpg --export --armor KEY_ID > public_key.asc
gpg --export-secret-keys --armor KEY_ID > private_key.asc
gpg --export-ownertrust > trust.txt

# Create and encrypt archive
tar -czf gpg_keys.tar.gz public_key.asc private_key.asc trust.txt
gpg --symmetric --cipher-algo AES256 gpg_keys.tar.gz

# Securely delete unencrypted files
shred -u public_key.asc private_key.asc trust.txt gpg_keys.tar.gz

# On destination system:
gpg --decrypt gpg_keys.tar.gz.gpg > gpg_keys.tar.gz
tar -xzf gpg_keys.tar.gz
gpg --import public_key.asc
gpg --import private_key.asc
gpg --import-ownertrust trust.txt
shred -u public_key.asc private_key.asc trust.txt gpg_keys.tar.gz
```

### Key Rotation Process

```bash
# Generate new key
gpg --full-generate-key

# Sign new key with old key
gpg --default-key OLD_KEY_ID --sign-key NEW_KEY_ID

# Export new public key
gpg --export --armor NEW_KEY_ID > new_public_key.asc

# Create revocation certificate for old key
gpg --gen-revoke OLD_KEY_ID > old_key_revocation.asc

# When ready, revoke old key
gpg --import old_key_revocation.asc
```

## Environment Variables

GPG behavior can be modified with these environment variables:

```bash
# Common environment variables:
export GNUPGHOME=~/.gnupg           # GPG home directory
export GPG_TTY=$(tty)               # Terminal for passphrase prompts
export PINENTRY_USER_DATA="USE_TTY=1" # Force TTY pinentry
```

## Configuration File Examples

### gpg.conf Example

```
# ~/.gnupg/gpg.conf
keyid-format 0xlong
with-fingerprint
personal-cipher-preferences AES256 AES192 AES
personal-digest-preferences SHA512 SHA384 SHA256 SHA224
cert-digest-algo SHA512
default-preference-list SHA512 SHA384 SHA256 SHA224 AES256 AES192 AES ZLIB BZIP2 ZIP Uncompressed
```

### gpg-agent.conf Example

```
# ~/.gnupg/gpg-agent.conf
default-cache-ttl 3600
max-cache-ttl 14400
pinentry-program /usr/bin/pinentry-curses
```

---

This command reference provides a comprehensive overview of GPG commands for key management. For more detailed information on specific commands, refer to the GPG manual pages (`man gpg`) or the [GnuPG documentation](https://www.gnupg.org/documentation/).

**See Also**:
- [Security Best Practices for Key Management](09-security-best-practices.md)
- [Troubleshooting and FAQs](10-troubleshooting-and-faqs.md)
- [Working with Your Transferred GPG Keys in WSL](12-using-gpg-keys-in-wsl.md)
