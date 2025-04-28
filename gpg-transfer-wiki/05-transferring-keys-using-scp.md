# Step-by-Step Guide to Transferring GPG Keys with SCP

This guide will walk you through the process of transferring your encrypted GPG keys from macOS to Windows WSL using SCP (Secure Copy Protocol). We'll explain each step in detail so you can understand exactly what's happening.

## What is SCP and How It Ensures Security

SCP (Secure Copy Protocol) is a command-line tool for securely transferring files between computers. Here's why it's a good choice for transferring sensitive files like GPG keys:

- **Encryption**: SCP encrypts both the file data and authentication information
- **Authentication**: Uses SSH keys or passwords to verify identity
- **Integrity**: Verifies that files aren't modified during transfer
- **Simplicity**: Simple command-line syntax for direct transfers

Think of SCP as a secure pipeline between two computers - anything that goes through this pipeline is protected from eavesdropping or tampering.

## Finding Your WSL IP Address

Before you can transfer files to WSL, you need to know its IP address:

```bash
# In your WSL terminal
ip addr show eth0 | grep -oP '(?<=inet\s)\d+(\.\d+){3}'
```

What this command does:
- `ip addr show eth0` displays information about your network interface
- `grep -oP '(?<=inet\s)\d+(\.\d+){3}'` extracts just the IP address
- The output will be something like `172.29.123.45`

Write down this IP address - you'll need it for the SCP command.

## Understanding SCP Command Syntax and Options

The basic syntax for SCP is:

```
scp [options] source_file username@destination_host:destination_path
```

Common options include:
- `-P port` - Specify a custom SSH port (capital P, unlike ssh which uses lowercase -p)
- `-r` - Recursively copy entire directories
- `-v` - Verbose mode (shows detailed progress)
- `-C` - Compress data during transfer (useful for slow connections)

For our purpose, we'll use a simple command with the `-v` option for better visibility of what's happening.

## Preparing for the Transfer

Before running the SCP command, make sure:

1. Your encrypted GPG key archive is ready (from the previous guide)
2. The SSH server is running in WSL
3. You know your WSL username and IP address
4. You have a directory in WSL to receive the files

```bash
# In WSL, create a directory to receive the files if you haven't already
mkdir -p ~/gpg_transfer
chmod 700 ~/gpg_transfer
```

What these commands do:
- `mkdir -p` creates a directory (and parent directories if needed)
- `chmod 700` sets permissions so only you can access the directory

## Executing the Transfer from macOS to WSL

Now, on your Mac, open Terminal and run the SCP command:

```bash
scp -v ~/Desktop/gpg_keys.tar.gz.gpg username@wsl_ip_address:~/gpg_transfer/
```

Replace:
- `username` with your actual WSL username
- `wsl_ip_address` with the IP address you found earlier

What this command does:
- `-v` enables verbose output so you can see what's happening
- `~/Desktop/gpg_keys.tar.gz.gpg` is the source file (your encrypted key archive)
- `username@wsl_ip_address:~/gpg_transfer/` is the destination

When you run this command:
1. You'll be prompted for your WSL user's password
2. SCP will establish a secure connection
3. It will copy the file to the specified directory in WSL
4. You'll see progress information because of the `-v` flag

Example output:
```
Executing: program /usr/bin/ssh host 172.29.123.45, user username, command scp -v -t ~/gpg_transfer/
OpenSSH_8.6p1, LibreSSL 3.3.5
debug1: Reading configuration data /Users/yourmacuser/.ssh/config
debug1: Connecting to 172.29.123.45 port 22.
debug1: Connection established.
debug1: Authenticating to 172.29.123.45:22 as 'username'
debug1: Authentication succeeded (password).
debug1: channel 0: new [client-session]
debug1: Entering interactive session.
debug1: Sending command: scp -v -t ~/gpg_transfer/
Sending file modes: C0644 2048 gpg_keys.tar.gz.gpg
gpg_keys.tar.gz.gpg                  100% 2048     1.0MB/s   00:00    
debug1: Exit status 0
```

## Verifying Successful File Transfer

After the transfer completes, verify that the file arrived correctly:

```bash
# In your WSL terminal
ls -la ~/gpg_transfer/
```

You should see your encrypted key archive listed with the correct file size.

For additional verification, you can check the file's checksum on both systems:

```bash
# On macOS
shasum -a 256 ~/Desktop/gpg_keys.tar.gz.gpg

# In WSL
sha256sum ~/gpg_transfer/gpg_keys.tar.gz.gpg
```

What these commands do:
- Calculate a unique "fingerprint" of the file based on its contents
- If the checksums match on both systems, the file transferred correctly
- The `-a 256` and `sha256sum` specify the SHA-256 algorithm

The output should be identical on both systems - a long string of letters and numbers followed by the filename.

## Troubleshooting Connection Issues

### Connection Refused

If you see "Connection refused":

```bash
# In WSL, check if SSH is running
sudo service ssh status

# If not running, start it
sudo service ssh start
```

### Authentication Failures

If your password isn't being accepted:

```bash
# Check that you're using the correct username
whoami  # Run this in WSL to confirm your username

# Make sure password authentication is enabled in WSL
sudo nano /etc/ssh/sshd_config
# Look for "PasswordAuthentication" and make sure it's set to "yes"
# If you changed it, restart SSH:
sudo service ssh restart
```

### Permission Denied for Files

If SCP fails with "Permission denied":

```bash
# Check permissions on the destination directory
ls -la ~/ | grep gpg_transfer

# Fix if needed
chmod 700 ~/gpg_transfer
```

### Network Connectivity Issues

If you can't connect at all:

```bash
# On macOS, test basic connectivity
ping wsl_ip_address

# Check if the port is reachable
nc -zv wsl_ip_address 22
```

## Security Considerations Specific to SCP

While SCP is generally secure, keep these points in mind:

1. **Password transmission**: When you type your password, it's encrypted but still transmitted each time
2. **Terminal history**: Your SCP command (including the hostname) is saved in your terminal history
3. **File permissions**: Transferred files inherit default permissions unless specified

To address these:

```bash
# Clear your terminal history after transfer
history -c  # On macOS

# Verify and fix permissions on the transferred file
chmod 600 ~/gpg_transfer/gpg_keys.tar.gz.gpg  # In WSL
```

## Using SSH Keys for More Secure Transfers (Optional)

For even more security, you can use SSH keys instead of passwords:

```bash
# On macOS, generate an SSH key if you don't have one
ssh-keygen -t ed25519 -C "your_email@example.com"

# Copy your public key to WSL
ssh-copy-id username@wsl_ip_address

# Now you can SCP without entering a password
scp ~/Desktop/gpg_keys.tar.gz.gpg username@wsl_ip_address:~/gpg_transfer/
```

What these commands do:
- `ssh-keygen` creates a new SSH key pair
- `ssh-copy-id` securely copies your public key to the WSL authorized_keys file
- This allows password-less authentication for future connections

## Next Steps

After successfully transferring your encrypted GPG key archive to WSL, you'll need to:

1. Decrypt the archive in WSL
2. Import the keys into your WSL GPG keyring

These steps are covered in detail in the guide [Successfully Importing GPG Keys into Windows WSL](07-importing-gpg-keys-in-wsl.md).

---

**Prerequisites**: 
- [Understanding GPG Keys and Secure Transfer Fundamentals](01-introduction-to-gpg-keys.md)
- [Preparing Your Systems for Secure GPG Key Transfer](02-prerequisites-and-system-setup.md)
- [Protecting Your GPG Keys During Transfer](04-securing-exported-keys.md)

**Next Steps**: [Successfully Importing GPG Keys into Windows WSL](07-importing-gpg-keys-in-wsl.md)

**See Also**:
- [Using SFTP to Securely Move GPG Keys Between Systems](06-transferring-keys-using-sftp.md)
- [Troubleshooting and FAQs](10-troubleshooting-and-faqs.md)
- [Setting Up SSH Server in WSL](appendix-ssh-configuration-guide.md)
