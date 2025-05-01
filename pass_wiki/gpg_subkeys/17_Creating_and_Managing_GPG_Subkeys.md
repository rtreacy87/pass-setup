# Creating and Managing GPG Subkeys

## Introduction

In this guide, we'll walk through the process of creating and managing GPG subkeys for use with the `pass` password manager. This approach allows you to keep your master key secure while still using `pass` on multiple devices.

Before we begin, it's important to understand that we'll be working with your GPG keys, which are the foundation of your password security. We'll take careful steps to ensure everything is backed up properly before making any changes.

## Prerequisites

Before creating subkeys, make sure you have:

1. An existing GPG key setup (if you don't, follow the [Setting Up GPG Keys on macOS](../02_Setting_Up_GPG_Keys_on_macOS.md) guide first)
2. A backup of your existing GPG keys (we'll cover this below)
3. About 30 minutes of uninterrupted time

## Backing Up Your Existing GPG Keys

Before making any changes to your GPG setup, it's crucial to create a backup of your existing keys.

### Step 1: Create a Backup Directory

First, let's create a directory to store your backups:

```bash
mkdir -p ~/gpg-backup
chmod 700 ~/gpg-backup
```

This creates a directory with permissions that only allow you to access it.

### Step 2: Export Your Public and Private Keys

Export your keys to the backup directory:

```bash
# List your keys to find your key ID
gpg --list-secret-keys --keyid-format LONG

# Export your public key
gpg --armor --export YOUR_KEY_ID > ~/gpg-backup/public-key-backup.asc

# Export your private key
gpg --armor --export-secret-key YOUR_KEY_ID > ~/gpg-backup/private-key-backup.asc

# Export your trust database
gpg --export-ownertrust > ~/gpg-backup/trust-backup.txt
```

Replace `YOUR_KEY_ID` with your actual GPG key ID, which looks something like `1A2B3C4D5E6F7G8H`.

### Step 3: Secure Your Backup

Create an encrypted archive of your backup:

```bash
# Create a tar archive
tar -czf ~/gpg-backup.tar.gz -C ~ gpg-backup

# Encrypt the archive
gpg -c ~/gpg-backup.tar.gz
```

You'll be prompted to create a password for this encrypted archive. Make it strong and don't forget it!

### Step 4: Store the Backup Securely

Move the encrypted backup to a secure location, such as an encrypted USB drive:

```bash
# Assuming your USB drive is mounted at /Volumes/USBDRIVE
cp ~/gpg-backup.tar.gz.gpg /Volumes/USBDRIVE/
```

Now that your keys are safely backed up, we can proceed with creating subkeys.

## Creating Subkeys for Your Master Key

GPG allows you to add subkeys to your existing master key. We'll create two types of subkeys:
1. An encryption subkey (for encrypting/decrypting passwords)
2. A signing subkey (for signing Git commits)

### Step 1: Edit Your GPG Key

Start by opening your key for editing:

```bash
gpg --expert --edit-key YOUR_KEY_ID
```

This opens an interactive GPG prompt where you can modify your key.

### Step 2: Add an Encryption Subkey

At the GPG prompt, add a new encryption subkey:

```
gpg> addkey
```

You'll be asked to select the type of key. Choose:
```
Please select what kind of key you want:
   (3) DSA (sign only)
   (4) RSA (sign only)
   (5) Elgamal (encrypt only)
   (6) RSA (encrypt only)
Your selection? 6
```

Next, choose the key size (4096 bits is recommended for good security):
```
RSA keys may be between 1024 and 4096 bits long.
What keysize do you want? (3072) 4096
```

Set an expiration date (1 year is a good practice, as it forces regular rotation):
```
Please specify how long the key should be valid.
         0 = key does not expire
      <n>  = key expires in n days
      <n>w = key expires in n weeks
      <n>m = key expires in n months
      <n>y = key expires in n years
Key is valid for? (0) 1y
```

Confirm the expiration date:
```
Key expires at ...
Is this correct? (y/N) y
```

You'll be asked to confirm the creation of the subkey:
```
Really create? (y/N) y
```

### Step 3: Add a Signing Subkey

Still in the GPG prompt, add a signing subkey:

```
gpg> addkey
```

Choose RSA (sign only):
```
Please select what kind of key you want:
   (3) DSA (sign only)
   (4) RSA (sign only)
   (5) Elgamal (encrypt only)
   (6) RSA (encrypt only)
Your selection? 4
```

Set the key size to 4096 bits:
```
RSA keys may be between 1024 and 4096 bits long.
What keysize do you want? (3072) 4096
```

Set an expiration date (1 year is recommended):
```
Please specify how long the key should be valid.
         0 = key does not expire
      <n>  = key expires in n days
      <n>w = key expires in n weeks
      <n>m = key expires in n months
      <n>y = key expires in n years
Key is valid for? (0) 1y
```

Confirm the expiration date and creation:
```
Key expires at ...
Is this correct? (y/N) y
Really create? (y/N) y
```

### Step 4: Save Your Changes

Save the changes to your key:

```
gpg> save
```

This will exit the GPG prompt and save your new subkeys.

### Step 5: Verify Your Subkeys

Check that your subkeys were created correctly:

```bash
gpg --list-secret-keys --keyid-format LONG
```

You should see output similar to:

```
sec   rsa4096/1A2B3C4D5E6F7G8H 2023-01-01 [SC] [expires: 2025-01-01]
      ABCDEF0123456789ABCDEF0123456789ABCDEF01
uid                 [ultimate] Your Name <your.email@example.com>
ssb   rsa4096/8H7G6F5E4D3C2B1A 2023-01-01 [E] [expires: 2025-01-01]
ssb   rsa4096/2B1A8H7G6F5E4D3C 2023-06-01 [E] [expires: 2024-06-01]
ssb   rsa4096/6F5E4D3C2B1A8H7G 2023-06-01 [S] [expires: 2024-06-01]
```

In this output:
- `sec` is your master/primary key
- `ssb` entries are your subkeys
- `[E]` indicates an encryption subkey
- `[S]` indicates a signing subkey

Note the IDs of your new subkeys (in this example, `2B1A8H7G6F5E4D3C` for encryption and `6F5E4D3C2B1A8H7G` for signing). You'll need these later.

## Managing Subkey Expiration

Setting expiration dates on your subkeys is a good security practice. When a subkey expires, you'll need to either extend its expiration date or create a new subkey.

### Extending a Subkey's Expiration Date

To extend the expiration date of a subkey:

```bash
# Edit your key
gpg --edit-key YOUR_KEY_ID

# Select the subkey to modify (replace N with the number of the subkey)
gpg> key N

# Set a new expiration date
gpg> expire

# Follow the prompts to set a new expiration date
# Save your changes
gpg> save
```

### Creating Replacement Subkeys

When a subkey is about to expire, you can create a new subkey following the same process we used earlier:

```bash
gpg --expert --edit-key YOUR_KEY_ID
gpg> addkey
# Follow the prompts to create a new subkey
gpg> save
```

### Revoking Compromised Subkeys

If you believe a subkey has been compromised (e.g., a device with the subkey was stolen), you should revoke it:

```bash
# Edit your key
gpg --edit-key YOUR_KEY_ID

# Select the subkey to revoke (replace N with the number of the subkey)
gpg> key N

# Revoke the subkey
gpg> revkey

# Confirm the revocation
# Save your changes
gpg> save
```

## Best Practices for Subkey Management

To maintain a secure setup with GPG subkeys:

### 1. Regular Rotation Schedule

- Set calendar reminders for subkey expiration dates
- Create new subkeys before the old ones expire
- Plan for overlap to ensure smooth transitions

### 2. Secure Storage of Master Key

- Consider moving your master key to an offline storage medium after creating subkeys
- Options include:
  - Encrypted USB drive kept in a secure location
  - Air-gapped computer that never connects to the internet
  - Paper backup (for emergency recovery only)

### 3. Documentation and Tracking

Keep a secure record of:
- Which subkeys are on which devices
- Expiration dates for all subkeys
- Procedures for rotation and revocation

A simple encrypted text file can work well for this:

```bash
# Create a documentation file
nano ~/subkey-documentation.txt

# Add your documentation, then save and exit (Ctrl+X, then Y)

# Encrypt the documentation
gpg -c ~/subkey-documentation.txt

# Securely delete the unencrypted file
shred -u ~/subkey-documentation.txt
```

## Conclusion

You've now successfully created and learned how to manage GPG subkeys for use with `pass`. These subkeys will allow you to use `pass` on multiple devices while keeping your master key secure.

In the next guide, we'll cover how to export these subkeys to use on your other devices.

## Navigation

- [README](../README.md) - Wiki Home
- Previous: [Understanding GPG Subkeys for Pass](16_Understanding_GPG_Subkeys_for_Pass.md)
- Next: [Exporting GPG Subkeys for Other Devices](18_Exporting_GPG_Subkeys_for_Other_Devices.md)
- Related: [Exporting GPG Keys for Cross-Platform Use](../05_Exporting_GPG_Keys_for_Cross_Platform_Use.md)
