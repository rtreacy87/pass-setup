# Troubleshooting Guide

## Introduction

Even with careful setup, you may encounter issues with your cross-platform password management system. This guide covers common problems and their solutions, focusing on GPG key issues, Git synchronization problems, Pass command failures, and WSL-specific troubleshooting.

## GPG Key Issues

### Problem: "gpg: decryption failed: No secret key"

This error occurs when trying to decrypt a password, but the corresponding private key is not available.

#### Diagnosis

Check if your GPG key is properly imported:

```bash
gpg --list-secret-keys --keyid-format LONG
```

If your key is not listed, it's not properly imported.

#### Solutions

1. Import your private key:
   ```bash
   gpg --import private-key.asc
   ```

2. Check if the password was encrypted with a different key:
   ```bash
   gpg --list-packets ~/.password-store/path/to/password.gpg
   ```
   Look for the "keyid" in the output to see which key was used.

3. Re-encrypt the password with your current key:
   ```bash
   # Decrypt with the old key (if available)
   gpg -d ~/.password-store/path/to/password.gpg > temp_password.txt

   # Re-encrypt with your current key
   cat temp_password.txt | gpg -e -r YOUR_KEY_ID > ~/.password-store/path/to/password.gpg

   # Securely delete the temporary file
   shred -u temp_password.txt
   ```

### Problem: "gpg: public key decryption failed: Bad passphrase"

This error occurs when you enter the wrong passphrase for your GPG key.

#### Solutions

1. Try again with the correct passphrase
2. If you've forgotten your passphrase, you'll need to:
   - Generate a new GPG key
   - Re-initialize your password store with the new key
   - Restore passwords from a backup (if available)

### Problem: "gpg: agent_genkey failed: No pinentry"

This error occurs when GPG can't find the pinentry program to prompt for your passphrase.

#### Solutions

1. Install pinentry:

   On macOS:
   ```bash
   brew install pinentry-mac
   ```

   On WSL Ubuntu:
   ```bash
   sudo apt install -y pinentry-curses
   ```

2. Configure GPG to use the installed pinentry:
   ```bash
   echo "pinentry-program /usr/local/bin/pinentry-mac" >> ~/.gnupg/gpg-agent.conf
   # Or for WSL:
   # echo "pinentry-program /usr/bin/pinentry-curses" >> ~/.gnupg/gpg-agent.conf

   gpgconf --kill gpg-agent
   ```

### Problem: "gpg: WARNING: unsafe permissions"

This warning appears when your GPG directory has incorrect permissions.

#### Solution

Fix the permissions:

```bash
# Set correct ownership
chown -R $(whoami) ~/.gnupg

# Set correct permissions
chmod 700 ~/.gnupg
chmod 600 ~/.gnupg/*
```

## Git Synchronization Problems

### Problem: "git push" fails with "Permission denied (publickey)"

This error occurs when SSH authentication fails.

#### Diagnosis

Test your SSH connection:

```bash
ssh -T git@github.com
# Or for GitLab:
# ssh -T git@gitlab.com
```

#### Solutions

1. Verify your SSH key is added to the SSH agent:
   ```bash
   ssh-add -l
   ```

   If your key is not listed, add it:
   ```bash
   ssh-add ~/.ssh/id_ed25519
   ```

2. Verify your public key is added to your Git hosting service:
   - Check your SSH keys in GitHub/GitLab settings
   - Add your public key if it's not there:
     ```bash
     cat ~/.ssh/id_ed25519.pub
     ```

3. Check your remote URL format:
   ```bash
   pass git remote -v
   ```

   It should use SSH format (`git@github.com:username/repo.git`) not HTTPS format (`https://github.com/username/repo.git`).

   To change it:
   ```bash
   pass git remote set-url origin git@github.com:username/repo.git
   ```

### Problem: "git pull" fails with merge conflicts

This occurs when there are conflicting changes in your local and remote repositories.

#### Solutions

For encrypted files, you can't resolve conflicts in the usual way. Instead:

1. Choose which version to keep:

   To keep your local version:
   ```bash
   pass git checkout --ours path/to/conflicted/password.gpg
   pass git add path/to/conflicted/password.gpg
   pass git commit -m "Resolve conflict, keep local version"
   ```

   To keep the remote version:
   ```bash
   pass git checkout --theirs path/to/conflicted/password.gpg
   pass git add path/to/conflicted/password.gpg
   pass git commit -m "Resolve conflict, keep remote version"
   ```

2. If you need to see both versions:
   ```bash
   # Create a temporary branch with your version
   pass git checkout -b my-version

   # Go back to the main branch
   pass git checkout master

   # Create a temporary branch with their version
   pass git checkout -b their-version
   pass git pull

   # Now you can compare the files in both branches
   ```

### Problem: "git push" fails with "Updates were rejected"

This occurs when your local repository is behind the remote repository.

#### Solution

Pull before pushing:

```bash
pass git pull --rebase
pass git push
```

The `--rebase` flag applies your local changes on top of the remote changes.

## Pass Command Failures

### Problem: "pass: command not found"

This error occurs when the `pass` command is not in your PATH.

#### Solutions

1. Verify Pass is installed:

   On macOS:
   ```bash
   brew list pass
   ```

   On WSL Ubuntu:
   ```bash
   dpkg -l | grep pass
   ```

2. Find the location of the Pass executable:

   On macOS:
   ```bash
   which pass
   ```

   On WSL Ubuntu:
   ```bash
   which pass
   ```

3. Add the directory to your PATH if needed:
   ```bash
   echo 'export PATH="/usr/local/bin:$PATH"' >> ~/.bashrc
   source ~/.bashrc
   ```

### Problem: "Error: password store is empty"

This occurs when Pass can't find your password store.

#### Diagnosis

Check if the password store directory exists:

```bash
ls -la ~/.password-store
```

#### Solutions

1. If the directory doesn't exist, initialize a new password store:
   ```bash
   pass init YOUR_GPG_KEY_ID
   ```

2. If you're using a custom password store location, make sure the `PASSWORD_STORE_DIR` environment variable is set correctly:
   ```bash
   echo 'export PASSWORD_STORE_DIR=~/custom-password-store' >> ~/.bashrc
   source ~/.bashrc
   ```

3. If you're using Git, clone your repository:
   ```bash
   git clone git@github.com:username/password-store.git ~/.password-store
   ```

### Problem: "Error: gpg failed to encrypt password"

This occurs when GPG encryption fails.

#### Solutions

1. Verify your GPG key is valid:
   ```bash
   gpg --list-secret-keys --keyid-format LONG
   ```

2. Test GPG encryption:
   ```bash
   echo "test" | gpg -e -r YOUR_GPG_KEY_ID
   ```

3. Check if your key has expired:
   ```bash
   gpg --list-secret-keys --keyid-format LONG
   ```

   If it has expired, extend its validity:
   ```bash
   gpg --edit-key YOUR_GPG_KEY_ID
   # In the GPG prompt:
   # Type "expire"
   # Follow prompts to set a new expiration date
   # Type "save" to save changes
   ```

### Problem: "gpg: problem with the agent: No pinentry"

This error occurs when GPG can't find the pinentry program that handles password prompts.

#### Solutions

1. Force GPG to use terminal for password input:
   ```bash
   gpg --pinentry-mode loopback -c filename.txt
   ```

2. Configure GPG to always use loopback mode:
   ```bash
   echo "pinentry-mode loopback" >> ~/.gnupg/gpg.conf
   ```

3. For macOS users, install and configure pinentry-mac:
   ```bash
   brew install pinentry-mac
   echo "pinentry-program /usr/local/bin/pinentry-mac" >> ~/.gnupg/gpg-agent.conf
   gpgconf --kill gpg-agent
   ```

4. For Linux users, ensure pinentry is installed:
   ```bash
   # On Debian/Ubuntu
   sudo apt install pinentry-curses

   # On RHEL/CentOS
   sudo yum install pinentry
   ```

5. Restart the GPG agent:
   ```bash
   gpgconf --kill gpg-agent
   ```

### Problem: "gpg: Sorry, we are in batchmode - can't get input"

This error occurs when GPG is running in batch mode but requires user input (like a passphrase). This commonly happens when using Pass in scripts or when environment variables are incorrectly set.

#### Diagnosis

This typically happens when:
- You're running Pass in a non-interactive environment
- The `GPG_TTY` environment variable is not set correctly
- The `--batch` flag is being used (explicitly or implicitly)
- The `--no-tty` option is active

#### Solutions

1. Set the `GPG_TTY` environment variable:
   ```bash
   export GPG_TTY=$(tty)
   ```
   Add this to your `~/.bashrc` or `~/.zshrc` for a permanent fix:
   ```bash
   echo 'export GPG_TTY=$(tty)' >> ~/.bashrc
   source ~/.bashrc
   ```

2. Force GPG to use loopback mode for passphrase input:
   ```bash
   echo "pinentry-mode loopback" >> ~/.gnupg/gpg.conf
   ```

3. Use the `--no-batch` flag explicitly when calling GPG:
   ```bash
   gpg --no-batch -d ~/.password-store/path/to/password.gpg
   ```

4. Configure the GPG agent to use the correct pinentry program:
   ```bash
   # For terminal-based pinentry
   echo "pinentry-program /usr/bin/pinentry-curses" >> ~/.gnupg/gpg-agent.conf

   # For GUI-based pinentry (on systems with X11)
   echo "pinentry-program /usr/bin/pinentry-gtk-2" >> ~/.gnupg/gpg-agent.conf

   # For macOS
   echo "pinentry-program /usr/local/bin/pinentry-mac" >> ~/.gnupg/gpg-agent.conf

   # Restart the agent
   gpgconf --kill gpg-agent
   ```

5. If using Pass in a script, provide the passphrase via a file descriptor:
   ```bash
   # Create a named pipe
   mkfifo /tmp/gpg-pipe

   # In one terminal, provide the passphrase
   echo "YOUR_PASSPHRASE" > /tmp/gpg-pipe

   # In another terminal or script
   gpg --batch --passphrase-fd 3 -d file.gpg 3< /tmp/gpg-pipe

   # Clean up
   rm /tmp/gpg-pipe
   ```
   Note: This method should be used with caution as it can expose your passphrase.

6. Use a GPG agent with cached credentials:
   ```bash
   # Start the agent if not running
   gpg-agent --daemon

   # Unlock your key once interactively
   echo "test" | gpg -e -r YOUR_KEY_ID > /dev/null

   # Now batch operations should work without prompting
   ```

7. For WSL users, ensure proper TTY handling:
   ```bash
   # Add to your .bashrc or .zshrc
   if [[ -n "$WSL_DISTRO_NAME" ]]; then
     export GPG_TTY=$(tty)
     gpg-connect-agent updatestartuptty /bye > /dev/null
   fi
   ```

For more detailed solutions when encrypting files for transfer, see the [Exporting GPG Keys guide](05_Exporting_GPG_Keys_for_Cross_Platform_Use.md#troubleshooting-encryption-issues).

## WSL-Specific Troubleshooting

### Problem: Cannot access Windows files from WSL

This occurs when the Windows drives are not properly mounted in WSL.

#### Solutions

1. Check if Windows drives are mounted:
   ```bash
   ls /mnt/
   ```

   You should see your Windows drives (c, d, etc.).

2. If not mounted, try:
   ```bash
   sudo mkdir -p /mnt/c
   sudo mount -t drvfs C: /mnt/c
   ```

3. If that doesn't work, restart WSL:
   ```bash
   # In PowerShell (as Administrator):
   wsl --shutdown
   # Then restart WSL
   ```

### Problem: GPG in WSL can't access keys

This can occur due to permission issues in the WSL file system.

#### Solutions

1. Check permissions on your GPG directory:
   ```bash
   ls -la ~/.gnupg
   ```

2. Fix permissions:
   ```bash
   chmod 700 ~/.gnupg
   chmod 600 ~/.gnupg/*
   ```

3. If using GPG 2.1 or later, create or edit `~/.gnupg/gpg.conf`:
   ```bash
   echo "pinentry-mode loopback" >> ~/.gnupg/gpg.conf
   ```

### Problem: Git in WSL can't connect to remote repositories

This can occur due to SSH configuration issues in WSL.

#### Solutions

1. Verify SSH is installed in WSL:
   ```bash
   ssh -V
   ```

   If not installed:
   ```bash
   sudo apt install -y openssh-client
   ```

2. Set up SSH keys in WSL:
   ```bash
   ssh-keygen -t ed25519 -C "your_email@example.com"
   ```

3. Add your SSH key to the agent:
   ```bash
   eval "$(ssh-agent -s)"
   ssh-add ~/.ssh/id_ed25519
   ```

4. Add your public key to your Git hosting service:
   ```bash
   cat ~/.ssh/id_ed25519.pub
   # Copy this output to GitHub/GitLab
   ```

## Diagnostic Commands

Here are some useful commands for diagnosing issues:

### GPG Diagnostics

```bash
# List all GPG keys
gpg --list-keys --keyid-format LONG

# List secret keys
gpg --list-secret-keys --keyid-format LONG

# Check a specific key
gpg --edit-key YOUR_GPG_KEY_ID

# Test GPG encryption/decryption
echo "test" | gpg -e -r YOUR_GPG_KEY_ID | gpg -d

# Check GPG agent status
gpg-connect-agent 'getinfo version' /bye

# Kill and restart GPG agent
gpgconf --kill gpg-agent
```

### Pass Diagnostics

```bash
# Check Pass version
pass --version

# List all passwords
pass

# Check Pass environment variables
env | grep PASSWORD_STORE

# Run Pass with verbose output
pass -v show Email/gmail
```

### Git Diagnostics

```bash
# Check Git configuration
pass git config --list

# Check remote repository configuration
pass git remote -v

# Check repository status
pass git status

# View commit history
pass git log --oneline -10

# Check for uncommitted changes
pass git diff
```

### WSL Diagnostics

```bash
# Check WSL version
wsl --status

# Check Ubuntu version
lsb_release -a

# Check mounted drives
mount | grep drvfs

# Check network connectivity
ping github.com

# Check WSL environment variables
env
```

## Recovery Procedures

### Recovering from a Corrupted Password Store

If your password store becomes corrupted:

1. Create a backup of the current state:
   ```bash
   cp -r ~/.password-store ~/.password-store-backup
   ```

2. Try to fix Git issues:
   ```bash
   cd ~/.password-store
   git fsck
   ```

3. If that doesn't work, clone a fresh copy:
   ```bash
   mv ~/.password-store ~/.password-store-old
   git clone git@github.com:username/password-store.git ~/.password-store
   ```

### Recovering from a Lost GPG Key

If you've lost your GPG key but have a backup:

1. Import the backup key:
   ```bash
   gpg --import private-key-backup.asc
   ```

2. Import the trust database:
   ```bash
   gpg --import-ownertrust trust-backup.txt
   ```

If you've lost your GPG key and don't have a backup:

1. Generate a new key:
   ```bash
   gpg --full-generate-key
   ```

2. If you have access to the encrypted passwords but can't decrypt them, you'll need to:
   - Create a new password store
   - Manually recreate your passwords
   - Consider this a lesson in the importance of key backups

## Conclusion

This troubleshooting guide covers the most common issues you might encounter when using Pass across multiple systems. By understanding these problems and their solutions, you'll be better equipped to maintain your password management system and resolve issues when they arise.

Remember that prevention is better than cure:
- Regularly back up your GPG keys
- Keep your password store synchronized across devices
- Follow security best practices
- Test your setup regularly

If you encounter an issue not covered in this guide, the Pass documentation and community forums are excellent resources for additional help.

In the next guide, we'll cover how to extend your password management system to additional platforms beyond macOS and WSL Ubuntu.

## Navigation

- [README](README.md) - Wiki Home
- Previous: [Advanced Features and Extensions](12_Advanced_Features_and_Extensions.md)
- Next: [Extending to Additional Platforms](14_Extending_to_Additional_Platforms.md)
- Related:
  - [Git Integration for Pass](04_Git_Integration_for_Pass.md) - For Git-related issues
  - [Setting Up GPG Keys on macOS](02_Setting_Up_GPG_Keys_on_macOS.md) - For GPG-related issues
  - [Synchronization Workflow Between Systems](09_Synchronization_Workflow_Between_Systems.md) - For synchronization issues
  - [Setting Up WSL Ubuntu for Pass](06_Setting_Up_WSL_Ubuntu_for_Pass.md) - For WSL-specific issues
