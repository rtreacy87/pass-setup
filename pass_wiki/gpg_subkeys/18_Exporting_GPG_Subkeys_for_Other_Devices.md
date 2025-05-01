# Exporting GPG Subkeys for Other Devices

## Introduction

Now that you've created GPG subkeys, the next step is to export them so they can be used on your other devices. This guide will walk you through the process of exporting only your subkeys (not your master key) for use on secondary devices.

The key difference between this approach and the standard key export (covered in [Exporting GPG Keys for Cross-Platform Use](../05_Exporting_GPG_Keys_for_Cross_Platform_Use.md)) is that we'll be creating a **partial export** that includes only the subkeys, not your master key. This enhances security by keeping your master key isolated.

## Creating a Subkey-Only Export

We'll create three files:
1. Your public key (which includes all public key material)
2. A partial secret key export (containing only your subkeys, not your master key)
3. Your trust database

### Step 1: Export Your Public Key

First, export your public key:

```bash
gpg --armor --export YOUR_KEY_ID > public-key.asc
```

Replace `YOUR_KEY_ID` with your actual GPG key ID (e.g., `1A2B3C4D5E6F7G8H`).

This command:
- Exports your public key in ASCII-armored format (text-based, not binary)
- Saves it to a file named `public-key.asc` in your current directory

### Step 2: Export Your Secret Subkeys Only

Now, here's the important difference - we'll export only your secret subkeys, not your master key:

```bash
gpg --armor --export-secret-subkeys YOUR_KEY_ID > secret-subkeys.asc
```

This command:
- Exports only your secret subkeys in ASCII-armored format
- Does NOT include your master/primary secret key
- Saves the export to a file named `secret-subkeys.asc`

### Step 3: Export Your Trust Database

Finally, export your trust database:

```bash
gpg --export-ownertrust > trust.txt
```

This command:
- Exports your trust database (which records how much you trust different keys)
- Saves it to a file named `trust.txt` in your current directory

### Step 4: Verify Your Exports

Let's verify that your exports contain the expected information:

```bash
# Check the public key
gpg --show-keys public-key.asc

# Check the secret subkeys (will only show metadata, not the actual keys)
gpg --show-keys secret-subkeys.asc

# Check the trust database
cat trust.txt
```

In the output for `secret-subkeys.asc`, you should see entries for your subkeys but not for your master key. This confirms that you've created a partial export that doesn't include your master key.

## Securing the Exported Subkeys

Even though this export doesn't include your master key, the secret subkeys are still sensitive and need to be protected during transfer.

### Creating an Encrypted Archive

Let's create an encrypted archive containing all three files:

```bash
# Create a directory to hold the files
mkdir -p ~/subkey-export
chmod 700 ~/subkey-export

# Move the files to this directory
mv public-key.asc secret-subkeys.asc trust.txt ~/subkey-export/

# Create a tar archive
tar -czf ~/subkey-export.tar.gz -C ~ subkey-export

# Encrypt the archive
gpg -c ~/subkey-export.tar.gz
```

The last command will prompt you to create a password for the encrypted archive. This should be:
- Different from your GPG passphrase
- Strong and complex
- Something you can remember or securely store
- Communicated securely to yourself on the other device

### Password Protection Best Practices

When creating a password for your encrypted archive:

1. **Use a strong, unique password**: Include uppercase and lowercase letters, numbers, and special characters.

2. **Don't reuse passwords**: Don't use the same password you use for other services.

3. **Consider a passphrase**: A long phrase with spaces can be more secure and easier to remember than a complex short password.

4. **Secure communication**: If you need to communicate this password to yourself on another device, use a secure method like:
   - A password manager you already have access to on both devices
   - A secure messaging app with end-to-end encryption
   - In person (writing it down temporarily and then destroying the paper)

## Backing Up Your Master Key Separately

While your subkeys are now ready for export to other devices, it's crucial to create a secure backup of your complete key (including the master key) and store it safely.

### Creating a Full Backup

```bash
# Create a directory for the full backup
mkdir -p ~/gpg-full-backup
chmod 700 ~/gpg-full-backup

# Export your public key
gpg --armor --export YOUR_KEY_ID > ~/gpg-full-backup/public-key-full.asc

# Export your complete secret key (including master key)
gpg --armor --export-secret-key YOUR_KEY_ID > ~/gpg-full-backup/secret-key-full.asc

# Export your trust database
gpg --export-ownertrust > ~/gpg-full-backup/trust-full.txt

# Create and encrypt an archive
tar -czf ~/gpg-full-backup.tar.gz -C ~ gpg-full-backup
gpg -c ~/gpg-full-backup.tar.gz
```

Use a different, very strong password for this archive, as it contains your master key.

### Secure Storage Options

Your master key backup should be stored with the highest level of security. Options include:

#### 1. Offline Storage

Store the encrypted backup on media that remains disconnected from the internet:

```bash
# Copy to an encrypted USB drive
cp ~/gpg-full-backup.tar.gz.gpg /Volumes/ENCRYPTED_USB/
```

Keep this USB drive in a secure physical location, like a safe.

#### 2. Paper Backup (Emergency Recovery)

For ultimate disaster recovery, consider a paper backup:

```bash
# Generate a QR code of your encrypted backup
# First, install qrencode if you don't have it
brew install qrencode

# Create QR codes (this will create multiple image files for large keys)
qrencode -r ~/gpg-full-backup.tar.gz.gpg -o gpg-backup.png -s 6 -l H
```

Print these QR codes and store them in a secure location like a safe deposit box. To recover, you would scan the QR codes and decrypt the archive.

#### 3. Split Storage

For critical keys, consider splitting the backup across multiple locations:

```bash
# Install ssss (Shamir's Secret Sharing Scheme)
brew install ssss

# Split your password into 3 parts, requiring 2 to reconstruct
echo "your-very-strong-password" | ssss-split -t 2 -n 3
```

Store each part in a different secure location. This way, no single location contains your complete backup password.

## Transferring Subkeys Between Devices

Now that you've secured your exports, you need to transfer them to your other devices.

### Method 1: Physical Transfer (Most Secure)

The most secure method is physical transfer using an encrypted USB drive:

```bash
# Copy the encrypted archive to a USB drive
cp ~/subkey-export.tar.gz.gpg /Volumes/USBDRIVE/

# Securely delete the local copy (optional)
shred -u ~/subkey-export.tar.gz.gpg  # On Linux
# or
rm ~/subkey-export.tar.gz.gpg  # On macOS
```

Then physically carry the USB drive to your other device.

### Method 2: Secure Network Transfer

If physical transfer isn't practical, use a secure network protocol:

```bash
# Using SCP (Secure Copy Protocol)
scp ~/subkey-export.tar.gz.gpg username@other-device:/destination/path/

# Or using SFTP (SSH File Transfer Protocol)
sftp username@other-device
# Then use 'put ~/subkey-export.tar.gz.gpg' in the SFTP session
```

These commands establish a secure, encrypted connection to transfer the file.

### Method 3: Secure Cloud Storage (Less Ideal)

If the above methods aren't available, you can use encrypted cloud storage:

1. Upload the encrypted archive (`subkey-export.tar.gz.gpg`) to a cloud storage service
2. Download it on your other device
3. Delete it from the cloud storage after successful transfer

This method is less ideal because it involves a third party, but the encryption of the archive provides protection.

### Verification Procedures

After transferring the files to your other device, verify the integrity of the transfer:

```bash
# Check the file size matches the original
ls -l subkey-export.tar.gz.gpg

# If you're concerned about file integrity, verify with a checksum
# On the source device:
shasum -a 256 ~/subkey-export.tar.gz.gpg

# On the destination device:
shasum -a 256 subkey-export.tar.gz.gpg
```

The checksum values should match on both devices, confirming the file was transferred correctly.

## Cleaning Up After Export

After you've successfully transferred your subkeys, clean up the sensitive files from your main computer:

```bash
# Remove the export directory and its contents
rm -rf ~/subkey-export

# Remove the encrypted archive if you still have it locally
rm ~/subkey-export.tar.gz.gpg

# If you created a full backup and have stored it securely
rm ~/gpg-full-backup.tar.gz.gpg
rm -rf ~/gpg-full-backup
```

## Conclusion

You've now successfully:
- Exported only your GPG subkeys (keeping your master key secure)
- Created an encrypted archive of these exports
- Created a separate, secure backup of your full GPG key
- Learned about secure methods for transferring your subkeys
- Cleaned up sensitive files after export

In the next guide, we'll cover how to import these subkeys on your secondary devices and configure `pass` to use them.

## Navigation

- [README](../README.md) - Wiki Home
- Previous: [Creating and Managing GPG Subkeys](17_Creating_and_Managing_GPG_Subkeys.md)
- Next: [Setting Up Pass with GPG Subkeys](19_Setting_Up_Pass_with_GPG_Subkeys.md)
- Related: [Exporting GPG Keys for Cross-Platform Use](../05_Exporting_GPG_Keys_for_Cross_Platform_Use.md)
