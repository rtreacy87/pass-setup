# Safely Exporting Your GPG Keys from macOS

This guide will walk you through the process of exporting your GPG keys from your Mac, with detailed explanations of each step and command.

## Identifying Your GPG Keys

Before exporting, you need to identify which keys you want to transfer:

```bash
gpg --list-secret-keys --keyid-format LONG
```

What this command does:
- Lists all private keys in your GPG keyring
- `--keyid-format LONG` shows the full key ID (which you'll need for export)
- Displays information about each key including:
  - The key ID (a long hexadecimal string)
  - The user ID (usually your name and email)
  - The creation date
  - The expiration date (if any)

Example output:
```
/Users/username/.gnupg/pubring.kbx
--------------------------------
sec   rsa4096/3AA5C34371567BD2 2022-03-10 [SC]
      1234567890ABCDEF1234567890ABCDEF12345678
uid                 [ultimate] Your Name <your.email@example.com>
ssb   rsa4096/42B317FD4BA89E7A 2022-03-10 [E]
```

In this example:
- `3AA5C34371567BD2` is your key ID (the part after "rsa4096/")
- This is what you'll use in the export commands

## Understanding Key Types

In the output above:
- `sec` means "secret key" (your private key)
- `ssb` means "secret subkey" (additional private keys for specific purposes)
- `[SC]` means this key is used for Signing and Certification
- `[E]` means this key is used for Encryption

## Exporting Your Public Key

First, export your public key:

```bash
gpg --export --armor YOUR_KEY_ID > ~/Desktop/public_key.asc
```

Replace `YOUR_KEY_ID` with the key ID you identified earlier (like `3AA5C34371567BD2`).

What this command does:
- `--export` tells GPG to export the public key
- `--armor` outputs the key in ASCII format (text that's easy to transfer)
- `> ~/Desktop/public_key.asc` saves the output to a file on your Desktop
- The file extension `.asc` indicates an ASCII-armored GPG file

## Exporting Your Private Key

Next, export your private key:

```bash
gpg --export-secret-keys --armor YOUR_KEY_ID > ~/Desktop/private_key.asc
```

What this command does:
- `--export-secret-keys` tells GPG to export the private key
- `--armor` outputs the key in ASCII format
- The command will prompt for your passphrase to verify it's really you
- This is the most sensitive file - it contains your private key

## Exporting Your Trust Database

The trust database contains information about how much you trust other people's keys:

```bash
gpg --export-ownertrust > ~/Desktop/trust.txt
```

What this command does:
- Exports your trust settings for other people's keys
- Saves them as a text file
- This file is important if you've marked other people's keys as trusted

## Exporting Specific Subkeys (Optional)

If you only want to transfer specific subkeys rather than your master key:

```bash
gpg --export-secret-subkeys --armor YOUR_KEY_ID > ~/Desktop/subkeys.asc
```

What this command does:
- Exports only your subkeys, not your master key
- This is a more secure approach for some situations
- Only the capabilities of the subkeys will be available on the new system

## Verifying Your Exported Files

It's important to verify that your exports were successful:

```bash
# Check that the files exist and have content
ls -la ~/Desktop/public_key.asc ~/Desktop/private_key.asc ~/Desktop/trust.txt

# View the beginning of each file to verify it looks correct
head -n 5 ~/Desktop/public_key.asc
head -n 5 ~/Desktop/private_key.asc
head -n 5 ~/Desktop/trust.txt
```

What these commands do:
- `ls -la` lists the files with details about size and permissions
- `head -n 5` shows the first 5 lines of each file
- Public and private key files should start with `-----BEGIN PGP PUBLIC KEY BLOCK-----` and `-----BEGIN PGP PRIVATE KEY BLOCK-----` respectively

## Creating a Backup of Your Entire GPG Directory (Optional)

For a complete backup, you can export your entire GPG directory:

```bash
# Create a tar archive of your .gnupg directory
tar -czf ~/Desktop/gnupg_backup.tar.gz -C ~ .gnupg
```

What this command does:
- `tar` is a program that combines multiple files into one archive
- `-c` creates a new archive
- `-z` compresses the archive with gzip
- `-f ~/Desktop/gnupg_backup.tar.gz` specifies the output file
- `-C ~` changes to your home directory before adding files
- `.gnupg` is the directory to add to the archive

## Best Practices for Key Management on macOS

### Protecting Your Exported Keys

After exporting, immediately secure your keys:

```bash
# Create a directory with restricted permissions
mkdir -p ~/Desktop/gpg_transfer
chmod 700 ~/Desktop/gpg_transfer

# Move your exported keys to this directory
mv ~/Desktop/public_key.asc ~/Desktop/private_key.asc ~/Desktop/trust.txt ~/Desktop/gpg_transfer/
```

What these commands do:
- `mkdir -p` creates a new directory
- `chmod 700` sets permissions so only you can access the directory
- `mv` moves the files to the new directory

### Using Keychain Access for Passphrase Management

macOS can store your GPG passphrase in the Keychain:

```bash
# Edit your GPG agent configuration
echo "pinentry-program /usr/local/bin/pinentry-mac" >> ~/.gnupg/gpg-agent.conf

# Restart the GPG agent
gpgconf --kill gpg-agent
```

What these commands do:
- Configures GPG to use the macOS-specific pinentry program
- This allows your passphrase to be stored in the macOS Keychain
- Restarts the GPG agent to apply the changes

## Troubleshooting Common Export Issues

### "No such key" Error

If you see "No such key" when trying to export:

```bash
# Double-check your key ID
gpg --list-secret-keys --keyid-format LONG

# Try using your email address instead of key ID
gpg --export --armor your.email@example.com > ~/Desktop/public_key.asc
```

### Passphrase Not Accepted

If your passphrase isn't being accepted:

```bash
# Reset the GPG agent
gpgconf --kill gpg-agent
gpg-agent --daemon
```

### Permission Denied Errors

If you get "Permission denied" errors:

```bash
# Check file permissions
ls -la ~/Desktop

# Fix permissions if needed
chmod 700 ~/.gnupg
chmod 600 ~/.gnupg/*
```

### Large Key Files

If your key files are unexpectedly large:

```bash
# Check if you're exporting all keys instead of just one
gpg --list-secret-keys

# Make sure to specify your key ID in the export command
gpg --export --armor YOUR_KEY_ID > ~/Desktop/public_key.asc
```

## Next Steps

After successfully exporting your keys, you'll need to:

1. Secure the exported keys for transfer (covered in the next guide)
2. Transfer them securely to your WSL system
3. Import them on the destination system

Remember: Your private key is extremely sensitive. In the next guide, we'll cover how to properly secure these exported files before transferring them.

---

**Prerequisites**: 
- [Understanding GPG Keys and Secure Transfer Fundamentals](01-introduction-to-gpg-keys.md)
- [Preparing Your Systems for Secure GPG Key Transfer](02-prerequisites-and-system-setup.md)

**Next Steps**: [Protecting Your GPG Keys During Transfer](04-securing-exported-keys.md)

**See Also**:
- [Security Best Practices for Key Management](09-security-best-practices.md)
- [Troubleshooting and FAQs](10-troubleshooting-and-faqs.md)
