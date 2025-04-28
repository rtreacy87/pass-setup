# Working with Your Transferred GPG Keys in WSL

This guide explains how to effectively use your GPG keys in Windows WSL after you've successfully transferred and imported them. We'll cover common use cases, configuration tips, and best practices.

## Configuring GPG in WSL for Daily Use

After importing your keys, you'll want to configure GPG for optimal use in WSL:

### Basic GPG Configuration

Create or edit your GPG configuration file:

```bash
# Create or edit gpg.conf
nano ~/.gnupg/gpg.conf
```

Add these useful settings:

```
# Display key IDs in a more readable format
keyid-format 0xlong

# Show key fingerprints
with-fingerprint

# Display the calculated validity of user IDs
list-options show-uid-validity
verify-options show-uid-validity

# Use stronger digest algorithm
personal-digest-preferences SHA512 SHA384 SHA256 SHA224

# Use stronger cipher algorithm
personal-cipher-preferences AES256 AES192 AES
```

What these settings do:
- Make key IDs easier to read and verify
- Show fingerprints for better key verification
- Display validity information for user IDs
- Set stronger encryption and hashing algorithms

### Configuring GPG Agent

The GPG agent manages your passphrases. Configure it for convenience:

```bash
# Create or edit gpg-agent.conf
nano ~/.gnupg/gpg-agent.conf
```

Add these settings:

```
# Cache passphrases for 1 hour
default-cache-ttl 3600

# Maximum cache time is 4 hours
max-cache-ttl 14400

# Use text-based pinentry
pinentry-program /usr/bin/pinentry-curses
```

What these settings do:
- Cache your passphrase for 1 hour after entry
- Set a maximum cache time of 4 hours
- Specify the text-based pinentry program for WSL

Restart the agent to apply changes:

```bash
gpgconf --kill gpg-agent
gpg-agent --daemon
```

### Testing Your Configuration

Verify your configuration is working:

```bash
# Create a test file
echo "This is a test message" > test.txt

# Sign the file
gpg --sign test.txt

# This creates test.txt.gpg
# Verify the signature
gpg --verify test.txt.gpg
```

What these commands do:
- Create a simple text file
- Sign it with your private key
- Verify the signature

If you can sign and verify without errors, your configuration is working correctly.

## Integrating with Git for Signed Commits

One of the most common uses for GPG keys in development is signing Git commits:

### Configuring Git to Use Your GPG Key

```bash
# Set your signing key (replace with your actual key ID)
git config --global user.signingkey YOUR_KEY_ID

# Enable commit signing by default
git config --global commit.gpgsign true

# Enable tag signing by default
git config --global tag.gpgsign true
```

What these commands do:
- Tell Git which GPG key to use for signing
- Configure Git to automatically sign all commits
- Configure Git to automatically sign all tags

### Testing Git Signing

Test that Git signing works:

```bash
# Create a test repository
mkdir ~/git-test
cd ~/git-test
git init

# Configure user information
git config user.name "Your Name"
git config user.email "your.email@example.com"

# Create and commit a test file
echo "Test file" > test.txt
git add test.txt
git commit -m "Test signed commit"
```

What these commands do:
- Create a new Git repository
- Set up your user information
- Create and commit a file with signing

Verify the commit is signed:

```bash
git log --show-signature
```

You should see output indicating the commit was signed with your GPG key.

### Configuring GitHub to Recognize Your GPG Key

If you use GitHub, you can add your GPG public key to your account:

```bash
# Export your public key
gpg --armor --export YOUR_KEY_ID
```

What this does:
- Displays your public key in ASCII-armored format
- You can copy this output and add it to your GitHub account settings

To add the key to GitHub:
1. Copy the output (including the BEGIN and END lines)
2. Go to GitHub → Settings → SSH and GPG keys
3. Click "New GPG key"
4. Paste your public key and save

### Troubleshooting Git Signing Issues

If you encounter problems with Git signing:

```bash
# Check if Git can find GPG
which gpg

# Tell Git explicitly where to find GPG
git config --global gpg.program $(which gpg)

# Set GPG to use the terminal for passphrase input
echo "no-tty" >> ~/.gnupg/gpg.conf
```

What these commands do:
- Verify GPG is in your PATH
- Explicitly tell Git where to find GPG
- Configure GPG to work better with Git's passphrase prompts

## Setting Up Password Caching with GPG Agent

The GPG agent can cache your passphrase so you don't have to enter it repeatedly:

### Basic Passphrase Caching

The settings we added to `gpg-agent.conf` earlier enable basic caching:

```
default-cache-ttl 3600
max-cache-ttl 14400
```

What these do:
- Cache your passphrase for 1 hour (3600 seconds) after each use
- Set a maximum cache time of 4 hours (14400 seconds)

### Testing Passphrase Caching

Test that caching is working:

```bash
# First operation (should prompt for passphrase)
gpg --sign test.txt

# Second operation within cache time (should not prompt)
gpg --sign test2.txt
```

If you're only prompted once, caching is working correctly.

### Adjusting Cache Times

You can adjust cache times based on your security needs:

```bash
# For higher security (shorter times)
echo "default-cache-ttl 300" >> ~/.gnupg/gpg-agent.conf
echo "max-cache-ttl 600" >> ~/.gnupg/gpg-agent.conf

# For more convenience (longer times)
echo "default-cache-ttl 28800" >> ~/.gnupg/gpg-agent.conf
echo "max-cache-ttl 86400" >> ~/.gnupg/gpg-agent.conf

# Reload the agent
gpgconf --kill gpg-agent
gpg-agent --daemon
```

What these settings do:
- Higher security: Cache for 5 minutes, max 10 minutes
- More convenience: Cache for 8 hours, max 24 hours

Choose settings that balance your security needs and convenience.

## Using Keys with Other Applications in WSL

Your GPG keys can be used with many other applications in WSL:

### Using GPG with the `pass` Password Manager

The `pass` password manager uses GPG for encryption:

```bash
# Install pass
sudo apt install pass

# Initialize pass with your GPG key
pass init YOUR_KEY_ID

# Store a password
pass insert email/gmail

# Retrieve a password
pass email/gmail
```

What these commands do:
- Install the `pass` password manager
- Set up `pass` to use your GPG key for encryption
- Store and retrieve passwords securely

### Encrypting and Decrypting Files

Use your keys for file encryption:

```bash
# Encrypt a file for yourself
gpg --encrypt --recipient YOUR_EMAIL file.txt

# Decrypt a file
gpg --decrypt file.txt.gpg > decrypted_file.txt

# Encrypt a file with a password (for sharing with others)
gpg --symmetric --cipher-algo AES256 file.txt
```

What these commands do:
- Encrypt files using your public key
- Decrypt files using your private key
- Create password-encrypted files for sharing

### Using GPG with Email in WSL

If you use command-line email clients in WSL:

```bash
# For Mutt/NeoMutt, add to ~/.muttrc:
set pgp_use_gpg_agent = yes
set pgp_sign_as = "YOUR_KEY_ID"
set pgp_timeout = 3600
```

What these settings do:
- Configure the email client to use your GPG key
- Enable the GPG agent for passphrase handling
- Set how long to cache your passphrase

## Sharing Keys Between WSL and Windows Host (Pros/Cons)

You might wonder if you can share your GPG keys between WSL and Windows:

### Option 1: Separate Keys (Recommended)

Keep separate keys in WSL and Windows:

**Pros:**
- Better security isolation
- No compatibility issues
- Follows security best practices

**Cons:**
- Need to manage multiple keys
- Need to keep keys in sync manually

### Option 2: Sharing Keys (Advanced)

Share your WSL keys with Windows:

```bash
# Export your public and private keys from WSL
gpg --export --armor YOUR_KEY_ID > ~/public_key.asc
gpg --export-secret-keys --armor YOUR_KEY_ID > ~/private_key.asc

# Import in Windows (using Gpg4win/Kleopatra)
# 1. Copy the files to Windows
# 2. Import using Kleopatra
```

**Pros:**
- Single set of keys to manage
- Same identity across systems

**Cons:**
- Increased security risk
- Potential compatibility issues
- More complex setup

### Option 3: Using Windows GPG in WSL (Advanced)

Configure WSL to use the Windows GPG installation:

```bash
# Add to your ~/.bashrc
export GPG_AGENT_SOCK="/mnt/c/Users/YourUsername/AppData/Roaming/gnupg/S.gpg-agent"
alias gpg="'/mnt/c/Program Files (x86)/GnuPG/bin/gpg.exe'"
```

**Pros:**
- Single key management
- Windows GUI tools available

**Cons:**
- Complex configuration
- Potential path and permission issues
- Less isolation between systems

For most users, Option 1 (separate keys) provides the best balance of security and usability.

## Maintaining Key Security in the WSL Environment

WSL presents some unique security considerations:

### Protecting Your Keys in WSL

```bash
# Set proper permissions
chmod 700 ~/.gnupg
chmod 600 ~/.gnupg/*

# Add to your ~/.bashrc to ensure proper permissions
echo "umask 077" >> ~/.bashrc
```

What these commands do:
- Set restrictive permissions on your GPG directory
- Set a default umask that creates private files

### Handling WSL Shutdown Behavior

WSL doesn't always shut down cleanly, which can affect GPG agent:

```bash
# Add to your ~/.bashrc
gpg-connect-agent /bye >/dev/null 2>&1 || true
```

What this does:
- Ensures the GPG agent is running when you start a shell
- Handles cases where the agent wasn't properly shut down

### Backup Considerations for WSL

Remember that WSL filesystems can be affected by Windows updates:

```bash
# Create periodic backups of your GPG directory
mkdir -p ~/backups
tar -czf ~/backups/gnupg-backup-$(date +%Y%m%d).tar.gz -C ~ .gnupg
chmod 600 ~/backups/gnupg-backup-*.tar.gz
```

What this does:
- Creates a dated backup of your GPG directory
- Sets secure permissions on the backup
- Protects against accidental data loss

## Practical Examples

Here are some practical examples of using GPG in WSL:

### Example 1: Encrypting a File for a Colleague

```bash
# Import your colleague's public key
gpg --import colleague_public_key.asc

# Encrypt a file for them
gpg --encrypt --recipient colleague@example.com sensitive_document.txt

# The result is sensitive_document.txt.gpg
# You can safely email this encrypted file
```

What this does:
- Imports your colleague's public key
- Encrypts a file so only they can decrypt it
- Creates an encrypted file you can safely share

### Example 2: Setting Up Automatic Git Signing

```bash
# Create a script to ensure Git signing works
nano ~/bin/git-sign-check.sh
```

Add this content:
```bash
#!/bin/bash
# Check if Git signing is working

# Create test repo
TEMP_DIR=$(mktemp -d)
cd "$TEMP_DIR"
git init >/dev/null 2>&1
git config user.name "Test User"
git config user.email "test@example.com"
git config user.signingkey "YOUR_KEY_ID"
git config commit.gpgsign true

# Try to create a signed commit
echo "test" > test.txt
git add test.txt >/dev/null 2>&1
if git commit -m "Test signed commit" >/dev/null 2>&1; then
    echo "✓ Git signing is working correctly"
    rm -rf "$TEMP_DIR"
    exit 0
else
    echo "✗ Git signing failed"
    rm -rf "$TEMP_DIR"
    exit 1
fi
```

Make it executable and run it:
```bash
chmod +x ~/bin/git-sign-check.sh
~/bin/git-sign-check.sh
```

What this does:
- Creates a temporary Git repository
- Attempts to make a signed commit
- Reports whether signing is working correctly

### Example 3: Using GPG with the Pass Password Manager

```bash
# Install pass
sudo apt install pass

# Initialize with your key
pass init YOUR_EMAIL

# Add some passwords
pass insert email/work
pass insert banking/checking

# Generate a random password
pass generate website/example 15

# Retrieve a password
pass email/work

# Copy a password to clipboard (requires xclip)
sudo apt install xclip
pass -c banking/checking
```

What these commands do:
- Set up a GPG-encrypted password store
- Store various passwords securely
- Generate strong random passwords
- Retrieve passwords when needed

## Next Steps and Advanced Topics

After mastering the basics of using GPG in WSL, you might want to explore:

### Hardware Security Keys

For enhanced security, consider using a hardware security key:

```bash
# Install required software
sudo apt install scdaemon pcscd

# Check if your security key is detected
gpg --card-status
```

### Encrypted Backup Solutions

Set up automated encrypted backups:

```bash
# Install restic backup tool
sudo apt install restic

# Initialize a backup repository
restic init --repo /path/to/backup

# Back up your GPG directory
restic backup ~/.gnupg
```

### Advanced Key Management

Explore more advanced key management techniques:

```bash
# Create a separate signing subkey
gpg --expert --edit-key YOUR_KEY_ID

# At the gpg> prompt:
addkey
# Choose "RSA (sign only)"
# Follow prompts to create the key
save
```

---

**Prerequisites**: 
- [Successfully Importing GPG Keys into Windows WSL](07-importing-gpg-keys-in-wsl.md)

**See Also**:
- [Security Best Practices for Key Management](09-security-best-practices.md)
- [Troubleshooting and FAQs](10-troubleshooting-and-faqs.md)
- [Command Reference for GPG Key Management](appendix-command-reference.md)
