# Solving Common GPG Key Transfer Problems

This guide addresses common issues you might encounter when transferring GPG keys between macOS and Windows WSL. We'll provide detailed troubleshooting steps and solutions for each problem.

## Diagnosing SSH Connection Issues

SSH connection problems are common when trying to transfer files using SCP or SFTP.

### Connection Refused Errors

**Problem**: `ssh: connect to host wsl_ip_address port 22: Connection refused`

**Diagnosis**:
```bash
# In WSL, check if SSH server is installed
which sshd

# Check if SSH service is running
service ssh status
```

**Solutions**:
```bash
# Install SSH server if missing
sudo apt update
sudo apt install openssh-server

# Start SSH service
sudo service ssh start

# Make SSH start automatically
echo "sudo service ssh start" >> ~/.bashrc
```

What these commands do:
- Check if the SSH server is installed and running
- Install the server if it's missing
- Start the service if it's not running
- Configure SSH to start automatically when WSL launches

### Host Key Verification Failed

**Problem**: `Host key verification failed` or `Someone could be eavesdropping on you right now`

**Diagnosis**:
```bash
# On macOS, check your known_hosts file
cat ~/.ssh/known_hosts | grep wsl_ip_address
```

**Solutions**:
```bash
# Remove the old key for this IP address
ssh-keygen -R wsl_ip_address

# Try connecting again (and accept the new key)
ssh username@wsl_ip_address
```

What these commands do:
- Remove the old, mismatched host key from your known_hosts file
- Allow you to accept the new, correct host key

### Permission Denied (publickey,password)

**Problem**: `Permission denied (publickey,password)`

**Diagnosis**:
```bash
# In WSL, check SSH configuration
sudo cat /etc/ssh/sshd_config | grep PasswordAuthentication
```

**Solutions**:
```bash
# Edit SSH configuration
sudo nano /etc/ssh/sshd_config

# Make sure this line exists and is not commented out:
# PasswordAuthentication yes

# Restart SSH service
sudo service ssh restart
```

What these commands do:
- Check if password authentication is enabled
- Enable it if it's disabled
- Restart the SSH service to apply changes

## Resolving GPG Version Compatibility Problems

Different GPG versions can sometimes cause compatibility issues.

### Key Import Failures

**Problem**: `gpg: key ABCD1234: no user ID`

**Diagnosis**:
```bash
# Check GPG versions on both systems
# On macOS
gpg --version

# In WSL
gpg --version
```

**Solutions**:
```bash
# If WSL has an older version, update it
sudo apt update
sudo apt install gnupg2

# Try exporting with compatibility options on macOS
gpg --export-options export-minimal --export-secret-keys --armor YOUR_KEY_ID > private_key.asc
```

What these commands do:
- Check the GPG versions on both systems
- Update GPG in WSL if needed
- Use export options that improve compatibility

### Unsupported Algorithm

**Problem**: `gpg: unsupported algorithm: 22`

**Diagnosis**:
```bash
# Check what algorithms your key uses
gpg --list-packets < public_key.asc
```

**Solutions**:
```bash
# Update GPG to the latest version
# On macOS
brew update
brew upgrade gnupg

# In WSL
sudo apt update
sudo apt upgrade gnupg

# If updating isn't possible, create a new key with more compatible algorithms
gpg --full-generate-key
# Choose RSA and RSA
# Choose 2048 or 4096 bits
```

What these commands do:
- Update GPG to support newer algorithms
- If updating isn't possible, create a new key with widely-supported algorithms

### Trust Database Issues

**Problem**: `gpg: trustdb: required modification not possible`

**Diagnosis**:
```bash
# Check trust database permissions
ls -la ~/.gnupg/trustdb.gpg
```

**Solutions**:
```bash
# Fix permissions
chmod 600 ~/.gnupg/trustdb.gpg
chmod 700 ~/.gnupg

# If that doesn't work, recreate the trust database
mv ~/.gnupg/trustdb.gpg ~/.gnupg/trustdb.gpg.bak
gpg --update-trustdb
```

What these commands do:
- Fix permissions on the trust database
- If needed, back up and recreate the trust database

## Handling Permission and Access Errors

Permission problems are common in both GPG and file transfers.

### GPG Home Directory Permissions

**Problem**: `gpg: can't create directory '/home/username/.gnupg': Permission denied`

**Diagnosis**:
```bash
# Check home directory permissions
ls -la ~ | grep "\.gnupg"
```

**Solutions**:
```bash
# Create the directory with correct permissions
mkdir -p ~/.gnupg
chmod 700 ~/.gnupg
```

What these commands do:
- Create the GPG directory if it doesn't exist
- Set permissions so only you can access it

### File Transfer Permission Denied

**Problem**: `scp: /home/username/gpg_transfer/gpg_keys.tar.gz.gpg: Permission denied`

**Diagnosis**:
```bash
# Check destination directory permissions
ls -la ~ | grep gpg_transfer
```

**Solutions**:
```bash
# Create the directory with correct permissions
mkdir -p ~/gpg_transfer
chmod 700 ~/gpg_transfer

# If the directory exists but has wrong permissions
chmod 700 ~/gpg_transfer
```

What these commands do:
- Create the transfer directory if it doesn't exist
- Set permissions so only you can access it

### Cannot Access Private Key

**Problem**: `gpg: signing failed: No secret key` or `gpg: decryption failed: No secret key`

**Diagnosis**:
```bash
# Check if your private key is available
gpg --list-secret-keys
```

**Solutions**:
```bash
# If the key is missing, import it
gpg --import private_key.asc

# If the key exists but isn't accessible, check permissions
chmod 600 ~/.gnupg/private-keys-v1.d/*
chmod 700 ~/.gnupg
```

What these commands do:
- Import the private key if it's missing
- Fix permissions if the key exists but isn't accessible

## Recovering from Failed Transfers

Sometimes transfers fail midway. Here's how to recover:

### Incomplete File Transfers

**Problem**: Transfer was interrupted, leaving an incomplete file

**Diagnosis**:
```bash
# Check file size on both systems
# On macOS
ls -la ~/Desktop/gpg_keys.tar.gz.gpg

# In WSL
ls -la ~/gpg_transfer/gpg_keys.tar.gz.gpg
```

**Solutions**:
```bash
# Using SFTP for resumable transfers
sftp username@wsl_ip_address
# At the sftp> prompt:
put -a ~/Desktop/gpg_keys.tar.gz.gpg
```

What these commands do:
- Check if the file sizes match
- Use SFTP's append mode (`-a`) to resume an interrupted transfer

### Corrupted Files

**Problem**: File transferred but is corrupted

**Diagnosis**:
```bash
# Check file checksums on both systems
# On macOS
shasum -a 256 ~/Desktop/gpg_keys.tar.gz.gpg

# In WSL
sha256sum ~/gpg_transfer/gpg_keys.tar.gz.gpg
```

**Solutions**:
```bash
# If checksums don't match, transfer again with verification
scp -v ~/Desktop/gpg_keys.tar.gz.gpg username@wsl_ip_address:~/gpg_transfer/

# After transfer, verify checksums again
```

What these commands do:
- Verify file integrity using checksums
- Transfer again if the file is corrupted
- Verify the new transfer

### Decryption Failures

**Problem**: `gpg: decryption failed: No secret key` or `gpg: encrypted with 1 passphrase`

**Diagnosis**:
```bash
# Check what key was used for encryption
gpg --list-packets gpg_keys.tar.gz.gpg
```

**Solutions**:
For symmetric encryption (passphrase):
```bash
# Make sure you're using the correct passphrase
gpg --decrypt gpg_keys.tar.gz.gpg > gpg_keys.tar.gz
```

For public key encryption:
```bash
# Make sure the corresponding private key is imported
gpg --import private_key.asc
gpg --decrypt gpg_keys.tar.gz.gpg > gpg_keys.tar.gz
```

What these commands do:
- Determine what type of encryption was used
- Ensure you have the correct decryption key or passphrase

## GPG Error Messages Explained

GPG error messages can be cryptic. Here are explanations for common ones:

### "No such file or directory"

**Problem**: `gpg: can't open 'file.asc': No such file or directory`

**Explanation**: GPG can't find the specified file.

**Solution**:
```bash
# Check if the file exists
ls -la file.asc

# Check if you're in the right directory
pwd

# Use the full path if needed
gpg --import /full/path/to/file.asc
```

### "Inappropriate ioctl for device"

**Problem**: `gpg: Inappropriate ioctl for device`

**Explanation**: GPG can't request a passphrase because it can't access a terminal.

**Solution**:
```bash
# Use loopback pinentry mode
echo "pinentry-mode loopback" >> ~/.gnupg/gpg.conf

# Or specify it on the command line
gpg --pinentry-mode loopback --decrypt file.gpg
```

### "Invalid option --armor"

**Problem**: `gpg: Invalid option "--armor"`

**Explanation**: You're using a short option (`-a`) where a long option (`--armor`) is needed, or vice versa.

**Solution**:
```bash
# Use the correct option format
gpg --armor --export YOUR_KEY_ID > public_key.asc
```

### "Secret key not available"

**Problem**: `gpg: signing failed: secret key not available`

**Explanation**: GPG can't find the private key needed for the operation.

**Solution**:
```bash
# Check available secret keys
gpg --list-secret-keys

# Specify the key explicitly
gpg --default-key YOUR_KEY_ID --sign file.txt
```

## When to Start Over vs. When to Troubleshoot

Sometimes it's better to start fresh rather than troubleshoot:

### Start Over When:

1. **Your key files are corrupted beyond repair**
   ```bash
   # Generate new keys
   gpg --full-generate-key
   ```

2. **You've forgotten your passphrase**
   ```bash
   # If you can't remember your passphrase, you'll need new keys
   gpg --full-generate-key
   ```

3. **You suspect your keys have been compromised**
   ```bash
   # Revoke the old key if possible
   gpg --gen-revoke YOUR_KEY_ID > revocation.asc
   gpg --import revocation.asc
   
   # Generate new keys
   gpg --full-generate-key
   ```

### Continue Troubleshooting When:

1. **Transfer issues are network-related**
   ```bash
   # Try alternative transfer methods
   # See guide: Alternative Transfer Methods
   ```

2. **You're having permission problems**
   ```bash
   # Fix permissions and try again
   chmod 700 ~/.gnupg
   chmod 600 ~/.gnupg/*
   ```

3. **You're encountering version compatibility issues**
   ```bash
   # Try exporting with compatibility options
   gpg --export-options export-minimal --export --armor YOUR_KEY_ID > public_key.asc
   ```

## Community Support Resources

If you're still stuck, these resources can help:

### Online Documentation

- [GnuPG User Guide](https://www.gnupg.org/documentation/manuals/gnupg/)
- [GPG Suite Documentation](https://gpgtools.org/documentation.html)
- [WSL Documentation](https://docs.microsoft.com/en-us/windows/wsl/)

### Community Forums

- [GnuPG Users Forum](https://forum.gnupg.org/)
- [Stack Exchange - Cryptography](https://crypto.stackexchange.com/questions/tagged/gnupg)
- [Stack Exchange - Unix & Linux](https://unix.stackexchange.com/questions/tagged/gnupg)
- [Reddit - r/GnuPG](https://www.reddit.com/r/GnuPG/)

### Getting Help from the Command Line

```bash
# View GPG help
gpg --help

# View help for a specific command
gpg --help command

# View the GPG man page
man gpg

# Get detailed information about your GPG installation
gpg --version
```

## Common Questions and Answers

### "Do I need to transfer my entire .gnupg directory?"

No, you only need to export and transfer your keys and trust database. Transferring the entire directory can cause permission and configuration issues.

### "Can I use the same key on multiple computers?"

Yes, but for better security, consider:
- Using subkeys on secondary computers
- Setting appropriate expiration dates
- Keeping your master key only on your most secure system

### "What if I can't remember my passphrase?"

Unfortunately, if you've forgotten your passphrase, you can't recover your private key. You'll need to:
1. Create a new key pair
2. Revoke the old key if possible
3. Inform your contacts about the new key

### "Is it safe to transfer keys over the internet?"

It's safe if you:
1. Encrypt your keys before transfer
2. Use secure transfer methods (SCP/SFTP)
3. Use strong, unique passphrases
4. Delete the transferred files after successful import

---

**Prerequisites**: 
- Any of the previous guides in the series

**Next Steps**: [Scripting and Automating Secure GPG Key Transfers](11-automating-gpg-key-transfers.md)

**See Also**:
- [Security Best Practices for Key Management](09-security-best-practices.md)
- [Working with Your Transferred GPG Keys in WSL](12-using-gpg-keys-in-wsl.md)
- [Command Reference for GPG Key Management](appendix-command-reference.md)
