# Setting Up Pass on WSL Ubuntu

## Introduction

Now that you have your GPG keys imported into your WSL Ubuntu system, it's time to set up Pass and connect it to your existing password store. This will allow you to access and manage your passwords from your Windows machine.

## Prerequisites

Before proceeding, make sure you have:
- Completed the setup of WSL Ubuntu as described in the previous guides
- Successfully imported your GPG keys
- Your GPG key ID ready (from the previous guide)

## Installing Pass

If you haven't already installed Pass during the WSL Ubuntu setup, you can do so now:

```bash
sudo apt update
sudo apt install -y pass
```

These commands:
1. Update the package lists
2. Install Pass

Verify the installation:

```bash
pass --version
```

You should see output showing the version of Pass installed, something like:
```
pass 1.7.3
```

## Initializing Pass with Your GPG Key

Now, initialize Pass with the same GPG key you used on your macOS system:

```bash
pass init "YOUR_GPG_KEY_ID"
```

Replace `YOUR_GPG_KEY_ID` with your actual GPG key ID (e.g., `1A2B3C4D5E6F7G8H`).

This command:
- Creates a new password store in `~/.password-store/`
- Configures it to use your imported GPG key for encryption

You should see a message like:
```
Password store initialized for 1A2B3C4D5E6F7G8H
```

## Connecting to Your Existing Password Store

If you've been using Pass on macOS with Git integration, you can clone your existing password store to replace the empty one we just created.

### Step 1: Remove the Empty Password Store

First, let's remove the empty password store that was just created:

```bash
rm -rf ~/.password-store
```

This command removes the directory and all its contents.

### Step 2: Clone Your Password Repository

Now, clone your existing password repository:

```bash
git clone git@github.com:yourusername/password-store.git ~/.password-store
```

Replace `git@github.com:yourusername/password-store.git` with the actual URL of your Git repository.

If you're using HTTPS instead of SSH:

```bash
git clone https://github.com/yourusername/password-store.git ~/.password-store
```

This command:
- Connects to your Git repository
- Downloads all the files
- Places them in the `~/.password-store/` directory

If you're using SSH authentication, you might be prompted to confirm the host's authenticity the first time you connect.

### Step 3: Verify the Clone

Let's make sure your password store was cloned correctly:

```bash
ls -la ~/.password-store
```

You should see your password directory structure, including the `.git` directory and your encrypted password files (`.gpg` files).

## Testing Pass

Now let's test that Pass is working correctly with your imported GPG key and cloned password store:

```bash
pass
```

This command should display your password store structure, showing all your categories and password entries (without revealing the actual passwords).

Try viewing a password:

```bash
pass Email/gmail
```

Replace `Email/gmail` with an actual path to one of your passwords.

You should be prompted for your GPG passphrase (unless it's cached by the agent), and then see your password displayed.

## Setting Up Git Integration

If you're using Git for synchronization, you should configure Pass to use Git automatically:

### Step 1: Enable Git Auto-Commit

Edit your `.bashrc` file:

```bash
nano ~/.bashrc
```

Add the following line at the end of the file:

```bash
export PASSWORD_STORE_ENABLE_GIT=true
```

This environment variable tells Pass to automatically commit changes to your password store.

Save the file by pressing `Ctrl+O`, then `Enter`, and exit nano with `Ctrl+X`.

### Step 2: Apply the Changes

To apply the changes without restarting your terminal:

```bash
source ~/.bashrc
```

### Step 3: Configure Git in the Password Store

Navigate to your password store and configure Git:

```bash
cd ~/.password-store

# Configure Git to use your name and email
git config user.name "Your Name"
git config user.email "your.email@example.com"
```

These commands set your name and email for commits made in this repository.

## Synchronization Workflow

Now that you have Pass set up on both your macOS and WSL Ubuntu systems, you'll want to keep your passwords synchronized between them.

### Before Making Changes

Always pull the latest changes before making any modifications:

```bash
cd ~/.password-store
git pull
```

Or, using Pass's Git integration:

```bash
pass git pull
```

### After Making Changes

If you've enabled auto-commit as described above, changes will be committed automatically. To push your changes to the remote repository:

```bash
pass git push
```

This makes your changes available to your other devices.

## Basic Password Management Operations

Here's a quick refresher on basic Pass commands:

### Viewing a Password

```bash
pass Category/service-name
```

### Copying a Password to Clipboard

```bash
pass -c Category/service-name
```

### Adding a New Password

```bash
pass insert Category/service-name
```

### Generating a Random Password

```bash
pass generate Category/service-name 15
```

The number `15` specifies the length of the generated password.

### Editing a Password

```bash
pass edit Category/service-name
```

### Removing a Password

```bash
pass rm Category/service-name
```

## Accessing Pass from Windows

While Pass runs in WSL Ubuntu, you might want to access it more conveniently from Windows. Here are a few options:

### Option 1: Create a Windows Shortcut to WSL Ubuntu

1. Create a new shortcut on your desktop
2. Set the target to: `wsl.exe -d Ubuntu`
3. Give it a name like "Ubuntu Terminal"

This shortcut will open a WSL Ubuntu terminal where you can use Pass commands.

### Option 2: Use Windows Terminal

Windows Terminal provides a better experience for working with WSL:

1. Install Windows Terminal from the Microsoft Store
2. Open Windows Terminal
3. Click the dropdown arrow and select "Ubuntu"

You can also set Ubuntu as the default profile in the Windows Terminal settings.

### Option 3: Use a GUI Client (Advanced)

For a more integrated experience, you can set up QtPass to work with your WSL Ubuntu Pass installation. This is more advanced and requires additional configuration, which we'll cover in a later guide.

## Conclusion

Congratulations! You've successfully:
- Installed Pass on your WSL Ubuntu system
- Initialized it with your imported GPG key
- Connected it to your existing password store
- Set up Git integration for synchronization
- Tested basic password management operations
- Learned how to access Pass from Windows

Your cross-platform password management system is now fully operational, allowing you to securely manage your passwords from both macOS and Windows systems.

In the next guide, we'll cover a detailed synchronization workflow to ensure your passwords stay in sync across your devices.

## Navigation

- [README](README.md) - Wiki Home
- Previous: [Importing GPG Keys to WSL Ubuntu](07_Importing_GPG_Keys_to_WSL_Ubuntu.md)
- Next: [Synchronization Workflow Between Systems](09_Synchronization_Workflow_Between_Systems.md)
- Related:
  - [Git Integration for Pass](04_Git_Integration_for_Pass.md) - For Git setup details
  - [GUI Clients and Browser Integration](11_GUI_Clients_and_Browser_Integration.md) - For GUI options
