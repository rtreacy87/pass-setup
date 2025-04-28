# Installing and Configuring Pass on macOS

**Difficulty Level:** Beginner to Intermediate  
**Time to Complete:** 45 minutes

## Introduction

In this guide, we'll install and configure the `pass` password manager on your macOS system. We'll set up GPG for encryption and integrate it with the Git repository we created in the previous guide.

## What is Pass?

Pass (short for "password store") is a command-line password manager that:
- Stores each password in an encrypted file
- Uses GPG (GNU Privacy Guard) for encryption
- Organizes passwords in a directory structure
- Integrates with Git for version control

## Prerequisites

Before starting, make sure you have:
- macOS 10.12 or newer
- Administrator access to your Mac
- A private Git repository (from the previous guide)
- SSH key set up for GitHub access

## Step 1: Install Required Software

We'll use Homebrew to install the necessary software. Homebrew is a package manager for macOS that makes it easy to install command-line tools.

### Installing Homebrew

If you don't already have Homebrew installed:

```bash
# Install Homebrew
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

This command downloads and runs the Homebrew installation script. It will:
1. Ask for your password (this is your Mac login password)
2. Install the necessary files
3. Set up Homebrew on your system

### Installing GPG, Pass, and Git

```bash
# Update Homebrew
brew update

# Install GPG
brew install gnupg

# Install Pass
brew install pass

# Install Git (if not already installed)
brew install git
```

What these commands do:
- `brew update` refreshes Homebrew's package list
- `brew install gnupg` installs GNU Privacy Guard for encryption
- `brew install pass` installs the Pass password manager
- `brew install git` installs Git for version control

## Step 2: Set Up GPG Keys

GPG keys are used to encrypt and decrypt your passwords. You need to create a key pair if you don't already have one.

### Check for Existing GPG Keys

```bash
# List existing GPG keys
gpg --list-secret-keys --keyid-format LONG
```

If you see output that includes something like `sec   rsa4096/3AA5C34371567BD2`, you already have a GPG key. The part after `rsa4096/` is your key ID.

### Create a New GPG Key (if needed)

If you don't have a GPG key, create one:

```bash
# Generate a new GPG key
gpg --full-generate-key
```

Follow the prompts:
1. Key type: Select `RSA and RSA` (usually option 1)
2. Key size: Enter `4096` for maximum security
3. Expiration: Choose how long the key should be valid (you can choose `0` for no expiration, but it's better security practice to set an expiration)
4. Real name: Enter your full name
5. Email address: Enter your email address
6. Comment: Optional, you can leave this blank
7. Passphrase: Enter a strong passphrase (and don't forget it!)

This process creates a public and private key pair. The public key is used to encrypt data, and the private key (protected by your passphrase) is used to decrypt it.

### Find Your GPG Key ID

```bash
# List your keys to find your key ID
gpg --list-secret-keys --keyid-format LONG
```

Look for a line like `sec   rsa4096/3AA5C34371567BD2`. The part after `rsa4096/` is your key ID. Write this down or copy it - you'll need it in the next step.

## Step 3: Initialize Pass

Now we'll set up Pass to use your GPG key for encryption.

```bash
# Initialize Pass with your GPG key ID
pass init YOUR_KEY_ID
```

Replace `YOUR_KEY_ID` with the key ID you found in the previous step. For example:

```bash
pass init 3AA5C34371567BD2
```

This command:
1. Creates a `.password-store` directory in your home folder
2. Sets up Pass to use your GPG key for encryption
3. Creates a `.gpg-id` file containing your key ID

## Step 4: Configure Git Integration

Now we'll connect your password store to the Git repository you created earlier.

```bash
# Navigate to your password store
cd ~/.password-store

# Initialize Git repository
git init

# Add your remote repository
git remote add origin git@github.com:yourusername/password-store.git

# Add all files
git add .

# Commit the initial setup
git commit -m "Initialize password store"

# Push to GitHub
git push -u origin master
```

What these commands do:
- `cd ~/.password-store` changes to your password store directory
- `git init` creates a new Git repository in this directory
- `git remote add origin...` connects to your GitHub repository
- `git add .` stages all files for commit
- `git commit -m "..."` creates a commit with the staged files
- `git push -u origin master` uploads your files to GitHub and sets up tracking

If you get an error about "master" vs "main", try this instead for the last command:
```bash
git push -u origin main
```

## Step 5: Test Basic Usage

Let's test that everything is working by adding a test password:

```bash
# Add a test password
pass insert test/example

# This will prompt you to enter a password
# Type a test password and press Enter
```

Now let's see if it was encrypted and stored correctly:

```bash
# List all passwords
pass

# Show the test password
pass test/example
```

The `pass` command should show your password store structure, and `pass test/example` should display the password you just entered.

## Step 6: Test Git Integration

Let's make sure the Git integration is working:

```bash
# Check Git status
cd ~/.password-store
git status

# You should see that the new password file is untracked
# Let's add and commit it
git add .
git commit -m "Add test password"
git push
```

Now go to your GitHub repository in a web browser and verify that the commit appears. You should see a new encrypted file at `test/example.gpg`.

## Step 7: Configure Git to Auto-Commit Password Changes

Pass can automatically commit changes to Git when you add or modify passwords:

```bash
# Enable Git auto-commit
pass git config --local pass.git true
```

This tells Pass to automatically run Git commands when you make changes to your password store.

## Common Commands

Here are some common Pass commands you'll use:

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
- You're using the same GPG key that was used to initialize Pass
- Your GPG key is properly imported on this system

### Git Push/Pull Errors

If you encounter errors with Git operations:
- Check that your SSH key is properly set up
- Verify that the remote repository URL is correct
- Make sure you have the right permissions on GitHub

### "pass: password store is empty"

If you see this message, it means either:
- You haven't added any passwords yet
- You're looking at a different password store than expected

## Next Steps

Now that you have Pass set up on your macOS system, the next step is to configure it on your WSL system. Then we'll set up the automation to keep them in sync.

---

**Prerequisites**: 
- [Understanding Automated Password Synchronization](01_Introduction_to_Automated_Password_Synchronization.md)
- [Creating and Configuring Your Password Repository](02_Setting_Up_Git_Repository.md)

**Next**: [Installing and Configuring Pass on WSL](04_Configuring_Pass_on_WSL.md)

**Related**:
- [Using Git Hooks to Automate Password Synchronization](05_Understanding_Git_Hooks.md)
- [Building Robust Synchronization Scripts](06_Creating_Synchronization_Scripts.md)
