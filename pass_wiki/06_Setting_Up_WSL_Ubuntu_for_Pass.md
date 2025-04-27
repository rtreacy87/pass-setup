# Setting Up WSL Ubuntu for Pass

## Introduction

Windows Subsystem for Linux (WSL) allows you to run a Linux environment directly on Windows without the need for a virtual machine. In this guide, we'll set up WSL with Ubuntu and prepare it for our password management system.

## Installing WSL and Ubuntu

### Step 1: Enable WSL Feature

First, you need to enable the WSL feature in Windows. Open PowerShell as Administrator (right-click on the Start menu and select "Windows PowerShell (Admin)") and run:

```powershell
dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart
```

This command enables the Windows Subsystem for Linux feature.

### Step 2: Enable Virtual Machine Platform

Next, enable the Virtual Machine Platform feature:

```powershell
dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart
```

### Step 3: Restart Your Computer

Restart your computer to complete the WSL installation.

### Step 4: Set WSL 2 as Default

After restarting, open PowerShell as Administrator again and run:

```powershell
wsl --set-default-version 2
```

This sets WSL 2 as the default version, which provides better performance and compatibility.

### Step 5: Install Ubuntu from Microsoft Store

1. Open the Microsoft Store app
2. Search for "Ubuntu"
3. Select "Ubuntu" (or "Ubuntu 20.04 LTS" or the latest version)
4. Click "Get" or "Install"
5. Wait for the download and installation to complete

### Step 6: Launch Ubuntu and Complete Setup

1. Click "Launch" in the Microsoft Store or find Ubuntu in your Start menu
2. Wait for the installation to complete (this may take a few minutes)
3. When prompted, create a new UNIX username and password
   - This can be different from your Windows username
   - Remember this password as you'll need it for sudo commands

You should now see a Ubuntu terminal prompt, indicating that WSL Ubuntu is installed and running.

## Updating Ubuntu

Before installing any software, update the package lists and upgrade existing packages:

```bash
sudo apt update && sudo apt upgrade -y
```

This command:
- `sudo apt update`: Updates the list of available packages
- `sudo apt upgrade -y`: Upgrades all installed packages to their latest versions
- The `-y` flag automatically answers "yes" to prompts

## Installing Required Software

Now, let's install the software needed for our password management system:

```bash
sudo apt install -y gnupg2 pass git
```

This command installs:
- `gnupg2`: The GNU Privacy Guard for encryption
- `pass`: The password manager
- `git`: The version control system

## Verifying Installations

Let's verify that everything was installed correctly:

```bash
# Check GPG version
gpg --version

# Check Pass version
pass --version

# Check Git version
git --version
```

You should see version information for each program, confirming they're installed correctly.

## Setting Up SSH for Git (Optional but Recommended)

If you plan to use Git with SSH authentication (recommended for security), you'll need to set up SSH keys:

### Step 1: Generate SSH Key

```bash
ssh-keygen -t ed25519 -C "your_email@example.com"
```

This command:
- Generates a new SSH key pair using the Ed25519 algorithm
- Associates it with your email address
- Prompts you for a file location (press Enter to accept the default)
- Asks for a passphrase (recommended for security)

### Step 2: Start the SSH Agent

```bash
eval "$(ssh-agent -s)"
```

This starts the SSH agent in the background.

### Step 3: Add Your SSH Key to the Agent

```bash
ssh-add ~/.ssh/id_ed25519
```

This adds your private key to the SSH agent.

### Step 4: Display Your Public Key

```bash
cat ~/.ssh/id_ed25519.pub
```

This displays your public key, which you'll need to add to your Git hosting service (GitHub, GitLab, etc.).

### Step 5: Add Your Public Key to Your Git Hosting Service

1. Copy the output from the previous command
2. Go to your Git hosting service website
3. Find the SSH keys section in your account settings
4. Add a new SSH key and paste your public key
5. Give it a descriptive name like "WSL Ubuntu"

## Configuring Git

Set up your Git identity:

```bash
git config --global user.name "Your Name"
git config --global user.email "your_email@example.com"
```

These commands configure Git with your name and email, which will be used in commit messages.

## Creating a Directory for Imported Keys

Create a directory to store the GPG keys you'll import:

```bash
mkdir -p ~/gpg-import
```

This creates a directory called `gpg-import` in your home directory.

## Accessing Windows Files from WSL

WSL can access your Windows files, which is useful for transferring files between systems:

### Accessing Windows Drives

Your Windows drives are mounted under `/mnt/` in WSL. For example:
- C: drive is at `/mnt/c/`
- D: drive is at `/mnt/d/`

To access your Windows Desktop:

```bash
cd /mnt/c/Users/YourWindowsUsername/Desktop/
```

Replace `YourWindowsUsername` with your actual Windows username.

### Copying Files from Windows to WSL

If you've transferred your GPG keys to your Windows system (e.g., on the Desktop), you can copy them to your WSL environment:

```bash
cp /mnt/c/Users/YourWindowsUsername/Desktop/gpg-keys.tar.gz.gpg ~/gpg-import/
```

This copies the encrypted archive from your Windows Desktop to the `gpg-import` directory in your WSL home directory.

## Conclusion

Congratulations! You've successfully:
- Installed WSL with Ubuntu on your Windows system
- Updated the system and installed required software
- Set up SSH for secure Git authentication
- Configured Git with your identity
- Created a directory for importing GPG keys
- Learned how to access Windows files from WSL

Your Windows system is now prepared for importing your GPG keys and setting up Pass. In the next guide, we'll import the GPG keys you exported from your macOS system.

Remember that WSL provides a full Linux environment, so all the Linux commands you've learned for macOS will work the same way in WSL Ubuntu.
