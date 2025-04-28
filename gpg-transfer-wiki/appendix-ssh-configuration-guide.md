# Setting Up SSH Server in WSL

This appendix provides detailed instructions for setting up and configuring an SSH server in Windows Subsystem for Linux (WSL). A properly configured SSH server is essential for securely transferring GPG keys using SCP or SFTP.

## Installing the SSH Server

First, you need to install the OpenSSH server package:

```bash
# Update package lists
sudo apt update

# Install OpenSSH server
sudo apt install openssh-server
```

What these commands do:
- `sudo apt update` refreshes the list of available packages
- `sudo apt install openssh-server` installs the SSH server and its dependencies

## Basic SSH Server Configuration

After installation, you need to configure the SSH server:

```bash
# Create a backup of the original configuration
sudo cp /etc/ssh/sshd_config /etc/ssh/sshd_config.bak

# Edit the SSH configuration file
sudo nano /etc/ssh/sshd_config
```

Make the following changes to the configuration file:

1. Ensure the SSH port is set (uncomment if needed):
   ```
   Port 22
   ```

2. Enable password authentication (needed for SCP/SFTP):
   ```
   PasswordAuthentication yes
   ```

3. Allow only specific authentication methods:
   ```
   # Allow password and public key authentication
   AuthenticationMethods publickey,password publickey password
   ```

4. Restrict SSH access to your user (optional but recommended):
   ```
   AllowUsers yourusername
   ```

5. Disable root login:
   ```
   PermitRootLogin no
   ```

Save the file by pressing Ctrl+O, then Enter, then Ctrl+X to exit.

## Starting and Managing the SSH Service

Now you can start the SSH service:

```bash
# Start the SSH service
sudo service ssh start

# Check if the service is running
sudo service ssh status
```

What these commands do:
- `sudo service ssh start` starts the SSH server
- `sudo service ssh status` shows whether the server is running correctly

You should see output indicating that the service is active (running).

## Making SSH Start Automatically

By default, the SSH service doesn't start automatically when you launch WSL. To make it start automatically:

```bash
# Add SSH startup to your .bashrc file
echo "sudo service ssh start > /dev/null 2>&1" >> ~/.bashrc
```

What this does:
- Adds a command to your `.bashrc` file that starts SSH when you open a terminal
- `> /dev/null 2>&1` suppresses output to keep your terminal clean

## Finding Your WSL IP Address

To connect to your SSH server, you need to know your WSL IP address:

```bash
# Display your IP address
ip addr show eth0 | grep -oP '(?<=inet\s)\d+(\.\d+){3}'
```

What this does:
- Shows the IP address of your WSL instance
- The output will be something like `172.29.123.45`

Note this IP address as you'll need it to connect via SCP or SFTP.

## Testing Your SSH Server

Test that your SSH server is working correctly:

```bash
# From another terminal or computer
ssh yourusername@your_wsl_ip
```

If you can connect successfully, your SSH server is working properly.

## Configuring Windows Firewall

Windows Firewall might block SSH connections to your WSL instance. To allow these connections:

### Using PowerShell (Run as Administrator)

```powershell
# Allow SSH traffic through Windows Firewall
New-NetFirewallRule -DisplayName "WSL SSH" -Direction Inbound -Protocol TCP -LocalPort 22 -Action Allow
```

### Using Windows Defender Firewall with Advanced Security (GUI)

1. Open "Windows Defender Firewall with Advanced Security"
2. Select "Inbound Rules" and click "New Rule..."
3. Choose "Port" and click Next
4. Select "TCP" and enter "22" for the port
5. Choose "Allow the connection" and click Next
6. Select all network types and click Next
7. Name the rule "WSL SSH" and click Finish

## Security Hardening for SSH

Securing your SSH server is critical, especially when it's used for transferring sensitive data like GPG keys. This section covers security hardening for both WSL and Windows environments.

### SSH Security Hardening in WSL

To enhance the security of your SSH server in WSL:

```bash
# Edit the SSH configuration
sudo nano /etc/ssh/sshd_config
```

Add or modify these settings:

```
# Use only strong encryption algorithms
Ciphers chacha20-poly1305@openssh.com,aes256-gcm@openssh.com,aes128-gcm@openssh.com,aes256-ctr,aes192-ctr,aes128-ctr

# Use only strong MAC algorithms
MACs hmac-sha2-512-etm@openssh.com,hmac-sha2-256-etm@openssh.com,hmac-sha2-512,hmac-sha2-256

# Use only strong key exchange algorithms
KexAlgorithms curve25519-sha256@libssh.org,diffie-hellman-group-exchange-sha256

# Disable weak authentication methods
HostbasedAuthentication no
IgnoreRhosts yes

# Limit login attempts
MaxAuthTries 3
MaxSessions 2

# Set login grace time
LoginGraceTime 30s

# Disable empty passwords
PermitEmptyPasswords no

# Disable X11 forwarding (not needed for key transfers)
X11Forwarding no

# Set strict mode
StrictModes yes

# Enable privilege separation
UsePrivilegeSeparation sandbox

# Set logging level
LogLevel VERBOSE

# Disable TCP forwarding (not needed for key transfers)
AllowTcpForwarding no
GatewayPorts no
PermitTunnel no
```

After making these changes, restart the SSH service:

```bash
sudo service ssh restart
```

### Security Hardening for Windows SSH Access

When accessing SSH from Windows or allowing SSH connections to your Windows machine, follow these additional security measures:

#### 1. Install Windows OpenSSH Client and Server (if needed)

```powershell
# Run in PowerShell as Administrator
Add-WindowsCapability -Online -Name OpenSSH.Client~~~~0.0.1.0
Add-WindowsCapability -Online -Name OpenSSH.Server~~~~0.0.1.0
```

#### 2. Configure Windows OpenSSH Server (if used)

```powershell
# Start and configure the service
Start-Service sshd
Set-Service -Name sshd -StartupType 'Automatic'

# Confirm the firewall rule is configured
Get-NetFirewallRule -Name *ssh*
```

#### 3. Create a Secure Windows SSH Configuration

Create or edit the file at `C:\ProgramData\ssh\sshd_config`:

```
# Use only strong encryption
Ciphers chacha20-poly1305@openssh.com,aes256-gcm@openssh.com,aes128-gcm@openssh.com,aes256-ctr,aes192-ctr,aes128-ctr

# Use only strong MACs
MACs hmac-sha2-512-etm@openssh.com,hmac-sha2-256-etm@openssh.com,hmac-sha2-512,hmac-sha2-256

# Disable password authentication (use keys only)
PasswordAuthentication no
PermitEmptyPasswords no

# Disable root login
PermitRootLogin no

# Limit login attempts
MaxAuthTries 3

# Set login grace time
LoginGraceTime 30s

# Disable X11 forwarding
X11Forwarding no

# Set strict mode
StrictModes yes

# Set logging level
LogLevel VERBOSE

# Restrict to specific users (replace with your Windows username)
AllowUsers YourWindowsUsername
```

After editing, restart the SSH service:

```powershell
Restart-Service sshd
```

#### 4. Use Windows Defender Firewall to Restrict SSH Access

```powershell
# Create a more restrictive firewall rule (example for allowing only specific IP)
New-NetFirewallRule -DisplayName "SSH Restricted" -Direction Inbound -Protocol TCP -LocalPort 22 -RemoteAddress "192.168.1.0/24" -Action Allow

# Remove the more permissive rule if it exists
Remove-NetFirewallRule -DisplayName "SSH" -ErrorAction SilentlyContinue
```

#### 5. Enable Windows Event Logging for SSH

```powershell
# Configure enhanced logging
wevtutil sl Microsoft-Windows-SSHd/Admin /e:true
wevtutil sl Microsoft-Windows-SSHd/Operational /e:true
```

#### 6. Use Windows Defender Advanced Threat Protection (if available)

If you have Microsoft Defender for Endpoint:

1. Go to Security Center → Settings → Advanced features
2. Enable "Custom network indicators" 
3. Add monitoring for suspicious SSH traffic patterns

#### 7. Implement Account Lockout Policies

```powershell
# Set account lockout policy
net accounts /lockoutthreshold:5 /lockoutduration:30 /lockoutwindow:30
```

#### 8. Use Windows Hello for SSH Authentication (Windows 10/11)

For enhanced security, you can configure Windows SSH to use Windows Hello:

1. Install the Windows Hello OpenSSH component
2. Configure SSH to use the Windows security key provider

```powershell
# Add this to your SSH config
Host *
    SecurityKeyProvider winhello.dll
```

### Best Practices for SSH Key Management in Windows

1. **Store SSH keys securely**:
   - Use Windows Credential Manager
   - Consider hardware security keys (YubiKey, etc.)

2. **Use separate keys for different purposes**:
   - Don't use the same key for all systems
   - Create dedicated keys for sensitive operations

3. **Set appropriate permissions**:
   ```powershell
   # Restrict access to SSH key files
   icacls C:\Users\YourUsername\.ssh\id_rsa /inheritance:r
   icacls C:\Users\YourUsername\.ssh\id_rsa /grant:r "YourUsername:(R,W)"
   ```

4. **Regularly rotate SSH keys**:
   - Create new keys periodically
   - Remove old authorized keys

5. **Use SSH config file for security settings**:
   Create or edit `C:\Users\YourUsername\.ssh\config`:
   ```
   Host *
       HashKnownHosts yes
       StrictHostKeyChecking ask
       VerifyHostKeyDNS yes
       ForwardAgent no
       ForwardX11 no
       ControlMaster no
   ```

## Advanced SSH Security Configuration

### Using SSH Keys Instead of Passwords

```bash
# On your client machine (e.g., macOS), generate an SSH key
ssh-keygen -t ed25519 -C "your_email@example.com"

# Copy the key to your WSL instance
ssh-copy-id yourusername@your_wsl_ip

# After confirming key-based login works, disable password authentication
sudo nano /etc/ssh/sshd_config
```

Change this line:
```
PasswordAuthentication yes
```

To:
```
PasswordAuthentication no
```

Then restart the SSH service:
```bash
sudo service ssh restart
```

### Changing the Default SSH Port

Using a non-standard port can reduce automated attacks:

```bash
# Edit the SSH configuration
sudo nano /etc/ssh/sshd_config
```

Change this line:
```
Port 22
```

To a different port (e.g., between 1024 and 65535):
```
Port 2222
```

Then restart the SSH service:
```bash
sudo service ssh restart
```

Remember to update your firewall rule for the new port and specify the port when connecting:
```bash
ssh -p 2222 yourusername@your_wsl_ip
```

### Limiting SSH Access Attempts

To prevent brute force attacks:

```bash
# Edit the SSH configuration
sudo nano /etc/ssh/sshd_config
```

Add these lines:
```
# Limit login attempts
MaxAuthTries 3
```

Then restart the SSH service:
```bash
sudo service ssh restart
```

## Troubleshooting SSH Server Issues

### Connection Refused

If you see "Connection refused" when trying to connect:

```bash
# Check if SSH is running
sudo service ssh status

# If not running, start it
sudo service ssh start

# Check if it's listening on the correct port
sudo netstat -tuln | grep 22
```

### Permission Denied

If you see "Permission denied" when trying to connect:

```bash
# Check SSH server logs
sudo cat /var/log/auth.log | grep sshd

# Verify your username and password
whoami

# Check if your user is allowed in sshd_config
sudo grep AllowUsers /etc/ssh/sshd_config
```

### SSH Process Crashes

If the SSH service keeps crashing:

```bash
# Check for errors in the system log
sudo journalctl -u ssh

# Verify SSH configuration syntax
sudo sshd -t

# Restore from backup if configuration is corrupted
sudo cp /etc/ssh/sshd_config.bak /etc/ssh/sshd_config
sudo service ssh restart
```

## SSH Configuration for File Transfers

For optimal file transfer performance:

```bash
# Edit the SSH configuration
sudo nano /etc/ssh/sshd_config
```

Add these lines:
```
# Optimize for file transfers
Compression yes
TCPKeepAlive yes
ClientAliveInterval 60
ClientAliveCountMax 10
```

What these settings do:
- `Compression yes` enables data compression for faster transfers
- `TCPKeepAlive yes` keeps connections alive during transfers
- `ClientAliveInterval 60` sends a keep-alive message every 60 seconds
- `ClientAliveCountMax 10` allows 10 missed keep-alive messages before disconnecting

Then restart the SSH service:
```bash
sudo service ssh restart
```

## Using SFTP with Your SSH Server

The SSH server you've configured also provides SFTP functionality:

```bash
# Connect using SFTP
sftp yourusername@your_wsl_ip

# At the sftp> prompt:
# List files
ls

# Navigate directories
cd directory_name

# Upload files
put local_file.txt

# Download files
get remote_file.txt

# Exit
exit
```

No additional configuration is needed for SFTP as it's included with the SSH server.

## Security Best Practices for SSH

Follow these best practices to keep your SSH server secure:

1. **Keep software updated**:
   ```bash
   sudo apt update && sudo apt upgrade
   ```

2. **Use strong passwords**:
   ```bash
   passwd
   ```

3. **Use key-based authentication** instead of passwords when possible

4. **Limit user access** to only those who need it

5. **Monitor login attempts**:
   ```bash
   sudo cat /var/log/auth.log | grep sshd
   ```

6. **Consider using fail2ban** to block repeated login attempts:
   ```bash
   sudo apt install fail2ban
   ```

7. **Regularly check for unauthorized access**:
   ```bash
   last
   ```

## Next Steps

After setting up your SSH server, you're ready to transfer your GPG keys using SCP or SFTP. Refer to the following guides:

- [Step-by-Step Guide to Transferring GPG Keys with SCP](05-transferring-keys-using-scp.md)
- [Using SFTP to Securely Move GPG Keys Between Systems](06-transferring-keys-using-sftp.md)

---

**See Also**:
- [Preparing Your Systems for Secure GPG Key Transfer](02-prerequisites-and-system-setup.md)
- [Troubleshooting and FAQs](10-troubleshooting-and-faqs.md)
