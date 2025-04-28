# Successfully Importing GPG Keys into Windows WSL

This guide will walk you through the process of importing your transferred GPG keys into your Windows WSL environment. We'll explain each step in detail so you understand exactly what's happening.

## Preparing the WSL Environment

Before importing your keys, make sure your WSL environment is properly set up:

```bash
# Update your package lists
sudo apt update

# Make sure GPG is installed
sudo apt install -y gnupg

# Check GPG version
gpg --version
```

What these commands do:
- `sudo apt update` refreshes the list of available packages
- `sudo apt install -y gnupg` installs GPG if it's not already installed
- `gpg --version` verifies the installation and shows the version number

You should see output showing GPG version 2.2.x or newer.

Next, check your GPG directory structure:

```bash
# Create the GPG directory if it doesn't exist
mkdir -p ~/.gnupg

# Set proper permissions
chmod 700 ~/.gnupg
```

What these commands do:
- `mkdir -p` creates the directory (and parent directories if needed)
- `chmod 700` sets permissions so only you can access the directory

## Decrypting the Transferred Key Archive

Now that your environment is ready, decrypt the transferred key archive:

```bash
# Navigate to where your encrypted archive is located
cd ~/gpg_transfer

# Decrypt the archive
gpg --decrypt gpg_keys.tar.gz.gpg > gpg_keys.tar.gz
```

What this command does:
- `gpg --decrypt` decrypts the file using GPG
- `> gpg_keys.tar.gz` saves the decrypted output to a new file
- You'll be prompted to enter the passphrase you used when encrypting the archive

When you run this command:
1. GPG will ask for the passphrase you used to encrypt the file
2. After entering the correct passphrase, it will decrypt the archive
3. The decrypted archive will be saved as `gpg_keys.tar.gz`

## Extracting the Key Files

After decrypting, extract the files from the archive:

```bash
# Extract the archive
tar -xzf gpg_keys.tar.gz
```

What this command does:
- `tar` is a program that handles archives
- `-x` extracts files from the archive
- `-z` decompresses the gzip-compressed archive
- `-f gpg_keys.tar.gz` specifies the archive file

This will extract the files you previously exported from your Mac:
- `public_key.asc` - Your public key
- `private_key.asc` - Your private key
- `trust.txt` - Your trust database

Verify the extraction was successful:

```bash
# List the extracted files
ls -la
```

You should see all the extracted files listed.

## Importing Public Keys

First, import your public key:

```bash
# Import the public key
gpg --import public_key.asc
```

What this command does:
- Adds your public key to your GPG keyring in WSL
- Shows information about the imported key
- Displays the key ID and user ID

Example output:
```
gpg: key 3AA5C34371567BD2: public key "Your Name <your.email@example.com>" imported
gpg: Total number processed: 1
gpg:               imported: 1
```

## Securely Importing Private Keys

Next, import your private key:

```bash
# Import the private key
gpg --import private_key.asc
```

What this command does:
- Adds your private key to your GPG keyring
- Requires your key's passphrase (not the archive passphrase)
- Shows information about the imported key

Example output:
```
gpg: key 3AA5C34371567BD2: secret key imported
gpg: Total number processed: 1
gpg:              unchanged: 1
gpg:       secret keys read: 1
gpg:   secret keys imported: 1
```

## Importing the Trust Database

Import your trust database to maintain your trust relationships:

```bash
# Import the trust database
gpg --import-ownertrust trust.txt
```

What this command does:
- Restores your trust settings for other people's keys
- Preserves the web of trust you established on your Mac

Example output:
```
gpg: inserting ownertrust of 6
```

## Verifying Successful Key Import

After importing, verify that your keys were imported correctly:

```bash
# List your secret keys
gpg --list-secret-keys --keyid-format LONG
```

What this command does:
- Lists all private keys in your GPG keyring
- Shows detailed information about each key
- Confirms that your import was successful

You should see output similar to what you saw on your Mac, showing your key ID, user ID, and other details.

## Setting Up GPG Agent in WSL

The GPG agent manages your key passphrases. Set it up properly:

```bash
# Create or edit the GPG agent configuration
nano ~/.gnupg/gpg-agent.conf
```

Add these lines to the file:
```
default-cache-ttl 3600
max-cache-ttl 86400
```

What these settings do:
- `default-cache-ttl 3600` caches your passphrase for 1 hour (3600 seconds)
- `max-cache-ttl 86400` sets the maximum cache time to 24 hours (86400 seconds)

Save the file by pressing Ctrl+O, then Enter, then Ctrl+X to exit.

Restart the GPG agent:

```bash
# Restart the GPG agent
gpgconf --kill gpg-agent
gpg-agent --daemon
```

## Testing the Imported Keys

To ensure your keys are working correctly, perform a simple test:

```bash
# Create a test file
echo "This is a test file" > test.txt

# Encrypt the file with your public key
gpg --encrypt --recipient your.email@example.com test.txt

# This creates test.txt.gpg
# Now try to decrypt it
gpg --decrypt test.txt.gpg
```

What these commands do:
- Create a simple text file
- Encrypt it using your public key
- Attempt to decrypt it using your private key (will prompt for passphrase)

If you can successfully decrypt the file and see "This is a test file", your keys are working correctly!

## Secure Cleanup

After successfully importing your keys, securely delete the temporary files:

```bash
# If you have secure-delete installed
sudo apt install -y secure-delete
srm public_key.asc private_key.asc trust.txt gpg_keys.tar.gz gpg_keys.tar.gz.gpg test.txt test.txt.gpg

# If secure-delete is not available, use shred
shred -u public_key.asc private_key.asc trust.txt gpg_keys.tar.gz gpg_keys.tar.gz.gpg test.txt test.txt.gpg
```

What these commands do:
- `srm` or `shred -u` securely delete files by overwriting them before removal
- This prevents recovery of sensitive key material

## Configuring Git to Use Your GPG Key (Optional)

If you plan to use your GPG key for signing Git commits, you'll need to configure Git to use your private key. This allows you to cryptographically sign your commits, proving they came from you.

### Finding Your GPG Key ID

First, you need to identify the correct key ID to use:

```bash
# List your secret keys and find your key ID
gpg --list-secret-keys --keyid-format LONG
```

The output will look something like this:
```
sec   rsa4096/3AA5C34371567BD2 2022-03-10 [SC]
      1234567890ABCDEF1234567890ABCDEF12345678
uid                 [ultimate] Your Name <your.email@example.com>
ssb   rsa4096/42B317FD4BA89E7A 2022-03-10 [E]
```

What to look for:
- The key ID is the part after `rsa4096/` on the `sec` line (in this example: `3AA5C34371567BD2`)
- This is your primary private key ID that you'll use for signing
- Make sure to use the ID from the `sec` line (primary key), not the `ssb` line (subkey)

You can also use this command to extract just the key ID:

```bash
# Extract the key ID automatically
KEY_ID=$(gpg --list-secret-keys --keyid-format LONG | grep sec | cut -d'/' -f2 | cut -d' ' -f1)
echo "Your key ID is: $KEY_ID"
```

### Configuring Git

Now configure Git to use your key for signing:

```bash
# Configure Git to use your GPG key
git config --global user.signingkey YOUR_KEY_ID
git config --global commit.gpgsign true

# If you want to sign tags by default too
git config --global tag.gpgsign true
```

Replace `YOUR_KEY_ID` with your actual key ID (like `3AA5C34371567BD2`) that you found in the previous step.

What these commands do:
- `user.signingkey` tells Git which key to use for signatures
- `commit.gpgsign true` makes Git sign all commits by default
- `tag.gpgsign true` makes Git sign all tags by default

### Understanding Git Signing: Benefits and Disadvantages

#### Benefits of GPG Signing in Git:

1. **Verification of Identity**: Signing proves that commits actually came from you, not someone impersonating you
2. **Integrity Protection**: Ensures your code hasn't been tampered with after you committed it
3. **Non-repudiation**: You cannot deny making a commit that has your valid signature
4. **Required by Some Projects**: Some open-source projects require signed commits
5. **GitHub Verification Badge**: GitHub displays a "Verified" badge next to signed commits

#### Disadvantages of GPG Signing:

1. **Additional Complexity**: Requires setting up and maintaining GPG keys
2. **Passphrase Management**: Need to manage your GPG passphrase securely
3. **Performance Impact**: Slight delay when making commits due to signing process
4. **Key Management Overhead**: Need to handle key expiration, revocation, and backup
5. **Potential Workflow Disruption**: Entering passphrase can interrupt coding flow

### Passphrase Requirements and Caching

When using GPG signing with Git, you don't need to enter your passphrase for every commit thanks to the GPG agent's caching mechanism:

- **First Commit**: You'll be prompted for your passphrase
- **Subsequent Commits**: No passphrase needed until the cache expires
- **Cache Expiration**: Based on your `default-cache-ttl` setting (we set it to 3600 seconds/1 hour)
- **After Inactivity**: You'll need to enter your passphrase again after the cache expires
- **After System Restart**: You'll need to enter your passphrase again

The GPG agent handles this caching securely, so you get both security and convenience.

### Enabling and Disabling Signing

You can enable or disable signing at different levels:

#### Global Enabling (for all repositories):
```bash
# Enable signing for all commits
git config --global commit.gpgsign true

# Enable signing for all tags
git config --global tag.gpgsign true
```

#### Global Disabling:
```bash
# Disable signing for all commits
git config --global commit.gpgsign false

# Disable signing for all tags
git config --global tag.gpgsign false
```

#### Repository-Specific Settings:
```bash
# Enable just for current repository (omit --global)
git config commit.gpgsign true

# Disable just for current repository
git config commit.gpgsign false
```

#### Per-Commit Control:
```bash
# Force signing for a single commit (even if disabled globally)
git commit -S -m "Signed commit message"

# Skip signing for a single commit (even if enabled globally)
git commit --no-gpg-sign -m "Unsigned commit message"
```

#### Per-Tag Control:
```bash
# Force signing for a single tag
git tag -s v1.0 -m "Signed tag"

# Create unsigned tag
git tag -a v1.0 -m "Unsigned tag"
```

### Testing Git Signing

To verify that Git signing is working correctly:

```bash
# Create a test repository
mkdir -p ~/git-test && cd ~/git-test
git init

# Configure user information for this test
git config user.name "Your Name"
git config user.email "your.email@example.com"

# Create a test file and commit it
echo "This is a test file" > test_file
git add test_file
git commit -m "Test GPG signing"
```

If your key is properly configured, Git will prompt you for your GPG key passphrase during the commit (but only the first time, thanks to caching).

### Verifying the Signed Commit

To confirm that your commit was properly signed:

```bash
# Check the signature on your commit
git log --show-signature -1
```

You should see output that includes:
```
commit abcdef1234567890... (HEAD -> master)
gpg: Signature made Wed Apr 27 21:30:00 2023 UTC
gpg:                using RSA key 3AA5C34371567BD2
gpg: Good signature from "Your Name <your.email@example.com>" [ultimate]
Author: Your Name <your.email@example.com>
Date:   Wed Apr 27 21:30:00 2023

    Test GPG signing
```

The "Good signature" line confirms that the signing worked correctly.

## Troubleshooting Common Import Issues

### "No such file or directory"

If you see this error:

```bash
# Check if the files exist
ls -la

# Make sure you're in the right directory
pwd
```

### "Secret key import failed: No such file or directory"

This usually means GPG can't access its directory:

```bash
# Check permissions on .gnupg directory
ls -la ~ | grep .gnupg

# Fix if needed
mkdir -p ~/.gnupg
chmod 700 ~/.gnupg
```

### "Inappropriate ioctl for device"

This can happen when GPG can't request a passphrase:

```bash
# Set GPG to use a simpler passphrase method
echo "pinentry-mode loopback" >> ~/.gnupg/gpg.conf
```

### "gpg: decryption failed: No secret key"

This means your private key wasn't imported correctly:

```bash
# Check if your private key is in your keyring
gpg --list-secret-keys

# Try importing again
gpg --import private_key.asc
```

### Fixing "gpg: signing failed: Inappropriate ioctl for device" in Git

If you encounter this error when trying to sign Git commits, it's a common issue in WSL where GPG can't properly access the terminal for passphrase input. Here are several solutions:

#### Solution 1: Set GPG to use loopback pinentry mode

```bash
# Add this to your GPG configuration
echo "pinentry-mode loopback" >> ~/.gnupg/gpg.conf

# Also add this to tell GPG which TTY to use
echo 'export GPG_TTY=$(tty)' >> ~/.bashrc
source ~/.bashrc
```

What this does:
- `pinentry-mode loopback` tells GPG to use a simpler method for passphrase input
- `export GPG_TTY=$(tty)` helps GPG find your terminal for input

#### Solution 2: Configure GPG to use the correct pinentry program

```bash
# Install a terminal-based pinentry program
sudo apt install pinentry-curses

# Configure GPG to use it
echo "pinentry-program /usr/bin/pinentry-curses" > ~/.gnupg/gpg-agent.conf

# Restart the GPG agent
gpgconf --kill gpg-agent
gpg-agent --daemon
```

What this does:
- Installs a terminal-friendly pinentry program
- Configures GPG to use this program for passphrase prompts
- Restarts the agent to apply changes

#### Solution 3: Use a different GPG_TTY approach

```bash
# Add these lines to your ~/.bashrc file
echo 'export GPG_TTY=$TTY' >> ~/.bashrc
echo 'gpg-connect-agent updatestartuptty /bye > /dev/null' >> ~/.bashrc
source ~/.bashrc
```

What this does:
- Sets the GPG TTY variable differently
- Tells the GPG agent to update its TTY settings on startup
- Applies the changes to your current session

#### Solution 4: Temporarily bypass signing for a single commit

If you need to make a commit urgently and can't solve the GPG issue right away:

```bash
# Make a commit without signing
git commit --no-gpg-sign -m "your commit message"
```

What this does:
- Creates a commit without a GPG signature
- Overrides your global signing settings for this one commit

#### Verifying the Fix

After applying one of these solutions, try making a signed commit again:

```bash
# Create a test file
echo "test" > test_file
git add test_file
git commit -m "Test GPG signing after fix"
```

If the commit succeeds without the error, the issue is resolved.

## Next Steps

Now that you've successfully imported your GPG keys into WSL, you can:

1. Use them to encrypt and decrypt files
2. Sign Git commits and tags
3. Use them with password managers like `pass`
4. Set up additional applications to use your keys

For more information on using your keys in WSL, see [Working with Your Transferred GPG Keys in WSL](12-using-gpg-keys-in-wsl.md).

---

**Prerequisites**: 
- [Understanding GPG Keys and Secure Transfer Fundamentals](01-introduction-to-gpg-keys.md)
- [Transferring Keys Using SCP](05-transferring-keys-using-scp.md) or [Transferring Keys Using SFTP](06-transferring-keys-using-sftp.md)

**Next Steps**: [Working with Your Transferred GPG Keys in WSL](12-using-gpg-keys-in-wsl.md)

**See Also**:
- [Security Best Practices for Key Management](09-security-best-practices.md)
- [Troubleshooting and FAQs](10-troubleshooting-and-faqs.md)
