# Importing GPG Keys to WSL Ubuntu

## Introduction

Now that you have WSL Ubuntu set up on your Windows system, it's time to import the GPG keys you exported from your macOS system. This will allow you to decrypt and access your passwords on your Windows machine.

## Prerequisites

Before proceeding, make sure you have:
- Completed the setup of WSL Ubuntu as described in the previous guide
- Transferred your exported GPG keys to your Windows system
- Located where these files are stored on your Windows system

## Accessing Your Exported Keys

First, let's make sure we can access the exported keys from WSL Ubuntu.

### If Keys are on a USB Drive

If your keys are on a USB drive, it should be automatically mounted in WSL under `/mnt/`. To find it:

```bash
ls /mnt/
```

Look for a drive letter that corresponds to your USB drive (e.g., `/mnt/d/` or `/mnt/e/`).

```bash
# Navigate to your USB drive
cd /mnt/d/
# List files to find your keys
ls -la
```

### If Keys are in a Windows Directory

If your keys are in a Windows directory (e.g., Downloads or Desktop):

```bash
# Navigate to your Windows user directory
cd /mnt/c/Users/YourWindowsUsername/

# Go to Desktop or Downloads
cd Desktop
# or
cd Downloads

# List files to find your keys
ls -la
```

Replace `YourWindowsUsername` with your actual Windows username.

## Preparing the Keys for Import

If your keys are in an encrypted archive (as recommended in the export guide), you'll need to decrypt it first.

### Step 1: Copy the Encrypted Archive to Your WSL Home Directory

```bash
# Assuming the file is on your Desktop
cp /mnt/c/Users/YourWindowsUsername/Desktop/gpg-keys.tar.gz.gpg ~/gpg-import/
```

This copies the encrypted archive to the `gpg-import` directory we created in the previous guide.

### Step 2: Navigate to the Import Directory

```bash
cd ~/gpg-import
```

### Step 3: Decrypt the Archive

```bash
gpg -d gpg-keys.tar.gz.gpg > gpg-keys.tar.gz
```

This command:
- Prompts you for the password you used to encrypt the archive
- Decrypts the archive and saves it as `gpg-keys.tar.gz`

### Step 4: Extract the Archive

```bash
tar -xzf gpg-keys.tar.gz
```

This extracts the contents of the archive, which should include:
- `public-key.asc`: Your public GPG key
- `private-key.asc`: Your private GPG key
- `trust.txt`: Your GPG trust database

Verify that the files were extracted correctly:

```bash
ls -la
```

You should see the three files listed above.

## Importing Your GPG Keys

Now let's import your keys into the GPG keyring on your WSL Ubuntu system.

### Step 1: Import the Public Key

```bash
gpg --import public-key.asc
```

This command imports your public key into your GPG keyring. You should see output like:
```
gpg: key 1A2B3C4D5E6F7G8H: public key "Your Name <your.email@example.com>" imported
gpg: Total number processed: 1
gpg:               imported: 1
```

### Step 2: Import the Private Key

```bash
gpg --import private-key.asc
```

This command imports your private key. You should see output like:
```
gpg: key 1A2B3C4D5E6F7G8H: secret key imported
gpg: Total number processed: 1
gpg:              unchanged: 1
gpg:       secret keys read: 1
gpg:   secret keys imported: 1
```

You may be prompted for your GPG key passphrase during this step.

### Step 3: Import the Trust Database

```bash
gpg --import-ownertrust trust.txt
```

This command imports your trust database. You should see output like:
```
gpg: inserting ownertrust of 6
```

## Verifying the Imported Keys

Let's verify that your keys were imported correctly:

```bash
gpg --list-secret-keys --keyid-format LONG
```

This should display your imported key, showing both the public and private key components:
```
/home/yourusername/.gnupg/pubring.kbx
-------------------------------------
sec   rsa4096/1A2B3C4D5E6F7G8H 2023-01-01 [SC] [expires: 2025-01-01]
      ABCDEF0123456789ABCDEF0123456789ABCDEF01
uid                 [ultimate] Your Name <your.email@example.com>
ssb   rsa4096/8H7G6F5E4D3C2B1A 2023-01-01 [E] [expires: 2025-01-01]
```

Make note of your key ID (in this example, `1A2B3C4D5E6F7G8H`), as you'll need it when setting up Pass.

## Configuring GPG Agent

Let's configure the GPG agent to cache your passphrase, so you don't have to enter it every time you use Pass:

### Step 1: Create or Edit the GPG Agent Configuration

```bash
mkdir -p ~/.gnupg
touch ~/.gnupg/gpg-agent.conf
nano ~/.gnupg/gpg-agent.conf
```

These commands:
1. Create the `.gnupg` directory if it doesn't exist
2. Create or open the `gpg-agent.conf` file
3. Open the file in the nano text editor

### Step 2: Add Configuration Settings

Add the following lines to the file:

```
default-cache-ttl 3600
max-cache-ttl 86400
```

These settings:
- `default-cache-ttl 3600`: Cache your passphrase for 1 hour (3600 seconds)
- `max-cache-ttl 86400`: Set the maximum cache time to 24 hours (86400 seconds)

Save the file by pressing `Ctrl+O`, then `Enter`, and exit nano with `Ctrl+X`.

### Step 3: Restart the GPG Agent

```bash
gpgconf --kill gpg-agent
```

This command stops the current GPG agent process, which will automatically restart when needed.

## Testing Your GPG Setup

Let's make sure everything is working correctly:

```bash
echo "Test message" | gpg --encrypt --armor --recipient your.email@example.com | gpg --decrypt
```

This command:
1. Creates a test message
2. Encrypts it using your public key
3. Immediately decrypts it using your private key

You should be prompted for your passphrase (unless the agent has cached it), and then see "Test message" as the output.

## Cleaning Up

After successfully importing your keys, you should securely delete the exported key files:

```bash
# Securely delete the files
shred -u public-key.asc private-key.asc trust.txt gpg-keys.tar.gz

# If shred is not available, you can use:
rm public-key.asc private-key.asc trust.txt gpg-keys.tar.gz
```

The `shred` command overwrites the files before deleting them, making recovery more difficult.

## Conclusion

Congratulations! You've successfully:
- Accessed your exported GPG keys in WSL Ubuntu
- Decrypted and extracted the keys (if they were in an encrypted archive)
- Imported your public and private keys into your GPG keyring
- Imported your trust database
- Configured the GPG agent
- Tested your GPG setup
- Cleaned up sensitive files

Your GPG keys are now set up on your WSL Ubuntu system, and you're ready to configure Pass to use these keys. In the next guide, we'll set up Pass on WSL Ubuntu and connect it to your existing password store.

Remember that your private key is extremely sensitive. Now that it's imported into your GPG keyring, you should ensure that your Windows system is secured with a strong password and disk encryption if possible.

## Navigation

- [README](README.md) - Wiki Home
- Previous: [Setting Up WSL Ubuntu for Pass](06_Setting_Up_WSL_Ubuntu_for_Pass.md)
- Next: [Setting Up Pass on WSL Ubuntu](08_Setting_Up_Pass_on_WSL_Ubuntu.md)
- Prerequisites:
  - [Setting Up GPG Keys on macOS](02_Setting_Up_GPG_Keys_on_macOS.md)
  - [Exporting GPG Keys for Cross-Platform Use](05_Exporting_GPG_Keys_for_Cross_Platform_Use.md)
