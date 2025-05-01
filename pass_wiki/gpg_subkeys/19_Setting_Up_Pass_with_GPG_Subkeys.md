# Setting Up Pass with GPG Subkeys on Secondary Devices

## Introduction

Now that you've exported your GPG subkeys, it's time to set them up on your secondary devices and configure `pass` to use them. This guide will walk you through importing your subkeys, configuring `pass`, and verifying that everything is working correctly.

The key advantage of this approach is that your secondary devices will only have your subkeys, not your master key. This means that even if one of these devices is compromised, your master GPG identity remains secure.

## Importing Subkeys on Secondary Devices

We'll start by importing the subkeys you exported in the previous guide. These instructions assume you've already transferred the encrypted archive (`subkey-export.tar.gz.gpg`) to your secondary device.

### Step 1: Decrypt the Archive

First, decrypt the archive containing your exported keys:

```bash
# Navigate to where you stored the encrypted archive
cd ~/Downloads  # or wherever you saved it

# Decrypt the archive
gpg -d subkey-export.tar.gz.gpg > subkey-export.tar.gz
```

You'll be prompted to enter the password you created when encrypting the archive.

### Step 2: Extract the Archive

Extract the contents of the archive:

```bash
# Create a directory for the extracted files
mkdir -p ~/subkey-import
chmod 700 ~/subkey-import

# Extract the archive
tar -xzf subkey-export.tar.gz -C ~/
```

This will extract the files to the `~/subkey-export` directory.

### Step 3: Import the Public Key

First, import your public key:

```bash
gpg --import ~/subkey-export/public-key.asc
```

You should see output like:
```
gpg: key 1A2B3C4D5E6F7G8H: public key "Your Name <your.email@example.com>" imported
gpg: Total number processed: 1
gpg:               imported: 1
```

### Step 4: Import the Secret Subkeys

Next, import your secret subkeys:

```bash
gpg --import ~/subkey-export/secret-subkeys.asc
```

You should see output like:
```
gpg: key 1A2B3C4D5E6F7G8H: secret key imported
gpg: Total number processed: 1
gpg:              unchanged: 1
gpg:       secret keys read: 1
gpg:   secret keys imported: 1
```

### Step 5: Import the Trust Database

Finally, import your trust database:

```bash
gpg --import-ownertrust ~/subkey-export/trust.txt
```

### Step 6: Verify the Imported Keys

Check that your keys were imported correctly:

```bash
gpg --list-secret-keys --keyid-format LONG
```

You should see output similar to:

```
sec#  rsa4096/1A2B3C4D5E6F7G8H 2023-01-01 [SC] [expires: 2025-01-01]
      ABCDEF0123456789ABCDEF0123456789ABCDEF01
uid                 [ultimate] Your Name <your.email@example.com>
ssb   rsa4096/2B1A8H7G6F5E4D3C 2023-06-01 [E] [expires: 2024-06-01]
ssb   rsa4096/6F5E4D3C2B1A8H7G 2023-06-01 [S] [expires: 2024-06-01]
```

The `sec#` entry (note the `#` symbol) indicates that the primary key is not available on this device - only a reference to it exists. This is exactly what we want! The `ssb` entries show your subkeys, which are available for use.

## Configuring Pass to Use Subkeys

Now that your subkeys are imported, let's set up `pass` to use them.

### Step 1: Install Pass (if not already installed)

If you haven't already installed `pass` on this device, do so now:

**On Ubuntu/Debian:**
```bash
sudo apt-get update
sudo apt-get install pass
```

**On macOS:**
```bash
brew install pass
```

**On other systems:**
Follow the installation instructions for your specific operating system.

### Step 2: Identify the Correct Key ID to Use

You need to use your GPG key ID when initializing `pass`. This should be your master key ID, even though only the subkeys are present on this device:

```bash
gpg --list-secret-keys --keyid-format LONG
```

From the output, note your master key ID (in our example, `1A2B3C4D5E6F7G8H`).

### Step 3: Initialize Pass with Your Key

Initialize `pass` with your key ID:

```bash
pass init 1A2B3C4D5E6F7G8H
```

Replace `1A2B3C4D5E6F7G8H` with your actual key ID.

You should see output like:
```
Password store initialized for 1A2B3C4D5E6F7G8H
```

### Step 4: Test Encryption and Decryption

Let's test that `pass` can encrypt and decrypt passwords using your subkeys:

```bash
# Create a test password
pass insert test/subkey-test
```

You'll be prompted to enter a password. Enter something simple for testing, like "testpassword".

Now try to view the password:

```bash
pass test/subkey-test
```

If you see your password displayed, congratulations! Your subkey setup is working correctly.

## Handling Git Operations with Subkeys

If you use Git with `pass` for synchronization, you'll need to configure it to work with your subkeys.

### Step 1: Initialize Git in Your Password Store (if not already done)

If you haven't already set up Git with your password store:

```bash
pass git init
```

### Step 2: Configure Git to Use Your Signing Subkey

If you created a signing subkey and want to use it to sign Git commits:

```bash
# Configure Git to use your signing key
pass git config user.signingkey 6F5E4D3C2B1A8H7G
```

Replace `6F5E4D3C2B1A8H7G` with the ID of your signing subkey.

### Step 3: Configure Git to Sign Commits

```bash
pass git config commit.gpgsign true
```

This tells Git to sign all commits in your password store.

### Step 4: Test Git Operations

Let's test that Git operations work correctly:

```bash
# Make a change to your password store
pass insert test/git-test

# Commit the change
pass git add .
pass git commit -m "Test commit with subkey"
```

If the commit succeeds without errors, your Git configuration is working correctly with your subkeys.

### Step 5: Configure Remote Repository (if using)

If you're using a remote Git repository to synchronize your passwords:

```bash
# Add your remote repository
pass git remote add origin YOUR_REPOSITORY_URL

# Push your changes
pass git push -u origin master
```

Replace `YOUR_REPOSITORY_URL` with the URL of your Git repository.

## Verifying Proper Subkey Usage

Let's verify that your setup is using subkeys correctly and that your master key is indeed not present on this device.

### Checking Which Keys Are Being Used

When you encrypt or decrypt a password, `pass` uses your GPG encryption subkey. You can see this in action with verbose output:

```bash
# Enable verbose output for pass
export PASSWORD_STORE_ENABLE_EXTENSIONS=true

# Create a new password with verbose output
pass -v insert test/verbose-test
```

In the output, you should see references to your encryption subkey being used.

### Confirming Primary Key Absence

To confirm that your primary key is not present on this device:

```bash
# Try to use your primary key directly
echo "test" | gpg --sign -u 1A2B3C4D5E6F7G8H
```

You should see an error like:
```
gpg: signing failed: No secret key
gpg: [stdin]: clear-sign failed: No secret key
```

This confirms that your primary key is not available on this device, which is what we want for security.

### Testing Pass Operations

Finally, let's test some common `pass` operations to ensure everything works:

```bash
# List all passwords
pass

# Generate a new password
pass generate test/generated-password 15

# Edit a password
pass edit test/subkey-test

# Remove a test password
pass rm test/verbose-test
```

If all these operations work without errors, your `pass` setup with subkeys is fully functional!

## Troubleshooting Common Issues

If you encounter problems, here are some common issues and their solutions:

### Issue: "gpg: decryption failed: No secret key"

This error occurs when trying to decrypt a password, but the corresponding encryption subkey is not available.

**Solution:**
- Verify that your encryption subkey was properly imported:
  ```bash
  gpg --list-secret-keys --keyid-format LONG
  ```
- Check that the encryption subkey (marked with [E]) is listed
- If not, re-import your secret subkeys

### Issue: "error: gpg failed to sign the data"

This error occurs when trying to sign Git commits but your signing subkey is not properly configured.

**Solution:**
- Verify that your signing subkey was properly imported:
  ```bash
  gpg --list-secret-keys --keyid-format LONG
  ```
- Check that the signing subkey (marked with [S]) is listed
- Ensure you've configured Git with the correct signing key ID:
  ```bash
  pass git config user.signingkey YOUR_SIGNING_SUBKEY_ID
  ```

### Issue: "gpg: WARNING: unsafe ownership on homedir"

This warning indicates that your GPG home directory has incorrect permissions.

**Solution:**
```bash
chmod 700 ~/.gnupg
```

## Conclusion

Congratulations! You've successfully:
- Imported your GPG subkeys on your secondary device
- Configured `pass` to use these subkeys
- Set up Git integration with your signing subkey
- Verified that your setup is working correctly
- Learned how to troubleshoot common issues

Your secondary device is now set up to use `pass` with only your subkeys, not your master key. This provides a good balance of security and convenience - you can use `pass` normally on this device, but if the device is compromised, your master GPG identity remains secure.

In the next guide, we'll cover how to handle subkey rotation and maintenance over time.

## Navigation

- [README](../README.md) - Wiki Home
- Previous: [Exporting GPG Subkeys for Other Devices](18_Exporting_GPG_Subkeys_for_Other_Devices.md)
- Next: [Subkey Rotation and Maintenance](20_Subkey_Rotation_and_Maintenance.md)
- Related: [Setting Up Pass on WSL Ubuntu](../08_Setting_Up_Pass_on_WSL_Ubuntu.md)
