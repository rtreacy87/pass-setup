# Creating and Configuring Your Password Repository

**Difficulty Level:** Beginner to Intermediate  
**Time to Complete:** 30 minutes

## Introduction

In this guide, we'll set up a Git repository to store your encrypted passwords. This repository will serve as the central hub that both your macOS and WSL systems will synchronize with.

## What is Git?

Git is a version control system that tracks changes to files over time. It's perfect for our password synchronization system because it:

- Records all changes to your password store
- Allows synchronization between multiple computers
- Provides a history of changes (useful if you need to recover old passwords)
- Handles merging changes from different sources

## Choosing a Git Hosting Service

You have several options for hosting your Git repository:

### 1. GitHub (Recommended for Beginners)

**Pros:**
- Easy to set up and use
- Reliable service with good uptime
- Free for private repositories
- Web interface for emergency access

**Cons:**
- Hosted by a third party (though your passwords remain encrypted)
- Requires internet access for synchronization

### 2. GitLab

**Pros:**
- Similar to GitHub with free private repositories
- Can be self-hosted if desired
- More built-in CI/CD features (not needed for our use case)

**Cons:**
- Slightly more complex interface than GitHub

### 3. Self-Hosted Git Server

**Pros:**
- Complete control over your data
- Can work within your local network without internet

**Cons:**
- Requires setting up and maintaining a server
- More complex to configure
- You're responsible for backups and reliability

For this guide, we'll use GitHub as it's the simplest option, but the process is similar for other services.

## Creating a Private GitHub Repository

### Step 1: Create a GitHub Account (if you don't have one)

1. Go to [GitHub's signup page](https://github.com/signup)
2. Follow the instructions to create an account
3. Verify your email address

### Step 2: Create a New Private Repository

1. Log in to GitHub
2. Click the "+" icon in the top-right corner
3. Select "New repository"
4. Fill in the repository details:
   - Name: `password-store` (or any name you prefer)
   - Description: "My encrypted password store" (optional)
   - Visibility: **Private** (very important!)
   - Initialize with a README: Yes
5. Click "Create repository"

![GitHub New Repository](https://i.imgur.com/example-image.png)

### Step 3: Set Up SSH Authentication

For secure access to your repository, we'll use SSH keys instead of passwords.

#### On macOS:

```bash
# Check if you already have SSH keys
ls -la ~/.ssh

# If you see id_rsa and id_rsa.pub, you already have keys
# If not, generate new keys:
ssh-keygen -t ed25519 -C "your_email@example.com"

# Press Enter to accept the default file location
# Enter a secure passphrase when prompted (and remember it!)

# Start the SSH agent
eval "$(ssh-agent -s)"

# Add your key to the agent
ssh-add ~/.ssh/id_ed25519

# Copy your public key to clipboard
cat ~/.ssh/id_ed25519.pub | pbcopy
```

What these commands do:
- `ls -la ~/.ssh` lists all files in your SSH directory to check for existing keys
- `ssh-keygen` creates a new SSH key pair (private and public)
- `eval "$(ssh-agent -s)"` starts the SSH agent, which manages your keys
- `ssh-add` adds your key to the agent so you don't have to enter your passphrase every time
- `cat ~/.ssh/id_ed25519.pub | pbcopy` displays your public key and copies it to clipboard

#### On Windows/WSL:

```bash
# Check if you already have SSH keys
ls -la ~/.ssh

# If you see id_rsa and id_rsa.pub, you already have keys
# If not, generate new keys:
ssh-keygen -t ed25519 -C "your_email@example.com"

# Press Enter to accept the default file location
# Enter a secure passphrase when prompted (and remember it!)

# Start the SSH agent
eval "$(ssh-agent -s)"

# Add your key to the agent
ssh-add ~/.ssh/id_ed25519

# Display your public key (copy it manually)
cat ~/.ssh/id_ed25519.pub
```

### Step 4: Add Your SSH Key to GitHub

1. Go to GitHub and log in
2. Click your profile picture in the top-right corner
3. Select "Settings"
4. In the left sidebar, click "SSH and GPG keys"
5. Click "New SSH key"
6. Title: "My Mac" or "My WSL" (depending on which system you're setting up)
7. Key type: Authentication Key
8. Key: Paste your public key (from clipboard or copied manually)
9. Click "Add SSH key"

### Step 5: Test Your SSH Connection

```bash
# Test connection to GitHub
ssh -T git@github.com

# You might see a warning about authenticity of host - type 'yes'
# If successful, you'll see: "Hi username! You've successfully authenticated..."
```

This command attempts to connect to GitHub using SSH. The first time you connect, you'll be asked to verify GitHub's identity - this is normal and you should type 'yes'.

## Repository Structure and Organization

Your password store will have a structure like this:

```
password-store/
├── .git/               # Git repository data (hidden)
├── .gpg-id             # Your GPG key ID
├── email/
│   ├── personal.gpg    # Encrypted password file
│   └── work.gpg        # Encrypted password file
├── finance/
│   ├── bank.gpg
│   └── credit-cards.gpg
└── README.md           # Information about your repository
```

This structure will be created automatically when we set up Pass in the next guides.

## Security Considerations

### Repository Access

- **Never** make your password repository public
- Only grant access to people you completely trust
- Regularly review who has access to your repository
- Enable two-factor authentication on your GitHub account

### SSH Key Security

- Use a strong passphrase for your SSH key
- Never share your private key (the file without .pub extension)
- If you suspect your key has been compromised, generate a new one immediately

### Emergency Access

Consider what happens if you lose access to your computers:

- Keep an encrypted backup of your GPG and SSH keys in a secure location
- Document the recovery process for yourself or trusted individuals

## Next Steps

Now that you have a private Git repository set up with SSH authentication, you're ready to configure Pass on your macOS system.

---

**Prerequisites**: [Understanding Automated Password Synchronization](01_Introduction_to_Automated_Password_Synchronization.md)

**Next**: [Installing and Configuring Pass on macOS](03_Configuring_Pass_on_macOS.md)

**Related**:
- [Installing and Configuring Pass on WSL](04_Configuring_Pass_on_WSL.md)
- [Security Best Practices for Automated Synchronization](11_Security_Best_Practices.md)
