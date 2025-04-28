# Understanding GPG Keys and Secure Transfer Fundamentals

## What are GPG Keys?

GPG (GNU Privacy Guard) is a free and open-source program that provides encryption and digital signatures. GPG keys are digital keys that allow you to:

- **Encrypt** messages or files so only specific people can read them
- **Decrypt** messages or files that were encrypted for you
- **Sign** messages or files to prove they came from you
- **Verify** signatures to confirm a message or file came from a specific person

Think of GPG keys like a sophisticated lockbox system:
- Your **public key** is like a padlock you can give to anyone
- Your **private key** is like the unique key that opens only your padlocks

## Components of GPG Keys

A complete GPG key setup consists of several important parts:

### 1. Public Key
- Can be freely shared with others
- Used by others to encrypt messages for you
- Used by others to verify your signatures
- Usually ends with `.asc` or `.gpg` when exported
- Often shared on key servers or websites

### 2. Private Key
- Must be kept secret at all times
- Used to decrypt messages encrypted with your public key
- Used to create digital signatures
- Protected by a passphrase (like a password)
- Should never be shared or transferred without encryption

### 3. Trust Database
- Contains information about how much you trust other people's keys
- Helps GPG decide whether to trust signatures
- Specific to your GPG installation
- Important to transfer if you want to preserve your trust settings

### 4. Configuration Files
- Control how GPG behaves
- Contain settings for encryption preferences
- May include key server information
- Located in the `.gnupg` directory in your home folder

## Why Transfer GPG Keys Securely?

Transferring your GPG keys between computers (like from your Mac to a Windows WSL system) requires careful security measures because:

1. **Your private key is extremely sensitive** - If someone obtains your private key, they can:
   - Decrypt any messages meant for you
   - Sign messages pretending to be you
   - Potentially access encrypted data or systems

2. **Trust can be compromised** - Insecure transfer methods could allow:
   - Tampering with your keys during transfer
   - Interception of your private key
   - Modification of your trust database

3. **Passphrases may be exposed** - If you're not careful during transfer:
   - Your passphrase might be visible or logged
   - Temporary files might contain sensitive information

## Overview of Transfer Methods

There are several secure ways to transfer GPG keys between systems:

### 1. SCP (Secure Copy Protocol)
- Command-line tool for securely transferring files
- Encrypts all data during transfer
- Uses SSH for authentication and security
- Good for direct transfers between computers on the same network

### 2. SFTP (Secure File Transfer Protocol)
- Interactive file transfer protocol
- Also uses SSH for security
- Allows browsing remote directories
- Good if you need to navigate the file system before transfer

### 3. Physical Media
- Using encrypted USB drives
- Physically moving the storage device between computers
- Good when computers aren't connected to the same network
- Requires additional encryption of the keys on the media

### 4. Temporary Cloud Storage
- Uploading encrypted key files to cloud storage
- Downloading on the destination system
- Only secure if keys are encrypted before upload
- Convenient for transfers between distant systems

## When You Might Need to Transfer GPG Keys

Common scenarios where you'd need to transfer your GPG keys include:

1. **Setting up a new computer** - When you get a new machine and want to use your existing keys

2. **Using multiple operating systems** - When you work across macOS, Windows, and Linux

3. **Development environments** - When you need to sign code or access encrypted data in WSL

4. **Backup purposes** - Creating secure backups of your keys to prevent loss

5. **Using password managers** - Programs like `pass` that use GPG for encryption

## Security Principles for Key Transfer

No matter which method you choose, always follow these principles:

1. **Never transfer unencrypted private keys** - Always encrypt your keys before transfer

2. **Use strong, unique passphrases** - Protect your keys with strong passphrases

3. **Use secure channels** - Only transfer over encrypted connections

4. **Delete temporary files** - Securely remove any temporary files created during the process

5. **Verify after transfer** - Check that keys were transferred correctly

In the next article, we'll cover how to prepare both your macOS and Windows WSL systems for the transfer process.

---

**Next Steps**: [Preparing Your Systems for Secure GPG Key Transfer](02-prerequisites-and-system-setup.md)

**See Also**:
- [Securing Exported Keys for Transfer](04-securing-exported-keys.md)
- [Security Best Practices for Key Management](09-security-best-practices.md)
