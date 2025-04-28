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

If you plan to use your GPG key for signing Git commits:

```bash
# Configure Git to use your GPG key
git config --global user.signingkey YOUR_KEY_ID
git config --global commit.gpgsign true

# Test signing
echo "test" > test_file
git add test_file
git commit -m "Test GPG signing"
```

Replace `YOUR_KEY_ID` with your actual key ID (like `3AA5C34371567BD2`).

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
