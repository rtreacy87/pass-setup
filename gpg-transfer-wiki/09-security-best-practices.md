# Maintaining GPG Key Security Across Multiple Systems

This guide covers best practices for managing your GPG keys securely when using them across multiple systems, including macOS and Windows WSL. Following these practices will help protect your keys from compromise and ensure they remain secure over time.

## Passphrase Management and Best Practices

Your passphrase is the primary protection for your private key. Here's how to manage it securely:

### Creating Strong Passphrases

A strong passphrase should be:
- **Long**: At least 12-15 characters
- **Complex**: Mix of uppercase, lowercase, numbers, and symbols
- **Unique**: Not used for any other purpose
- **Memorable**: Something you can remember without writing it down
- **Not based on personal information**: Avoid names, dates, or common words

Bad example: `password123`
Good example: `Tr4ff1c-L1ght$-Banana-Cl0ck`

```bash
# Test your passphrase strength (on macOS or WSL)
echo "Your potential passphrase" | cracklib-check
```

### Using a Password Manager

Consider using a password manager to store your GPG passphrase:

```bash
# Install a password manager in WSL
sudo apt install pass

# Initialize pass with your GPG key
pass init your.email@example.com

# Store your passphrase (ironically, protected by your GPG key)
pass insert gpg/master-key-passphrase
```

What this does:
- Creates an encrypted store for your passwords
- Uses your own GPG key for encryption
- Provides a secure way to store complex passphrases

### Changing Passphrases Periodically

It's good practice to change your passphrase periodically:

```bash
# Change your key's passphrase
gpg --passwd YOUR_KEY_ID
```

What this command does:
- Prompts for your current passphrase
- Asks for a new passphrase (twice)
- Updates the passphrase protection on your private key

## Key Rotation Policies

Regularly rotating your keys helps limit the impact of potential compromises:

### When to Rotate Keys

Consider rotating your keys:
- Every 1-2 years for regular use
- After a suspected compromise
- When moving to a new primary device
- When key technology becomes outdated

### Creating New Keys

```bash
# Generate a new key pair
gpg --full-generate-key
```

What this does:
- Walks you through creating a new key pair
- Lets you specify key type, size, expiration, and user ID
- Generates a completely new key, not connected to your old one

### Transitioning to New Keys

After creating a new key:

```bash
# Export your new public key
gpg --export --armor NEW_KEY_ID > new_public_key.asc

# Sign your new key with your old key to create a trust path
gpg --default-key OLD_KEY_ID --sign-key NEW_KEY_ID

# Create a revocation certificate for your old key
gpg --gen-revoke OLD_KEY_ID > old_key_revocation.asc
```

What these commands do:
- Export your new public key to share with others
- Create a trust link between your old and new keys
- Prepare a revocation certificate for your old key

### Informing Contacts

Let your contacts know about your new key:
1. Share your new public key
2. Sign emails with both old and new keys during transition
3. Explain when you'll stop using the old key

## Using Subkeys Instead of Transferring Master Keys

Subkeys provide a more secure alternative to transferring your master key:

### Understanding Subkeys

Subkeys are additional keys associated with your master key that:
- Can be used for specific purposes (signing, encryption, authentication)
- Can be revoked individually if compromised
- Limit exposure of your master key

Think of your master key as your identity, and subkeys as limited-purpose credentials.

### Creating Subkeys

```bash
# Edit your key
gpg --expert --edit-key YOUR_KEY_ID

# At the gpg> prompt:
addkey
# Follow prompts to create a signing subkey
# Select RSA (sign only)
# Choose 4096 bits
# Set an expiration date

addkey
# Follow prompts to create an encryption subkey
# Select RSA (encrypt only)
# Choose 4096 bits
# Set an expiration date

addkey
# Follow prompts to create an authentication subkey
# Select RSA (authenticate only)
# Choose 4096 bits
# Set an expiration date

save
```

What this does:
- Creates separate subkeys for signing, encryption, and authentication
- Each subkey has a specific purpose
- Each can have its own expiration date

### Exporting Only Subkeys

Instead of transferring your master key, export only the subkeys:

```bash
# Export only your subkeys
gpg --export-secret-subkeys --armor YOUR_KEY_ID > subkeys.asc
```

What this does:
- Exports only your subkeys, not your master key
- The exported file can be transferred to other systems
- Your master key remains secure on your primary system

### Using Subkeys on Secondary Systems

On your WSL system, after importing subkeys:

```bash
# Import the subkeys
gpg --import subkeys.asc

# Verify that only subkeys were imported
gpg --list-secret-keys --keyid-format LONG
```

In the output, look for:
- `sec#` instead of `sec` (the # indicates the master key is not present)
- `ssb` entries for your subkeys

This approach means that even if your WSL system is compromised, your master key remains safe.

## Limiting Key Validity Periods

Setting expiration dates on your keys provides several security benefits:

### Benefits of Expiration Dates

- Forces regular review of your key security
- Limits the usefulness of compromised keys
- Encourages key rotation
- Prevents indefinite use of outdated cryptography

### Setting Expiration on New Keys

When creating a new key:

```bash
# During key creation, you'll be asked about expiration
# Choose a reasonable period (1-2 years)
gpg --full-generate-key
```

### Updating Expiration on Existing Keys

```bash
# Edit your key
gpg --edit-key YOUR_KEY_ID

# At the gpg> prompt:
expire
# Follow prompts to set a new expiration date

# For subkeys, first select the subkey:
key 1  # Selects the first subkey
expire
# Set expiration

# Save changes
save
```

What this does:
- Updates the expiration date on your key or subkey
- Requires re-sharing your public key with updated expiration

### Extending Expiration Before It Occurs

Before your key expires:

```bash
# Edit your key
gpg --edit-key YOUR_KEY_ID

# At the gpg> prompt:
expire
# Set a new expiration date
save
```

Remember to distribute your updated public key to your contacts.

## Backup Strategies for Keys

Proper backups ensure you don't lose access to encrypted data:

### Creating Secure Backups

```bash
# Export your complete key
gpg --export-secret-keys --armor YOUR_KEY_ID > master_key_backup.asc

# Export your trust database
gpg --export-ownertrust > trust_backup.txt

# Encrypt the backup
gpg --symmetric --cipher-algo AES256 master_key_backup.asc
gpg --symmetric --cipher-algo AES256 trust_backup.txt
```

What these commands do:
- Create complete exports of your keys and trust settings
- Encrypt them with a strong passphrase
- The resulting `.gpg` files can be stored securely

### Secure Storage Locations

Store your encrypted backups in multiple secure locations:
- Encrypted USB drives kept in secure physical locations
- Password-protected cloud storage (different from your regular accounts)
- Paper backups for emergency recovery (see below)

### Paper Backups (For Emergency Recovery)

For critical keys, consider creating a paper backup:

```bash
# Create a paper-friendly backup
gpg --export-secret-keys --armor YOUR_KEY_ID | paperkey --output paper-key-backup.txt
```

What this does:
- `paperkey` extracts the essential parts of your private key
- Creates a text file that can be printed
- The printed copy can be stored in a secure physical location (like a safe)

To restore from a paper backup:
```bash
# First import your public key
gpg --import public_key.asc

# Then restore the private key from the paper backup
paperkey --pubring ~/.gnupg/pubring.gpg --secrets paper-key-backup.txt --output restored_key.gpg
gpg --import restored_key.gpg
```

## Revoking Compromised Keys

If you believe your key has been compromised:

### Using a Revocation Certificate

```bash
# If you previously created a revocation certificate
gpg --import revocation_certificate.asc

# If not, create and use one immediately
gpg --gen-revoke YOUR_KEY_ID > revocation.asc
gpg --import revocation.asc
```

What this does:
- Marks your key as revoked in your local keyring
- When you distribute this revoked public key, others will know not to use it

### Distributing the Revocation

```bash
# Export your revoked public key
gpg --export --armor YOUR_KEY_ID > revoked_key.asc

# Share this with contacts and key servers
```

### Creating a New Key After Compromise

After revoking a compromised key:
1. Generate a completely new key (don't just create a new subkey)
2. Use different user IDs if possible
3. Use a completely different passphrase
4. Review your security practices to prevent future compromises

## Audit and Monitoring Considerations

Regularly audit your GPG setup to maintain security:

### Regular Security Checks

```bash
# List all keys and check for unexpected entries
gpg --list-keys

# Check key permissions
ls -la ~/.gnupg/

# Verify key expiration dates
gpg --list-keys --with-colons | grep "^pub" | cut -d: -f7
```

### Monitoring Key Usage

```bash
# Check your public key fingerprint
gpg --fingerprint YOUR_KEY_ID

# Periodically search for your key on key servers
gpg --search-keys your.email@example.com
```

What this does:
- Helps you verify your key hasn't been tampered with
- Allows you to check if unauthorized copies exist on key servers

### Logging GPG Operations

Enable logging to track GPG usage:

```bash
# Edit your gpg.conf file
echo "log-file ~/.gnupg/gpg.log" >> ~/.gnupg/gpg.conf
echo "verbose" >> ~/.gnupg/gpg.conf
```

What this does:
- Creates a log of GPG operations
- Helps identify unusual activity
- Provides an audit trail of key usage

## Maintaining Different Security Levels Across Systems

When using keys on multiple systems:

### Primary System (Highest Security)

Your primary system (e.g., your personal Mac) should:
- Hold your master key
- Have the strongest physical security
- Use full disk encryption
- Have limited network exposure

### Secondary Systems (Medium Security)

Secondary systems (like your WSL environment) should:
- Only have subkeys, not your master key
- Use shorter expiration periods for those subkeys
- Have separate passphrases if possible

### Temporary Systems (Lowest Security)

For temporary or shared systems:
- Use one-time subkeys with short expirations
- Consider using keys stored on hardware tokens
- Remove keys completely when finished

## Next Steps

After implementing these security practices, you'll have a robust system for managing your GPG keys across multiple devices. For practical applications of your securely transferred keys, see [Working with Your Transferred GPG Keys in WSL](12-using-gpg-keys-in-wsl.md).

---

**Prerequisites**: 
- [Understanding GPG Keys and Secure Transfer Fundamentals](01-introduction-to-gpg-keys.md)
- [Successfully Importing GPG Keys into Windows WSL](07-importing-gpg-keys-in-wsl.md)

**Next Steps**: [Troubleshooting and FAQs](10-troubleshooting-and-faqs.md)

**See Also**:
- [Working with Your Transferred GPG Keys in WSL](12-using-gpg-keys-in-wsl.md)
- [Command Reference for GPG Key Management](appendix-command-reference.md)
