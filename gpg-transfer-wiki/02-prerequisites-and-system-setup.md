# Preparing Your Systems for Secure GPG Key Transfer

This guide will help you set up both your macOS and Windows WSL systems for securely transferring GPG keys. We'll walk through each step in detail, explaining what each command does and why it's necessary.

## macOS System Requirements and Preparation

### Installing GPG on macOS

If you don't already have GPG installed on your Mac, you'll need to install it first:

1. **Using Homebrew** (recommended):

   ```bash
   # Install Homebrew if you don't have it
   /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
   
   # Install GPG
   brew install gnupg
   ```

   What this does:
   - The first command downloads and runs the Homebrew installer script
   - The second command uses Homebrew to install GPG

2. **Using GPG Suite**:
   
   Alternatively, you can download and install GPG Suite from the official website:
   https://gpgtools.org/

### Checking Your GPG Installation

Verify that GPG is installed correctly:

```bash
gpg --version
```

This command displays the version of GPG installed on your system. You should see output showing the version number (ideally 2.2.x or newer).

### Checking Existing GPG Configuration

Check if you already have GPG keys:

```bash
gpg --list-secret-keys --keyid-format LONG
```

What this does:
- Lists all private keys in your GPG keyring
- Uses the LONG format to display key IDs
- Shows user IDs, key sizes, and creation dates

If you see any keys listed, you already have GPG keys that you can transfer. If not, you'll need to create keys before transferring them.

### Checking Your GPG Configuration Directory

Your GPG configuration is stored in a directory called `.gnupg` in your home folder:

```bash
ls -la ~/.gnupg
```

What this does:
- Lists all files in your GPG configuration directory
- The `-la` flags show all files (including hidden ones) in a detailed list format

Important files you might see:
- `pubring.kbx` or `pubring.gpg`: Contains your public keys
- `trustdb.gpg`: Contains your trust database
- `private-keys-v1.d/`: Directory containing your private keys
- `gpg.conf`: Your GPG configuration file

## Windows/WSL System Requirements and Preparation

### Installing WSL on Windows

If you don't already have WSL installed:

1. Open PowerShell as Administrator and run:

   ```powershell
   wsl --install
   ```

   What this does:
   - Installs WSL with the default Ubuntu distribution
   - Enables necessary Windows features
   - May require a system restart

2. After installation, open the Ubuntu application from the Start menu to complete setup.

3. When prompted, create a username and password for your WSL environment.

### Installing GPG in WSL

Once WSL is set up, you need to install GPG:

```bash
# Update package lists
sudo apt update

# Install GPG
sudo apt install gnupg
```

What these commands do:
- `sudo apt update`: Refreshes the list of available packages
- `sudo apt install gnupg`: Installs the GPG package
- `sudo` runs the commands with administrator privileges

### Verifying GPG Installation in WSL

Check that GPG is installed correctly:

```bash
gpg --version
```

This should display the version information for GPG in your WSL environment.

### Setting Up SSH Server in WSL

To transfer files directly to WSL, you'll need an SSH server:

```bash
# Install OpenSSH server
sudo apt install openssh-server

# Edit the SSH configuration
sudo nano /etc/ssh/sshd_config
```

In the editor, make sure these lines are set (uncomment if needed):
```
Port 22
PasswordAuthentication yes
```

Save the file by pressing Ctrl+O, then Enter, then Ctrl+X to exit.

```bash
# Start the SSH service
sudo service ssh start

# Make SSH start automatically when WSL launches
echo "sudo service ssh start" >> ~/.bashrc
```

What these commands do:
- Install the OpenSSH server package
- Configure SSH to allow password authentication
- Start the SSH service
- Add a command to your `.bashrc` file so SSH starts automatically when you open WSL

### Finding Your WSL IP Address

You'll need to know your WSL IP address for the transfer:

```bash
ip addr show eth0 | grep -oP '(?<=inet\s)\d+(\.\d+){3}'
```

What this does:
- `ip addr show eth0`: Shows network interface information
- `grep -oP '(?<=inet\s)\d+(\.\d+){3}'`: Extracts just the IP address
- The output will be something like `172.29.123.45`

Write down this IP address as you'll need it for the transfer.

### Configuring Windows Firewall

You may need to allow SSH connections through the Windows firewall:

1. Open Windows Defender Firewall with Advanced Security
2. Select "Inbound Rules" and click "New Rule..."
3. Choose "Port" and click Next
4. Select "TCP" and enter "22" for the port
5. Choose "Allow the connection" and click Next
6. Select all network types and click Next
7. Name the rule "SSH for WSL" and click Finish

## Preparing Directories for Transfer

### Creating a Transfer Directory in WSL

Create a dedicated directory for receiving the GPG keys:

```bash
# Create a directory
mkdir -p ~/gpg_transfer

# Set restrictive permissions
chmod 700 ~/gpg_transfer
```

What these commands do:
- `mkdir -p`: Creates a directory and any parent directories if needed
- `chmod 700`: Sets permissions so only you can read, write, or access the directory

### Checking Available Space

Ensure you have enough disk space for the transfer:

```bash
# On macOS
df -h ~

# In WSL
df -h ~
```

What this does:
- Shows disk space usage in human-readable format
- GPG keys are typically small, but it's good to check

## Tools You'll Need for the Transfer Process

### Command-Line Tools

Make sure these essential tools are available:

1. **On macOS**:
   ```bash
   # Check if SCP is available
   which scp
   
   # Check if SFTP is available
   which sftp
   
   # Check if tar is available
   which tar
   ```

2. **In WSL**:
   ```bash
   # Check if SSH server is running
   service ssh status
   
   # Check if tar is available
   which tar
   ```

What these commands do:
- `which`: Shows the path to the command if it's installed
- `service ssh status`: Shows whether the SSH server is running

### Optional: Installing Secure Deletion Tools

For added security, install tools to securely delete files:

1. **On macOS**:
   ```bash
   # Install srm (secure remove)
   brew install srm
   ```

2. **In WSL**:
   ```bash
   # Install secure-delete package
   sudo apt install secure-delete
   ```

What these do:
- Install tools that overwrite files before deleting them
- This prevents recovery of sensitive data

## Testing the Connection

Before attempting to transfer your keys, test that you can connect from macOS to WSL:

```bash
# From macOS Terminal
ssh username@wsl_ip_address
```

Replace `username` with your WSL username and `wsl_ip_address` with the IP address you found earlier.

If you can successfully connect and log in, your setup is working correctly.

## Troubleshooting Common Setup Issues

### SSH Connection Refused

If you see "Connection refused" when trying to connect:

```bash
# In WSL, check if SSH is running
sudo service ssh status

# If not running, start it
sudo service ssh start

# Check if it's listening on port 22
sudo netstat -tuln | grep 22
```

### GPG Not Found

If GPG commands aren't working:

```bash
# Check if GPG is in your PATH
which gpg

# If not found, try installing again
sudo apt update && sudo apt install gnupg
```

### Permission Issues

If you encounter permission errors:

```bash
# Check ownership of .gnupg directory
ls -la ~/.gnupg

# Fix permissions if needed
chmod 700 ~/.gnupg
chmod 600 ~/.gnupg/*
```

---

**Prerequisites**: [Understanding GPG Keys and Secure Transfer Fundamentals](01-introduction-to-gpg-keys.md)

**Next Steps**: [Safely Exporting Your GPG Keys from macOS](03-exporting-gpg-keys-from-macos.md)

**See Also**:
- [Setting Up SSH Server in WSL](appendix-ssh-configuration-guide.md)
- [Troubleshooting and FAQs](10-troubleshooting-and-faqs.md)
