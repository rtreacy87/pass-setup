# Using SFTP to Securely Move GPG Keys Between Systems

This guide explains how to transfer your encrypted GPG keys from macOS to Windows WSL using SFTP (Secure File Transfer Protocol). We'll provide detailed explanations of each step and command.

## SFTP vs. SCP: When to Choose SFTP

Both SFTP and SCP provide secure file transfers, but they have different strengths:

**SFTP (Secure File Transfer Protocol)**:
- Interactive file transfer session
- Allows browsing remote directories
- Supports resuming interrupted transfers
- More flexible for exploring the remote system
- Better for transferring multiple files

**SCP (Secure Copy Protocol)**:
- Simpler, single-command file transfers
- Non-interactive (good for scripts)
- Slightly faster for single file transfers
- Less overhead

Choose SFTP when:
- You need to explore the remote file system before transferring
- You want to transfer multiple files
- You prefer an interactive session
- You might need to resume interrupted transfers

## Connecting to WSL via SFTP

To start an SFTP session from your Mac to WSL:

```bash
sftp username@wsl_ip_address
```

Replace:
- `username` with your WSL username
- `wsl_ip_address` with your WSL system's IP address (found using `ip addr show eth0 | grep -oP '(?<=inet\s)\d+(\.\d+){3}'` in WSL)

What this command does:
- Initiates an SFTP connection to your WSL system
- Uses SSH for authentication and encryption
- Prompts for your password (unless you've set up SSH keys)
- Provides an interactive prompt when connected

Example of a successful connection:
```
Connected to wsl_ip_address.
sftp>
```

The `sftp>` prompt indicates you're now in an interactive SFTP session.

## Navigating the File System with SFTP

Once connected, you can navigate both your local and remote file systems:

```
# List files in the current remote directory
sftp> ls

# List files in the current local directory
sftp> lls

# Change to a different remote directory
sftp> cd directory_name

# Change to a different local directory
sftp> lcd directory_name

# Show current remote directory
sftp> pwd

# Show current local directory
sftp> lpwd
```

What these commands do:
- Commands starting with `l` (like `lls`, `lcd`) affect your local system
- Commands without `l` (like `ls`, `cd`) affect the remote system
- This dual-navigation allows you to position yourself in the right directories on both systems

For example, to prepare for the transfer:

```
# Navigate to the directory containing your encrypted key archive on your Mac
sftp> lcd ~/Desktop

# Create and navigate to a directory for the keys on WSL (if it doesn't exist)
sftp> mkdir gpg_transfer
sftp> cd gpg_transfer
```

## Uploading Encrypted Key Archives

Once you're in the right directories, upload your encrypted key archive:

```
sftp> put gpg_keys.tar.gz.gpg
```

What this command does:
- `put` uploads a file from your local system to the remote system
- It takes the file from your current local directory
- It places the file in your current remote directory
- Shows progress during the transfer

Example output:
```
Uploading gpg_keys.tar.gz.gpg to /home/username/gpg_transfer/gpg_keys.tar.gz.gpg
gpg_keys.tar.gz.gpg                  100% 2048     1.0MB/s   00:00
```

For additional options:
```
# Upload with a different filename
sftp> put local_file.name remote_file.name

# Upload with verbose output
sftp> put -v gpg_keys.tar.gz.gpg
```

## Verifying Transfers with SFTP

After uploading, verify that the file transferred correctly:

```
# Check the file exists on the remote system
sftp> ls -l

# Check the file size
sftp> ls -la gpg_keys.tar.gz.gpg
```

What to look for:
- The file should be listed in the directory
- The file size should match the original
- The permissions should be appropriate (typically `-rw-------` or `-rw-r--r--`)

For more thorough verification, you can check file checksums:

```
# Exit SFTP first
sftp> exit

# On macOS
shasum -a 256 ~/Desktop/gpg_keys.tar.gz.gpg

# In WSL terminal
sha256sum ~/gpg_transfer/gpg_keys.tar.gz.gpg
```

The checksums should match exactly, confirming the file wasn't corrupted during transfer.

## Using Graphical SFTP Clients (Alternative)

While command-line SFTP is powerful, some users might prefer graphical clients:

**For macOS**:
- **FileZilla**: Free, open-source SFTP client
- **Cyberduck**: User-friendly file transfer client
- **Transmit**: Premium macOS SFTP client

To use these:
1. Install the client of your choice
2. Create a new connection with:
   - Host: Your WSL IP address
   - Username: Your WSL username
   - Password: Your WSL password
   - Port: 22 (default SSH port)
3. Connect and use drag-and-drop to transfer files

However, command-line SFTP is recommended because:
- It's built into macOS (no additional software needed)
- It's more secure (fewer components that could have vulnerabilities)
- It's easier to verify exactly what's happening
- It works the same way across different systems

## Troubleshooting SFTP Connections

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

### Permission Issues

If you can't upload files due to permission errors:

```
# In WSL, check permissions on the destination directory
ls -la ~/ | grep gpg_transfer

# Fix if needed
chmod 700 ~/gpg_transfer
```

### Command Not Found

If `sftp` command is not found on macOS (very rare):

```bash
# Check if sftp is available
which sftp

# If not found, you may need to install OpenSSH
brew install openssh
```

## Advanced SFTP Features

SFTP has some advanced features that can be useful in certain situations:

```
# Resume a failed transfer
sftp> reput gpg_keys.tar.gz.gpg

# Set file permissions during upload
sftp> put -P gpg_keys.tar.gz.gpg

# Transfer multiple files at once
sftp> mput *.gpg

# Create a remote directory and upload to it in one step
sftp> mkdir new_directory
sftp> cd new_directory
sftp> put gpg_keys.tar.gz.gpg
```

## Securing Your SFTP Session

To enhance security during SFTP transfers:

```bash
# Use SSH keys instead of passwords
# On macOS, generate an SSH key if you don't have one
ssh-keygen -t ed25519 -C "your_email@example.com"

# Copy your public key to WSL
ssh-copy-id username@wsl_ip_address

# Connect with SFTP (no password needed)
sftp username@wsl_ip_address
```

After your transfer is complete:

```
# Exit SFTP
sftp> exit

# Clear your terminal history
history -c  # On macOS
```

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
- [Step-by-Step Guide to Transferring GPG Keys with SCP](05-transferring-keys-using-scp.md)
- [Troubleshooting and FAQs](10-troubleshooting-and-faqs.md)
- [Setting Up SSH Server in WSL](appendix-ssh-configuration-guide.md)
