# Git Integration for Pass

## Introduction

One of the powerful features of Pass is its ability to integrate with Git for version control and synchronization. This integration allows you to:

- Track changes to your password store
- Revert to previous versions if needed
- Synchronize passwords between multiple devices
- Back up your passwords to a remote repository

In this guide, we'll set up Git for your Pass password store on macOS.

## Prerequisites

Before proceeding, make sure you have:
- Completed the previous guides on setting up GPG and Pass
- Git installed on your macOS system (we'll install it if not)

## Installing Git

If you don't already have Git installed, you can install it using Homebrew:

```bash
brew install git
```

This command:
- Downloads the Git software package
- Installs it on your system
- Sets up the necessary configurations

Verify the installation by running:

```bash
git --version
```

You should see output showing the version of Git installed, something like:
```
git version 2.33.0
```

## Initializing Git in Your Password Store

Pass makes it easy to use Git with your password store through built-in commands.

### Step 1: Initialize Git Repository

Run the following command to initialize a Git repository in your password store:

```bash
pass git init
```

This command:
- Creates a new Git repository in your `~/.password-store/` directory
- Sets up the initial Git configuration
- Prepares the repository for tracking changes

You should see output similar to:
```
Initialized empty Git repository in /Users/yourusername/.password-store/.git/
```

### Step 2: Add and Commit Existing Files

If you've already added some passwords to your store, you should add them to Git and create an initial commit:

```bash
pass git add .
pass git commit -m "Initial commit"
```

These commands:
1. `pass git add .`: Stages all existing files in your password store for commit
2. `pass git commit -m "Initial commit"`: Creates a commit with the message "Initial commit"

The output will show which files were added and committed.

## Understanding How Pass Uses Git

When you use Pass with Git integration, every change you make to your password store can be automatically tracked. Pass provides a wrapper around Git commands, allowing you to use Git through Pass itself.

For any Git command, you can simply prefix it with `pass git`:

```bash
pass git [git-command]
```

For example:
- `pass git status`: Shows the current status of your Git repository
- `pass git log`: Shows the commit history
- `pass git diff`: Shows changes between commits

## Setting Up a Remote Repository (Optional but Recommended)

To synchronize your passwords between devices, you'll need to set up a remote repository. You can use services like GitHub, GitLab, or Bitbucket, but make sure to use a **private** repository since your password store contains sensitive information (even though it's encrypted).

### Step 1: Create a Private Repository

Go to your preferred Git hosting service (GitHub, GitLab, etc.) and create a new private repository. Do not initialize it with any files.

### Step 2: Add the Remote Repository to Your Local Git Configuration

Run the following command, replacing the URL with your repository's URL:

```bash
pass git remote add origin git@github.com:yourusername/password-store.git
```

This command:
- Adds a remote repository named "origin"
- Sets the URL to your private repository

### Step 3: Push Your Local Repository to the Remote

```bash
pass git push -u origin master
```

This command:
- Pushes your local commits to the remote repository
- Sets up tracking between your local and remote repositories
- Uses the `-u` flag to remember these settings for future pushes

If you're using SSH authentication for Git (recommended), you might need to set up SSH keys if you haven't already. If you're using HTTPS, you'll be prompted for your username and password.

## Automating Git Operations with Pass

You can configure Pass to automatically commit changes to your password store.

### Step 1: Enable Git Auto-Commit

Edit your `.bashrc` or `.zshrc` file (depending on which shell you use) by running:

```bash
nano ~/.bashrc
# Or if you use Zsh:
# nano ~/.zshrc
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
# Or if you use Zsh:
# source ~/.zshrc
```

Now, whenever you make changes to your password store (add, edit, or remove passwords), Pass will automatically commit those changes to Git.

## Basic Git Workflow for Password Management

Here's a typical workflow for managing your passwords with Git integration:

### Before Making Changes

If you're working across multiple devices, always pull the latest changes before making any modifications:

```bash
pass git pull
```

This command fetches and merges any changes from your remote repository.

### After Making Changes

If you've enabled auto-commit as described above, changes will be committed automatically. Otherwise, manually commit your changes:

```bash
pass git add .
pass git commit -m "Add new bank password"
```

### Pushing Changes to Remote Repository

To make your changes available to other devices:

```bash
pass git push
```

This command uploads your local commits to the remote repository.

## Handling Conflicts

If you make changes to the same password on different devices without synchronizing, you might encounter conflicts when pulling or pushing.

### Resolving a Merge Conflict

If you get a merge conflict, you'll see a message like:
```
CONFLICT (content): Merge conflict in Email/gmail.gpg
Automatic merge failed; fix conflicts and then commit the result.
```

Since the files are encrypted, you can't resolve conflicts in the usual way. Instead:

1. Decide which version you want to keep
2. If you want to keep the local version:
   ```bash
   pass git checkout --ours Email/gmail.gpg
   pass git add Email/gmail.gpg
   pass git commit -m "Resolve conflict, keep local version"
   ```
3. If you want to keep the remote version:
   ```bash
   pass git checkout --theirs Email/gmail.gpg
   pass git add Email/gmail.gpg
   pass git commit -m "Resolve conflict, keep remote version"
   ```

To avoid conflicts, always pull before making changes and push after making changes.

## Best Practices for Git with Pass

1. **Use a Private Repository**: Never store your password repository in a public Git repository, even though the files are encrypted.

2. **Commit Frequently**: Make small, frequent commits with descriptive messages.

3. **Pull Before Making Changes**: Always pull the latest changes before modifying your password store.

4. **Push After Making Changes**: Push your changes after adding or modifying passwords.

5. **Use SSH Authentication**: For remote repositories, use SSH keys instead of HTTPS with username/password.

6. **Backup Your GPG Keys**: Your Git repository is only useful if you have the GPG keys to decrypt it.

## Conclusion

Congratulations! You've successfully:
- Set up Git integration for your Pass password store
- Configured a remote repository for synchronization
- Learned the basic Git workflow for password management
- Understood how to handle conflicts and follow best practices

With Git integration, your password management system is now more robust, with version control and the ability to synchronize between devices. In the next guide, we'll cover how to export your GPG keys so you can set up Pass on additional systems.

Remember that while your passwords are encrypted in the Git repository, the repository itself contains metadata about your password organization. Always use a private repository and follow security best practices.
