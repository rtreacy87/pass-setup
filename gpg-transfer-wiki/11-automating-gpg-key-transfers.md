# Scripting and Automating Secure GPG Key Transfers

This guide covers how to automate the process of transferring GPG keys between systems. We'll explain when automation is appropriate, provide example scripts, and discuss security considerations.

## When Automation is Appropriate (and When It Isn't)

Automation can save time and reduce errors, but it's not always the right choice for sensitive operations like key transfers.

### Appropriate for Automation:

- Regular transfers of subkeys to multiple systems
- Scheduled key rotations in a secure environment
- Deployments in controlled enterprise environments
- Backup procedures for keys

### Not Appropriate for Automation:

- One-time transfers of master keys
- Transfers involving high-security keys
- Situations where passphrases would need to be stored in scripts
- When you don't fully understand the security implications

## Creating Secure Shell Scripts for Key Export

Here's a script to safely export your GPG keys from macOS:

```bash
#!/bin/bash
# gpg_export.sh - Safely export GPG keys

# Exit on any error
set -e

# Configuration (edit these variables)
KEY_ID="YOUR_KEY_ID"
EXPORT_DIR="$HOME/gpg_export_$(date +%Y%m%d)"
ENCRYPT_PASSWORD=""  # Will prompt if left empty

# Create export directory with secure permissions
mkdir -p "$EXPORT_DIR"
chmod 700 "$EXPORT_DIR"

echo "Exporting GPG keys to $EXPORT_DIR..."

# Export public key
gpg --export --armor "$KEY_ID" > "$EXPORT_DIR/public_key.asc"
echo "Public key exported."

# Export private key
gpg --export-secret-keys --armor "$KEY_ID" > "$EXPORT_DIR/private_key.asc"
echo "Private key exported."

# Export trust database
gpg --export-ownertrust > "$EXPORT_DIR/trust.txt"
echo "Trust database exported."

# Create archive
tar -czf "$EXPORT_DIR/gpg_keys.tar.gz" -C "$EXPORT_DIR" public_key.asc private_key.asc trust.txt
echo "Archive created."

# Encrypt the archive
if [ -z "$ENCRYPT_PASSWORD" ]; then
    # Interactive mode - will prompt for password
    gpg --symmetric --cipher-algo AES256 "$EXPORT_DIR/gpg_keys.tar.gz"
else
    # Non-interactive mode with provided password (less secure)
    echo "$ENCRYPT_PASSWORD" | gpg --batch --yes --passphrase-fd 0 --symmetric --cipher-algo AES256 "$EXPORT_DIR/gpg_keys.tar.gz"
fi
echo "Archive encrypted."

# Securely delete unencrypted files
if command -v srm &> /dev/null; then
    srm -v "$EXPORT_DIR/public_key.asc" "$EXPORT_DIR/private_key.asc" "$EXPORT_DIR/trust.txt" "$EXPORT_DIR/gpg_keys.tar.gz"
else
    rm -P "$EXPORT_DIR/public_key.asc" "$EXPORT_DIR/private_key.asc" "$EXPORT_DIR/trust.txt" "$EXPORT_DIR/gpg_keys.tar.gz"
fi
echo "Temporary files securely deleted."

echo "Export complete. Encrypted archive is at: $EXPORT_DIR/gpg_keys.tar.gz.gpg"
```

What this script does:
- Creates a secure directory for exports
- Exports your public key, private key, and trust database
- Creates and encrypts an archive containing these files
- Securely deletes the unencrypted files
- Provides status updates throughout the process

To use this script:
```bash
# Make it executable
chmod +x gpg_export.sh

# Edit the script to set your KEY_ID
nano gpg_export.sh

# Run the script
./gpg_export.sh
```

## Automating the Encryption Process

For more advanced encryption automation, you can use a script that handles different encryption methods:

```bash
#!/bin/bash
# gpg_encrypt.sh - Encrypt files for secure transfer

# Exit on any error
set -e

# Configuration
INPUT_FILE="$1"
OUTPUT_DIR="${2:-$HOME/encrypted_files}"
ENCRYPTION_TYPE="${3:-symmetric}"  # symmetric or recipient
RECIPIENT_KEY="${4:-}"  # Used if ENCRYPTION_TYPE is recipient

# Validate input
if [ -z "$INPUT_FILE" ]; then
    echo "Error: No input file specified."
    echo "Usage: $0 input_file [output_dir] [encryption_type] [recipient_key]"
    exit 1
fi

if [ ! -f "$INPUT_FILE" ]; then
    echo "Error: Input file '$INPUT_FILE' not found."
    exit 1
fi

# Create output directory
mkdir -p "$OUTPUT_DIR"
chmod 700 "$OUTPUT_DIR"

# Get filename without path
FILENAME=$(basename "$INPUT_FILE")
OUTPUT_FILE="$OUTPUT_DIR/$FILENAME.gpg"

echo "Encrypting $INPUT_FILE to $OUTPUT_FILE..."

# Perform encryption based on type
if [ "$ENCRYPTION_TYPE" = "symmetric" ]; then
    # Symmetric encryption (password-based)
    gpg --symmetric --cipher-algo AES256 --output "$OUTPUT_FILE" "$INPUT_FILE"
    echo "File encrypted with symmetric encryption."
elif [ "$ENCRYPTION_TYPE" = "recipient" ]; then
    # Public key encryption
    if [ -z "$RECIPIENT_KEY" ]; then
        echo "Error: Recipient key required for public key encryption."
        exit 1
    fi
    gpg --encrypt --recipient "$RECIPIENT_KEY" --output "$OUTPUT_FILE" "$INPUT_FILE"
    echo "File encrypted for recipient $RECIPIENT_KEY."
else
    echo "Error: Unknown encryption type '$ENCRYPTION_TYPE'."
    echo "Use 'symmetric' or 'recipient'."
    exit 1
fi

echo "Encryption complete. Encrypted file is at: $OUTPUT_FILE"
```

What this script does:
- Takes an input file and encrypts it
- Supports both symmetric (password) and public key encryption
- Creates a secure output directory
- Provides clear error messages and validation

To use this script:
```bash
# Make it executable
chmod +x gpg_encrypt.sh

# For symmetric encryption (will prompt for password)
./gpg_encrypt.sh gpg_keys.tar.gz

# For public key encryption
./gpg_encrypt.sh gpg_keys.tar.gz ~/encrypted_output recipient recipient@email.com
```

## Secure Transfer Scripts

Here's a script to securely transfer files using SCP:

```bash
#!/bin/bash
# secure_transfer.sh - Securely transfer files using SCP

# Exit on any error
set -e

# Configuration
SOURCE_FILE="$1"
DESTINATION_USER="$2"
DESTINATION_HOST="$3"
DESTINATION_PATH="$4"
VERIFY_CHECKSUM="${5:-yes}"  # yes or no

# Validate input
if [ -z "$SOURCE_FILE" ] || [ -z "$DESTINATION_USER" ] || [ -z "$DESTINATION_HOST" ] || [ -z "$DESTINATION_PATH" ]; then
    echo "Error: Missing required parameters."
    echo "Usage: $0 source_file destination_user destination_host destination_path [verify_checksum]"
    echo "Example: $0 ~/encrypted_files/gpg_keys.tar.gz.gpg username 192.168.1.100 ~/gpg_transfer"
    exit 1
fi

if [ ! -f "$SOURCE_FILE" ]; then
    echo "Error: Source file '$SOURCE_FILE' not found."
    exit 1
fi

echo "Preparing to transfer $SOURCE_FILE to $DESTINATION_USER@$DESTINATION_HOST:$DESTINATION_PATH..."

# Calculate checksum before transfer
if [ "$VERIFY_CHECKSUM" = "yes" ]; then
    echo "Calculating source file checksum..."
    SOURCE_CHECKSUM=$(shasum -a 256 "$SOURCE_FILE" | cut -d ' ' -f 1)
    echo "Source checksum: $SOURCE_CHECKSUM"
fi

# Create destination directory if it doesn't exist
echo "Creating destination directory if needed..."
ssh "$DESTINATION_USER@$DESTINATION_HOST" "mkdir -p $DESTINATION_PATH && chmod 700 $DESTINATION_PATH"

# Transfer the file
echo "Transferring file..."
scp -v "$SOURCE_FILE" "$DESTINATION_USER@$DESTINATION_HOST:$DESTINATION_PATH/"

# Verify transfer
echo "Verifying transfer..."
FILENAME=$(basename "$SOURCE_FILE")
ssh "$DESTINATION_USER@$DESTINATION_HOST" "ls -la $DESTINATION_PATH/$FILENAME"

# Verify checksum after transfer
if [ "$VERIFY_CHECKSUM" = "yes" ]; then
    echo "Calculating destination file checksum..."
    DEST_CHECKSUM=$(ssh "$DESTINATION_USER@$DESTINATION_HOST" "sha256sum $DESTINATION_PATH/$FILENAME" | cut -d ' ' -f 1)
    echo "Destination checksum: $DEST_CHECKSUM"
    
    if [ "$SOURCE_CHECKSUM" = "$DEST_CHECKSUM" ]; then
        echo "Checksum verification successful!"
    else
        echo "ERROR: Checksum verification failed! The transferred file may be corrupted."
        exit 1
    fi
fi

echo "Transfer complete and verified."
```

What this script does:
- Takes source file and destination details as parameters
- Creates the destination directory if needed
- Transfers the file using SCP
- Verifies the transfer with checksums
- Provides detailed status updates

To use this script:
```bash
# Make it executable
chmod +x secure_transfer.sh

# Transfer a file
./secure_transfer.sh ~/encrypted_files/gpg_keys.tar.gz.gpg username 192.168.1.100 ~/gpg_transfer
```

## Automated Import and Verification

Here's a script to import GPG keys on the destination system:

```bash
#!/bin/bash
# gpg_import.sh - Import GPG keys from an encrypted archive

# Exit on any error
set -e

# Configuration
ENCRYPTED_ARCHIVE="$1"
WORK_DIR="${2:-$HOME/gpg_import_temp}"

# Validate input
if [ -z "$ENCRYPTED_ARCHIVE" ]; then
    echo "Error: No encrypted archive specified."
    echo "Usage: $0 encrypted_archive.gpg [work_directory]"
    exit 1
fi

if [ ! -f "$ENCRYPTED_ARCHIVE" ]; then
    echo "Error: Encrypted archive '$ENCRYPTED_ARCHIVE' not found."
    exit 1
fi

# Create temporary working directory with secure permissions
mkdir -p "$WORK_DIR"
chmod 700 "$WORK_DIR"

echo "Importing GPG keys from $ENCRYPTED_ARCHIVE..."

# Decrypt the archive
echo "Decrypting archive..."
gpg --decrypt "$ENCRYPTED_ARCHIVE" > "$WORK_DIR/gpg_keys.tar.gz"

# Extract the archive
echo "Extracting files..."
tar -xzf "$WORK_DIR/gpg_keys.tar.gz" -C "$WORK_DIR"

# Import public key
echo "Importing public key..."
gpg --import "$WORK_DIR/public_key.asc"

# Import private key
echo "Importing private key..."
gpg --import "$WORK_DIR/private_key.asc"

# Import trust database
echo "Importing trust database..."
gpg --import-ownertrust "$WORK_DIR/trust.txt"

# Verify import
echo "Verifying imported keys..."
gpg --list-secret-keys --keyid-format LONG

# Securely delete temporary files
echo "Cleaning up temporary files..."
if command -v shred &> /dev/null; then
    shred -u "$WORK_DIR/public_key.asc" "$WORK_DIR/private_key.asc" "$WORK_DIR/trust.txt" "$WORK_DIR/gpg_keys.tar.gz"
    rm -rf "$WORK_DIR"
else
    rm -rf "$WORK_DIR"
    echo "Warning: 'shred' not found. Files deleted but not securely overwritten."
fi

echo "Import complete. Verify your keys are working correctly."
```

What this script does:
- Decrypts and extracts the encrypted archive
- Imports the public key, private key, and trust database
- Verifies the import by listing the keys
- Securely deletes temporary files
- Provides status updates throughout the process

To use this script:
```bash
# Make it executable
chmod +x gpg_import.sh

# Import keys from an encrypted archive
./gpg_import.sh ~/gpg_transfer/gpg_keys.tar.gz.gpg
```

## Securing Automation Credentials

When automating sensitive operations, credential management is critical:

### Using Environment Variables (More Secure)

```bash
#!/bin/bash
# Example of using environment variables for credentials

# Set these before running the script
# export GPG_PASSPHRASE="your-secure-passphrase"
# export SSH_PASSWORD="your-ssh-password"

if [ -z "$GPG_PASSPHRASE" ]; then
    echo "Error: GPG_PASSPHRASE environment variable not set."
    exit 1
fi

# Use the passphrase from environment
echo "$GPG_PASSPHRASE" | gpg --batch --yes --passphrase-fd 0 --decrypt input.gpg
```

What this does:
- Uses environment variables instead of hardcoding credentials
- Avoids storing credentials in the script file
- Prevents credentials from appearing in process listings

### Using SSH Keys Instead of Passwords

```bash
#!/bin/bash
# Example of using SSH keys for authentication

# Generate SSH key if needed (run once, not in the script)
# ssh-keygen -t ed25519 -C "key-transfer-automation"

# Copy SSH key to destination (run once, not in the script)
# ssh-copy-id username@destination_host

# Now you can transfer without password prompts
scp -i ~/.ssh/id_ed25519 source_file.gpg username@destination_host:~/destination/
```

What this does:
- Uses SSH key authentication instead of passwords
- Eliminates the need to handle passwords in scripts
- Provides better security and auditability

### Using GPG Agent for Passphrase Handling

```bash
#!/bin/bash
# Example of using GPG agent

# Configure GPG agent (run once, not in the script)
# echo "default-cache-ttl 3600" >> ~/.gnupg/gpg-agent.conf
# echo "max-cache-ttl 7200" >> ~/.gnupg/gpg-agent.conf
# gpgconf --reload gpg-agent

# Now you can use GPG without storing passphrases
gpg --decrypt input.gpg
# GPG agent will prompt for passphrase once and remember it
```

What this does:
- Uses GPG agent to securely cache passphrases
- Avoids storing passphrases in scripts or environment
- Provides a balance of security and convenience

## Logging and Auditing Automated Transfers

Proper logging is essential for security and troubleshooting:

```bash
#!/bin/bash
# Example of secure logging for key transfers

# Configuration
LOG_FILE="$HOME/.key_transfer_logs/transfer_$(date +%Y%m%d_%H%M%S).log"
mkdir -p "$(dirname "$LOG_FILE")"
chmod 700 "$(dirname "$LOG_FILE")"

# Function to log messages
log_message() {
    echo "[$(date +"%Y-%m-%d %H:%M:%S")] $1" | tee -a "$LOG_FILE"
}

# Start logging
log_message "Starting key transfer process"
log_message "User: $(whoami)"
log_message "Source: $HOSTNAME"

# Perform operations, logging each step
log_message "Exporting public key"
gpg --export --armor "$KEY_ID" > public_key.asc 2>> "$LOG_FILE"

log_message "Creating encrypted archive"
tar -czf keys.tar.gz public_key.asc 2>> "$LOG_FILE"
gpg --symmetric --cipher-algo AES256 keys.tar.gz 2>> "$LOG_FILE"

log_message "Transferring to destination"
scp -v keys.tar.gz.gpg user@host:~/destination/ 2>> "$LOG_FILE"

# Log completion
log_message "Transfer completed successfully"

# Set secure permissions on log file
chmod 600 "$LOG_FILE"
```

What this script does:
- Creates a timestamped log file in a secure directory
- Logs each operation with timestamps
- Captures command output and errors
- Sets secure permissions on log files
- Provides an audit trail of all operations

### Security Considerations for Logs

Remember these important points about logs:
- Logs should never contain passphrases or private keys
- Log files should have restricted permissions
- Consider log rotation to prevent unlimited growth
- Regularly review logs for security issues
- Consider using a centralized logging system for critical operations

## Complete Automation Example

Here's a complete example that combines the concepts above into a full automation script:

```bash
#!/bin/bash
# complete_key_transfer.sh - Fully automated GPG key transfer

# Exit on any error
set -e

# Configuration
KEY_ID="YOUR_KEY_ID"
DESTINATION_USER="username"
DESTINATION_HOST="wsl_ip_address"
DESTINATION_PATH="~/gpg_transfer"
WORK_DIR="$HOME/gpg_transfer_temp"
LOG_FILE="$HOME/.key_transfer_logs/transfer_$(date +%Y%m%d_%H%M%S).log"

# Setup logging
mkdir -p "$(dirname "$LOG_FILE")"
chmod 700 "$(dirname "$LOG_FILE")"

# Function to log messages
log_message() {
    echo "[$(date +"%Y-%m-%d %H:%M:%S")] $1" | tee -a "$LOG_FILE"
}

# Start process
log_message "Starting automated GPG key transfer"
log_message "Key ID: $KEY_ID"
log_message "Destination: $DESTINATION_USER@$DESTINATION_HOST:$DESTINATION_PATH"

# Create working directory
mkdir -p "$WORK_DIR"
chmod 700 "$WORK_DIR"
log_message "Created working directory: $WORK_DIR"

# Export keys
log_message "Exporting public key"
gpg --export --armor "$KEY_ID" > "$WORK_DIR/public_key.asc"

log_message "Exporting private key"
gpg --export-secret-keys --armor "$KEY_ID" > "$WORK_DIR/private_key.asc"

log_message "Exporting trust database"
gpg --export-ownertrust > "$WORK_DIR/trust.txt"

# Create and encrypt archive
log_message "Creating archive"
tar -czf "$WORK_DIR/gpg_keys.tar.gz" -C "$WORK_DIR" public_key.asc private_key.asc trust.txt

log_message "Encrypting archive"
gpg --symmetric --cipher-algo AES256 "$WORK_DIR/gpg_keys.tar.gz"

# Calculate checksum
log_message "Calculating checksum"
SOURCE_CHECKSUM=$(shasum -a 256 "$WORK_DIR/gpg_keys.tar.gz.gpg" | cut -d ' ' -f 1)
log_message "Source checksum: $SOURCE_CHECKSUM"

# Create destination directory
log_message "Creating destination directory"
ssh "$DESTINATION_USER@$DESTINATION_HOST" "mkdir -p $DESTINATION_PATH && chmod 700 $DESTINATION_PATH"

# Transfer file
log_message "Transferring encrypted archive"
scp "$WORK_DIR/gpg_keys.tar.gz.gpg" "$DESTINATION_USER@$DESTINATION_HOST:$DESTINATION_PATH/"

# Verify checksum
log_message "Verifying checksum"
DEST_CHECKSUM=$(ssh "$DESTINATION_USER@$DESTINATION_HOST" "sha256sum $DESTINATION_PATH/gpg_keys.tar.gz.gpg" | cut -d ' ' -f 1)
log_message "Destination checksum: $DEST_CHECKSUM"

if [ "$SOURCE_CHECKSUM" = "$DEST_CHECKSUM" ]; then
    log_message "Checksum verification successful"
else
    log_message "ERROR: Checksum verification failed!"
    exit 1
fi

# Import on destination
log_message "Importing keys on destination"
ssh "$DESTINATION_USER@$DESTINATION_HOST" "bash -s" << 'EOF'
# Destination import script
ENCRYPTED_ARCHIVE="$HOME/gpg_transfer/gpg_keys.tar.gz.gpg"
WORK_DIR="$HOME/gpg_import_temp"

# Create working directory
mkdir -p "$WORK_DIR"
chmod 700 "$WORK_DIR"

# Decrypt and extract
gpg --decrypt "$ENCRYPTED_ARCHIVE" > "$WORK_DIR/gpg_keys.tar.gz"
tar -xzf "$WORK_DIR/gpg_keys.tar.gz" -C "$WORK_DIR"

# Import keys
gpg --import "$WORK_DIR/public_key.asc"
gpg --import "$WORK_DIR/private_key.asc"
gpg --import-ownertrust "$WORK_DIR/trust.txt"

# Clean up
rm -rf "$WORK_DIR"
EOF

# Clean up local files
log_message "Cleaning up temporary files"
rm -rf "$WORK_DIR"

log_message "Transfer and import completed successfully"
chmod 600 "$LOG_FILE"

echo "GPG key transfer completed successfully. See log at $LOG_FILE"
```

What this script does:
- Exports your GPG keys
- Creates and encrypts an archive
- Transfers the archive to the destination
- Verifies the transfer with checksums
- Imports the keys on the destination
- Cleans up temporary files
- Logs the entire process

## Security Considerations for Automation

When automating sensitive operations like key transfers, keep these security principles in mind:

1. **Minimize automation scope**: Only automate what's necessary
2. **Use principle of least privilege**: Scripts should have minimal permissions
3. **Secure credential handling**: Never hardcode credentials in scripts
4. **Comprehensive logging**: Log all operations for audit purposes
5. **Secure cleanup**: Always securely delete temporary files
6. **Verification**: Always verify transfers and imports
7. **Error handling**: Scripts should fail safely and report errors
8. **Regular review**: Periodically review automation for security issues

## Next Steps

After setting up automation for your GPG key transfers, you should:

1. Test your automation thoroughly in a safe environment
2. Document your automation procedures
3. Set up monitoring for your automated processes
4. Create a schedule for key rotation and updates
5. Regularly review logs and audit your key management

For more information on using your transferred keys, see [Working with Your Transferred GPG Keys in WSL](12-using-gpg-keys-in-wsl.md).

---

**Prerequisites**: 
- [Understanding GPG Keys and Secure Transfer Fundamentals](01-introduction-to-gpg-keys.md)
- [Successfully Importing GPG Keys into Windows WSL](07-importing-gpg-keys-in-wsl.md)

**Next Steps**: [Working with Your Transferred GPG Keys in WSL](12-using-gpg-keys-in-wsl.md)

**See Also**:
- [Security Best Practices for Key Management](09-security-best-practices.md)
- [Troubleshooting and FAQs](10-troubleshooting-and-faqs.md)
- [Command Reference for GPG Key Management](appendix-command-reference.md)
