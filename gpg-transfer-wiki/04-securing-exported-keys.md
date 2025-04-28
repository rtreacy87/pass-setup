# Protecting Your GPG Keys During Transfer

This guide explains how to secure your exported GPG keys before transferring them between systems. We'll focus on using encryption and secure handling practices to protect your sensitive key material.

## Why Encryption is Necessary for Key Transfer

Your GPG private key is one of the most sensitive pieces of digital information you own. Even though you'll be using secure transfer methods (like SCP or SFTP), adding an extra layer of encryption provides:

1. **Defense in depth** - Multiple layers of security in case one fails
2. **Protection from accidental exposure** - Encrypted files are useless without the password
3. **Protection during storage** - Keys remain secure while waiting to be transferred
4. **Protection from untrusted intermediaries** - If your transfer goes through any servers or services

## Creating Password-Protected Archives

The first step is to combine your exported key files into a single archive and encrypt it:

```bash
# Navigate to where your exported keys are stored
cd ~/Desktop/gpg_transfer

# Create a tar archive containing all your key files
tar -czf gpg_keys.tar.gz public_key.asc private_key.asc trust.txt
```

What this command does:
- `tar` is a program that combines multiple files into one archive
- `-c` creates a new archive
- `-z` compresses the archive with gzip
- `-f gpg_keys.tar.gz` specifies the output filename
- The remaining arguments are the files to include in the archive

## Choosing Strong Passphrases

Before encrypting your archive, you need to choose a strong passphrase:

A strong passphrase should:
- Be at least 12 characters long
- Include a mix of uppercase letters, lowercase letters, numbers, and symbols
- Not be based on dictionary words
- Not be used for anything else
- Be something you can remember without writing it down

Bad example: `password123`
Good example: `T5%bVn$K9p@LzQ2&`

## Using GPG to Encrypt the Key Archive

Now, encrypt the archive with a strong passphrase:

```bash
# Encrypt the archive with a symmetric cipher
gpg --symmetric --cipher-algo AES256 gpg_keys.tar.gz
```

What this command does:
- `--symmetric` uses symmetric encryption (password-based)
- `--cipher-algo AES256` specifies the AES-256 encryption algorithm (very strong)
- GPG will prompt you to enter a passphrase twice
- Creates a new file called `gpg_keys.tar.gz.gpg`

When you run this command:
1. You'll be prompted to enter a passphrase
2. You'll be asked to confirm the passphrase by entering it again
3. GPG will encrypt the file using your passphrase

The result will be a file named `gpg_keys.tar.gz.gpg` that can only be decrypted with your passphrase.

## Secure Handling of Temporary Files

After creating the encrypted archive, you should securely delete the original unencrypted files:

```bash
# On macOS, if you have 'srm' installed (secure remove)
srm -v public_key.asc private_key.asc trust.txt gpg_keys.tar.gz

# If you don't have 'srm', use the built-in 'rm' with secure options
rm -P public_key.asc private_key.asc trust.txt gpg_keys.tar.gz
```

What these commands do:
- `srm -v` securely removes files by overwriting them multiple times before deletion
- The `-v` flag makes it verbose, showing what it's doing
- `rm -P` on macOS overwrites the file three times before removing it

Why this matters:
- Regular deletion doesn't actually remove the file data from disk
- Secure deletion makes it much harder to recover the deleted files
- This prevents someone from recovering your private key later

## Verifying the Integrity of Encrypted Archives

Before proceeding, verify that your encrypted archive was created successfully:

```bash
# Check that the encrypted file exists and has a reasonable size
ls -la gpg_keys.tar.gz.gpg

# Test that you can decrypt it
gpg --decrypt gpg_keys.tar.gz.gpg > /dev/null
```

What these commands do:
- `ls -la` shows file details including size and permissions
- The decrypt command tests that you can decrypt the file
- `> /dev/null` discards the output (we're just testing decryption)
- You'll be prompted for your passphrase

If the decryption test works without errors, your file is properly encrypted and can be decrypted with your passphrase.

## Additional Security Measures

### Splitting the Archive (Optional)

For extremely sensitive keys, you might want to split the encrypted archive into parts:

```bash
# Split the encrypted file into 1MB chunks
split -b 1m gpg_keys.tar.gz.gpg gpg_keys_part_
```

What this does:
- Splits the file into multiple parts of 1MB each
- Creates files named gpg_keys_part_aa, gpg_keys_part_ab, etc.
- You'll need to transfer all parts and reassemble them

To reassemble on the destination system:
```bash
cat gpg_keys_part_* > gpg_keys.tar.gz.gpg
```

### Using Different Transfer Methods for Different Parts

For maximum security, you could:
- Send some parts via SCP
- Send other parts via a different secure channel
- This ensures no single compromise exposes all parts

## Preparing for Transfer

Now that your keys are securely encrypted, you're ready to transfer them:

```bash
# Move the encrypted file to an easy-to-find location
mv gpg_keys.tar.gz.gpg ~/Desktop/
```

What to remember:
- Keep track of the encrypted file's location
- Remember the passphrase you used (don't write it down)
- You'll need both to decrypt the archive on the destination system

## Handling the Passphrase

The passphrase you used to encrypt the archive is now critical:
- **Don't write it down** in an unsecured location
- **Don't send it** through the same channel as the encrypted file
- **Do memorize it** if possible
- If you must share it with someone else, do so through a different secure channel than the one used for the file transfer

## Common Mistakes to Avoid

1. **Using weak encryption** - Always use strong algorithms like AES-256
2. **Reusing passphrases** - Use a unique passphrase for this purpose
3. **Leaving unencrypted files** - Always securely delete the original files
4. **Sending the passphrase with the file** - Use separate channels
5. **Using predictable passphrases** - Avoid personal information or patterns

## Next Steps

After securing your exported keys, you're ready to transfer them to your WSL system using either SCP or SFTP, which we'll cover in the next guides.

Remember: The security of your encrypted archive is only as strong as the passphrase protecting it. Choose a strong passphrase and keep it secure.

---

**Prerequisites**: 
- [Understanding GPG Keys and Secure Transfer Fundamentals](01-introduction-to-gpg-keys.md)
- [Safely Exporting Your GPG Keys from macOS](03-exporting-gpg-keys-from-macos.md)

**Next Steps**: 
- [Step-by-Step Guide to Transferring GPG Keys with SCP](05-transferring-keys-using-scp.md)
- [Using SFTP to Securely Move GPG Keys Between Systems](06-transferring-keys-using-sftp.md)

**See Also**:
- [Security Best Practices for Key Management](09-security-best-practices.md)
- [Alternative Secure Ways to Transfer GPG Keys](08-alternative-transfer-methods.md)
