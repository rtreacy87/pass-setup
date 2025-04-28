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

After running this command, restart your computer.

### Step 3: Install the Linux Kernel Update Package

Download and install the WSL2 Linux kernel update package from Microsoft:
[WSL2 Linux Kernel Update Package](https://wslstorestorage.blob.core.windows.net/wslblob/wsl_update_x64.msi)

Run the downloaded MSI file and follow the installation prompts.

### Step 4: Set WSL 2 as Default

Open PowerShell as Administrator and run:

```powershell
wsl --set-default-version 2
```

This sets WSL 2 as the default version for new Linux distributions.

### Step 5: Install Ubuntu

1. Open the Microsoft Store
2. Search for "Ubuntu"
3. Select "Ubuntu" (or a specific version like "Ubuntu 20.04 LTS")
4. Click "Get" or "Install"
5. Once installed, click "Launch"

When Ubuntu launches for the first time, you'll need to:
1. Wait for the installation to complete
2. Create a username and password for your Ubuntu user account

## Installing Required Packages

Now that Ubuntu is installed, we need to install the necessary packages for Pass:

```bash
# Update package lists
sudo apt update

# Upgrade existing packages
sudo apt upgrade -y

# Install required packages
sudo apt install -y git gnupg pass
```

These commands:
1. Update the package repository information
2. Upgrade existing packages to their latest versions
3. Install Git (for version control), GnuPG (for encryption), and Pass (the password manager)

## Setting Up SSH for Secure File Transfers

If you plan to transfer files between your Windows host and WSL Ubuntu, or between your macOS machine and WSL Ubuntu, setting up SSH is recommended:

```bash
# Install OpenSSH server
sudo apt install -y openssh-server

# Edit SSH configuration
sudo nano /etc/ssh/sshd_config
```

In the configuration file, make sure these lines are uncommented or added:

```
Port 22
PasswordAuthentication yes
```

Save the file (Ctrl+O, then Enter) and exit (Ctrl+X).

Start the SSH service:

```bash
sudo service ssh start
```

Find your WSL IP address:

```bash
ip addr show eth0 | grep -oP '(?<=inet\s)\d+(\.\d+){3}'
```

Note this IP address as you'll need it to connect via SSH from your Windows host or other machines.

## Security Hardening for Windows SSH Access

When accessing SSH from Windows or allowing SSH connections to your Windows machine, follow these additional security measures:

### 1. Install Windows OpenSSH Client and Server (if needed)

```powershell
# Run in PowerShell as Administrator
Add-WindowsCapability -Online -Name OpenSSH.Client~~~~0.0.1.0
Add-WindowsCapability -Online -Name OpenSSH.Server~~~~0.0.1.0
```

### 2. Configure Windows OpenSSH Server (if used)

```powershell
# Start and configure the service
Start-Service sshd
Set-Service -Name sshd -StartupType 'Automatic'

# Confirm the firewall rule is configured
Get-NetFirewallRule -Name *ssh*
```

### 3. Create a Secure Windows SSH Configuration

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

### 4. Use Windows Defender Firewall to Restrict SSH Access

```powershell
# Create a more restrictive firewall rule (example for allowing only specific IP)
New-NetFirewallRule -DisplayName "SSH Restricted" -Direction Inbound -Protocol TCP -LocalPort 22 -RemoteAddress "192.168.1.0/24" -Action Allow

# Remove the more permissive rule if it exists
Remove-NetFirewallRule -DisplayName "SSH" -ErrorAction SilentlyContinue
```

### 5. Enable Windows Event Logging for SSH

```powershell
# Configure enhanced logging
wevtutil sl Microsoft-Windows-SSHd/Admin /e:true
wevtutil sl Microsoft-Windows-SSHd/Operational /e:true
```

### 6. Use Windows Defender Advanced Threat Protection (if available)

If you have Microsoft Defender for Endpoint:

1. Go to Security Center → Settings → Advanced features
2. Enable "Custom network indicators" 
3. Add monitoring for suspicious SSH traffic patterns

### 7. Implement Account Lockout Policies

```powershell
# Set account lockout policy
net accounts /lockoutthreshold:5 /lockoutduration:30 /lockoutwindow:30
```

### 8. Use Windows Hello for SSH Authentication (Windows 10/11)

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

## Verifying the Installation

To verify that everything is installed correctly:

```bash
# Check Git version
git --version

# Check GnuPG version
gpg --version

# Check Pass version
pass --version
```

Each command should display the version information for the respective tool.

## Next Steps

Now that you have WSL Ubuntu set up with the necessary packages, you're ready to:

1. Import your GPG keys from macOS (covered in the next guide)
2. Initialize Pass with your imported GPG key
3. Set up synchronization between your macOS and WSL Ubuntu environments

Continue to the next guide: [Importing GPG Keys to WSL Ubuntu](07_Importing_GPG_Keys_to_WSL_Ubuntu.md)

---

**Previous**: [Exporting GPG Keys for Cross-Platform Use](05_Exporting_GPG_Keys_for_Cross_Platform_Use.md)  
**Next**: [Importing GPG Keys to WSL Ubuntu](07_Importing_GPG_Keys_to_WSL_Ubuntu.md)
