# Installing and Configuring Pass on Windows Subsystem for Linux

**Difficulty Level:** Beginner to Intermediate  
**Time to Complete:** 60 minutes

## Introduction

In this guide, we'll install and configure the `pass` password manager on Windows Subsystem for Linux (WSL). We'll also import your GPG key from macOS and connect to the same Git repository, setting the foundation for synchronized password management.

## What is WSL?

Windows Subsystem for Linux (WSL) is a feature of Windows that allows you to run a Linux environment directly on Windows without the need for a virtual machine. It's perfect for developers and command-line users who need Linux tools on a Windows machine.

## Prerequisites

Before starting, make sure you have:
- Windows 10 version 1903 or higher, or Windows 11
- WSL installed (preferably WSL 2)
- Ubuntu or another Linux distribution installed in WSL
- A private Git repository (from the previous guide)
- Pass configured on macOS (from the previous guide)
- Your GPG key from macOS

## Step 1: Install WSL (if not already installed)

If you don't already have WSL installed, follow these steps:

1. Open PowerShell as Administrator (right-click on the Start menu and select "Windows PowerShell (Admin)")
2. Run the following command:

```powershell
wsl --install
```

This command:
- Enables the WSL feature
- Installs the WSL 2 Linux kernel
- Sets WSL 2 as the default
- Installs Ubuntu as the default Linux distribution

After installation, restart your computer when prompted. When you first launch Ubuntu, you'll be asked to create a username and password.

## Step 2: Install Required Software in WSL

Open your WSL terminal (Ubuntu) and install the necessary packages:

```bash
# Update package lists
sudo apt update

# Upgrade existing packages
sudo apt upgrade -y

# Install required packages
sudo apt install -y gnupg pass git
```

What these commands do:
- `sudo apt update` refreshes the package lists
- `sudo apt upgrade -y` updates all installed packages
- `sudo apt install -y gnupg pass git` installs GPG, Pass, and Git

## Step 3: Transfer Your GPG Key from macOS to WSL

To use the same passwords on both systems, you need to use the same GPG key. We'll export the key from macOS and import it into WSL.

### On macOS: Export Your GPG Key

```bash
# Create a directory for the export
mkdir -p ~/gpg-transfer

# Export your public key
gpg --export --armor YOUR_KEY_ID > ~/gpg-transfer/public.key

# Export your private key (keep this secure!)
gpg --export-secret-keys --armor YOUR_KEY_ID > ~/gpg-transfer/private.key

# Export your trust database
gpg --export-ownertrust > ~/gpg-transfer/trustdb.txt
```

Replace `YOUR_KEY_ID` with your actual GPG key ID from the previous guide.

These commands:
- Create a directory to store the exported keys
- Export your public key in ASCII-armored format
- Export your private key in ASCII-armored format
- Export your trust database

### Transfer the Keys to Windows

There are several ways to transfer the keys:

#### Option 1: Using a USB Drive

1. Copy the files to a USB drive
2. Connect the USB drive to your Windows computer
3. Copy the files to a location on your Windows system

#### Option 2: Using Secure Copy (SCP)

If your Mac and Windows computer are on the same network:

```bash
# On macOS, find your IP address
ifconfig | grep "inet " | grep -v 127.0.0.1

# Use SCP to copy the files (run this on macOS)
scp -r ~/gpg-transfer username@windows-ip:/path/to/destination
```

Replace `username` with your Windows username and `windows-ip` with your Windows computer's IP address.

#### Option 3: Using a Secure Cloud Service

1. Encrypt the directory before uploading:
   ```bash
   # On macOS
   tar -czf gpg-transfer.tar.gz ~/gpg-transfer
   gpg --symmetric gpg-transfer.tar.gz
   ```
2. Upload the encrypted file to a cloud service
3. Download it on your Windows computer
4. Decrypt it:
   ```bash
   # On Windows
   gpg --decrypt gpg-transfer.tar.gz.gpg > gpg-transfer.tar.gz
   ```

### On WSL: Import Your GPG Key

Once you have the files on your Windows system, you can access them from WSL through the `/mnt` directory. For example, if your files are in `C:\Users\YourName\Documents\gpg-transfer`, you can access them in WSL at `/mnt/c/Users/YourName/Documents/gpg-transfer`.

```bash
# Navigate to where your keys are stored
cd /mnt/c/Users/YourName/Documents/gpg-transfer

# Import your public key
gpg --import public.key

# Import your private key
gpg --import private.key

# Import your trust database
gpg --import-ownertrust trustdb.txt
```

These commands:
- Navigate to the directory containing your exported keys
- Import your public key into the GPG keyring
- Import your private key into the GPG keyring
- Import your trust database

### Verify the Import

```bash
# List your GPG keys
gpg --list-secret-keys --keyid-format LONG
```

You should see the same key that you have on your macOS system.

## Step 4: Initialize Pass with Your GPG Key

Now that your GPG key is imported, you can initialize Pass:

```bash
# Initialize Pass with your GPG key ID
pass init YOUR_KEY_ID
```

Replace `YOUR_KEY_ID` with your actual GPG key ID.

This command:
- Creates a `.password-store` directory in your home folder
- Sets up Pass to use your GPG key for encryption
- Creates a `.gpg-id` file containing your key ID

## Step 5: Set Up SSH for GitHub Access

To connect to your GitHub repository, you need to set up SSH authentication in WSL:

```bash
# Check if you already have SSH keys
ls -la ~/.ssh

# If you don't see id_rsa and id_rsa.pub, generate new keys:
ssh-keygen -t ed25519 -C "your_email@example.com"

# Start the SSH agent
eval "$(ssh-agent -s)"

# Add your key to the agent
ssh-add ~/.ssh/id_ed25519

# Display your public key (copy this to add to GitHub)
cat ~/.ssh/id_ed25519.pub
```

Now add this SSH key to your GitHub account:

1. Go to GitHub and log in
2. Click your profile picture in the top-right corner
3. Select "Settings"
4. In the left sidebar, click "SSH and GPG keys"
5. Click "New SSH key"
6. Title: "My WSL"
7. Key type: Authentication Key
8. Key: Paste your public key
9. Click "Add SSH key"

Test your SSH connection:

```bash
# Test connection to GitHub
ssh -T git@github.com
```

You should see a message confirming successful authentication.

## Step 6: Clone Your Password Store Repository

Instead of initializing a new Git repository, we'll clone the existing one from GitHub:

```bash
# Backup the initial Pass directory (just in case)
mv ~/.password-store ~/.password-store.bak

# Clone your repository
git clone git@github.com:yourusername/password-store.git ~/.password-store
```

Replace `yourusername` with your GitHub username.

These commands:
- Back up the initial Pass directory created when you initialized Pass
- Clone your GitHub repository to the `.password-store` directory

## Step 7: Test Your Setup

Let's verify that everything is working correctly:

```bash
# List all passwords
pass

# Try to view a password (if you have any)
pass test/example
```

You should be able to see the same passwords that you have on your macOS system.

## Step 8: Configure Git to Auto-Commit Password Changes

Just like on macOS, we'll configure Pass to automatically commit changes to Git:

```bash
# Enable Git auto-commit
cd ~/.password-store
pass git config --local pass.git true
```

This tells Pass to automatically run Git commands when you make changes to your password store.

## Common Commands

Here are some common Pass commands you'll use (same as on macOS):

```bash
# Add a new password
pass insert category/name

# Generate a random password
pass generate category/name 15  # Creates a 15-character random password

# Show a password
pass category/name

# Edit a password
pass edit category/name

# Remove a password
pass rm category/name

# List all passwords
pass
```

## Troubleshooting

### "gpg: decryption failed: No secret key"

This means GPG can't find the private key needed to decrypt the password. Make sure:
- You've properly imported your GPG key from macOS
- The key ID used to initialize Pass matches your imported key

### Git Push/Pull Errors

If you encounter errors with Git operations:
- Check that your SSH key is properly set up
- Verify that the remote repository URL is correct
- Make sure you have the right permissions on GitHub

### WSL File Permission Issues

WSL can sometimes have issues with file permissions. If you encounter permission errors:

```bash
# Fix permissions on the .password-store directory
chmod -R 700 ~/.password-store
```

This sets the permissions so that only your user can read, write, and execute files in the password store.

## Next Steps

Now that you have Pass set up on both macOS and WSL, the next step is to implement Git hooks to automate the synchronization process.

---

**Prerequisites**: 
- [Understanding Automated Password Synchronization](01_Introduction_to_Automated_Password_Synchronization.md)
- [Creating and Configuring Your Password Repository](02_Setting_Up_Git_Repository.md)
- [Installing and Configuring Pass on macOS](03_Configuring_Pass_on_macOS.md)

**Next**: [Using Git Hooks to Automate Password Synchronization](05_Understanding_Git_Hooks.md)

**Related**:
- [Building Robust Synchronization Scripts](06_Creating_Synchronization_Scripts.md)
- [Setting Up Scheduled Password Synchronization on WSL](08_Scheduled_Synchronization_on_WSL.md)
