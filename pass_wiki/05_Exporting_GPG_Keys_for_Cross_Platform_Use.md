# Exporting GPG Keys for Cross-Platform Use

## Introduction

To use Pass on multiple systems, you need to have the same GPG keys on each system. In this guide, we'll cover how to safely export your GPG keys from macOS so they can be imported on your Windows WSL Ubuntu system.

This is a critical step in setting up a cross-platform password management system, as the same private key must be available on all systems where you want to access your passwords.

## Understanding GPG Key Export Security

Before we begin, it's important to understand that your private GPG key is extremely sensitive. Anyone who has access to your private key and knows your passphrase can decrypt all your passwords. Therefore:

- Never share your private key over insecure channels
- Always encrypt your private key when transferring it
- Delete exported key files after they're no longer needed
- Use secure methods for transferring the key files

## Exporting Your GPG Keys

We'll export three components:
1. Your public key
2. Your private key
3. Your trust database

### Step 1: Export Your Public Key

First, let's export your public key:

```bash
gpg --armor --export YOUR_GPG_KEY_ID > public-key.asc
```

Replace `YOUR_GPG_KEY_ID` with your actual GPG key ID (e.g., `1A2B3C4D5E6F7G8H`).

This command:
- Exports your public key in ASCII-armored format (text-based, not binary)
- Saves it to a file named `public-key.asc` in your current directory

The public key is not sensitive and can be shared freely.

### Step 2: Export Your Private Key

Next, export your private key:

```bash
gpg --armor --export-secret-key YOUR_GPG_KEY_ID > private-key.asc
```

This command:
- Exports your private key in ASCII-armored format
- Saves it to a file named `private-key.asc` in your current directory

This file is extremely sensitive and should be protected carefully.

### Step 3: Export Your Trust Database

Finally, export your trust database:

```bash
gpg --export-ownertrust > trust.txt
```

This command:
- Exports your trust database (which records how much you trust different keys)
- Saves it to a file named `trust.txt` in your current directory

## Securing Your Exported Keys

Now that you have exported your keys, you need to secure them for transfer.

### Option 1: Create an Encrypted Archive

One of the safest methods is to create an encrypted archive containing your key files:

```bash
# Create a tar archive containing all three files
tar -czf gpg-keys.tar.gz public-key.asc private-key.asc trust.txt

# Encrypt the archive with a strong password
gpg -c gpg-keys.tar.gz
```

The second command:
- Prompts you to enter a password
- Creates an encrypted file named `gpg-keys.tar.gz.gpg`

This password should be different from your GPG key passphrase and should be strong. You'll need to remember this password to decrypt the archive on your other system.

### Option 2: Use an Encrypted USB Drive

If you have an encrypted USB drive:

```bash
# Assuming your USB drive is mounted at /Volumes/USBDRIVE
cp public-key.asc private-key.asc trust.txt /Volumes/USBDRIVE/
```

Make sure your USB drive is encrypted using FileVault or another full-disk encryption solution.

## Transferring Your Keys Securely

Choose one of these methods to transfer your keys:

### Method 1: Physical Transfer (Most Secure)

1. Copy the encrypted archive or key files to an encrypted USB drive
2. Physically carry the USB drive to your Windows computer
3. Mount the USB drive on your Windows system
4. Access the files from WSL Ubuntu

This method is the most secure as it doesn't involve transmitting your keys over a network.

### Method 2: Secure Network Transfer

If physical transfer isn't possible, you can use secure network protocols:

```bash
# Using SCP (Secure Copy Protocol)
scp gpg-keys.tar.gz.gpg username@windows-computer:/path/to/destination/

# Or using SFTP (SSH File Transfer Protocol)
sftp username@windows-computer
# Then use 'put gpg-keys.tar.gz.gpg' in the SFTP session
```

These commands:
- Establish a secure, encrypted connection to your Windows computer
- Transfer the encrypted archive over this secure connection

You'll need to have SSH server running on your Windows computer for this to work.

### Method 3: Secure Cloud Storage (Less Ideal)

If the above methods aren't available, you can use encrypted cloud storage:

1. Upload the encrypted archive (`gpg-keys.tar.gz.gpg`) to a cloud storage service
2. Download it on your Windows computer
3. Delete it from the cloud storage after successful transfer

This method is less ideal because it involves a third party, but the encryption of the archive provides protection.

## Verifying the Exported Keys

Before proceeding to the next step, verify that your key files were exported correctly:

```bash
# Check the public key
gpg --show-keys public-key.asc

# Check the private key (will only show metadata, not the actual key)
gpg --show-keys private-key.asc

# Check the trust database
cat trust.txt
```

These commands allow you to verify that the files contain the expected information.

## Cleaning Up After Export

After you've successfully transferred and imported your keys on your other system, you should securely delete the exported files from your macOS system:

```bash
# Securely delete the files
srm -v public-key.asc private-key.asc trust.txt gpg-keys.tar.gz gpg-keys.tar.gz.gpg

# If srm is not available, you can use:
rm public-key.asc private-key.asc trust.txt gpg-keys.tar.gz gpg-keys.tar.gz.gpg
```

The `srm` command (secure remove) overwrites the files before deleting them, making recovery more difficult. If `srm` is not available, regular `rm` is better than leaving the files on your system.

## Conclusion

Congratulations! You've successfully:
- Exported your GPG public and private keys
- Exported your trust database
- Secured these files for transfer
- Learned about secure methods for transferring sensitive key material

You're now ready to import these keys on your Windows WSL Ubuntu system, which we'll cover in the next guide.

Remember that your private key is extremely sensitive. Always handle it with care, use strong encryption when transferring it, and delete any copies once they're no longer needed.

In the next guide, we'll set up WSL Ubuntu on your Windows system in preparation for importing your GPG keys.

## Navigation

- [README](README.md) - Wiki Home
- Previous: [Git Integration for Pass](04_Git_Integration_for_Pass.md)
- Next: [Setting Up WSL Ubuntu for Pass](06_Setting_Up_WSL_Ubuntu_for_Pass.md)
- Prerequisites: [Setting Up GPG Keys on macOS](02_Setting_Up_GPG_Keys_on_macOS.md)
