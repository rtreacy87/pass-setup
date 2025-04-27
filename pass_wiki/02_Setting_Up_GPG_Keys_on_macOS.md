# Setting Up GPG Keys on macOS

## Introduction

GPG (GNU Privacy Guard) is the encryption tool that Pass uses to keep your passwords secure. In this guide, we'll install GPG on macOS and create a key pair that will be used to encrypt and decrypt your passwords.

## Installing Required Software

### Step 1: Install Homebrew

Homebrew is a package manager for macOS that makes it easy to install software. If you don't already have it installed, open Terminal (you can find it in Applications > Utilities > Terminal) and run:

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

This command downloads and runs the Homebrew installation script. It will:
- Ask for your password (this is your Mac login password)
- Install the necessary files
- Set up Homebrew on your system

Follow any on-screen instructions to complete the installation.

### Step 2: Install GPG

With Homebrew installed, you can now install GPG by running:

```bash
brew install gnupg
```

This command:
- Downloads the GPG software package
- Installs it on your system
- Sets up the necessary configurations

### Step 3: Verify GPG Installation

To make sure GPG was installed correctly, run:

```bash
gpg --version
```

You should see output showing the version of GPG installed, something like:
```
gpg (GnuPG) 2.3.6
libgcrypt 1.10.1
```

## Generating a GPG Key Pair

Now that GPG is installed, we'll create a key pair (a private key and a public key) that will be used for encryption.

### Step 1: Generate a New Key Pair

Run the following command to start the key generation process:

```bash
gpg --full-generate-key
```

This command launches an interactive process to create your keys.

### Step 2: Choose Key Type

When prompted for the kind of key you want, select the default option (RSA and RSA) by pressing Enter:

```
Please select what kind of key you want:
   (1) RSA and RSA (default)
   (2) DSA and Elgamal
   (3) DSA (sign only)
   (4) RSA (sign only)
  (14) Existing key from card
Your selection? 1
```

RSA is a strong encryption algorithm suitable for password management.

### Step 3: Choose Key Size

When asked about the key size, type `4096` and press Enter:

```
RSA keys may be between 1024 and 4096 bits long.
What keysize do you want? (3072) 4096
```

A 4096-bit key provides strong security for your passwords.

### Step 4: Set Key Expiration

When asked about the key validity period, you can choose:
- `0` for a key that never expires
- A number followed by `d` (days), `w` (weeks), `m` (months), or `y` (years)

For security reasons, it's recommended to set an expiration date:

```
Please specify how long the key should be valid.
         0 = key does not expire
      <n>  = key expires in n days
      <n>w = key expires in n weeks
      <n>m = key expires in n months
      <n>y = key expires in n years
Key is valid for? (0) 2y
```

Here, we've set the key to expire in 2 years. You can always extend this later.

Confirm your choice by typing `y` when prompted.

### Step 5: Enter User Information

Next, you'll be asked to provide:
- Your real name
- Email address
- A comment (optional)

This information helps identify your key:

```
Real name: Your Full Name
Email address: your.email@example.com
Comment: macOS password key
```

Review the information and type `O` (for "Okay") when prompted.

### Step 6: Set a Passphrase

You'll be prompted to enter a passphrase for your key. This is extremely important as it protects your private key:

- Choose a strong, unique passphrase
- Make it something you can remember
- Consider using a passphrase (multiple words) rather than a simple password
- Don't reuse passwords from other services

Your passphrase won't be visible as you type it.

### Step 7: Generate Entropy

GPG will now generate your key pair. It may ask you to perform some actions on your computer to generate randomness (entropy), such as typing or moving your mouse.

Once complete, you'll see a message confirming your key has been created:

```
gpg: key 1A2B3C4D5E6F7G8H marked as ultimately trusted
gpg: revocation certificate stored as '/Users/yourusername/.gnupg/openpgp-revocs.d/1A2B3C4D5E6F7G8H.rev'
public and secret key created and signed.
```

## Finding Your GPG Key ID

To use your key with Pass, you'll need to know its ID. Run:

```bash
gpg --list-secret-keys --keyid-format LONG
```

This command:
- Lists all your secret (private) keys
- Displays the key IDs in the long format

You'll see output similar to:

```
/Users/yourusername/.gnupg/pubring.kbx
-------------------------------------
sec   rsa4096/1A2B3C4D5E6F7G8H 2023-01-01 [SC] [expires: 2025-01-01]
      ABCDEF0123456789ABCDEF0123456789ABCDEF01
uid                 [ultimate] Your Full Name (macOS password key) <your.email@example.com>
ssb   rsa4096/8H7G6F5E4D3C2B1A 2023-01-01 [E] [expires: 2025-01-01]
```

The part after `rsa4096/` on the `sec` line is your key ID (in this example, `1A2B3C4D5E6F7G8H`). Make note of this ID as you'll need it when setting up Pass.

## Configuring GPG Agent

The GPG agent manages your keys and remembers your passphrase temporarily so you don't have to enter it every time.

### Step 1: Create or Edit the GPG Agent Configuration

Create or edit the GPG agent configuration file:

```bash
mkdir -p ~/.gnupg
touch ~/.gnupg/gpg-agent.conf
nano ~/.gnupg/gpg-agent.conf
```

These commands:
1. Create the `.gnupg` directory if it doesn't exist
2. Create or open the `gpg-agent.conf` file
3. Open the file in the nano text editor

### Step 2: Add Configuration Settings

Add the following lines to the file:

```
default-cache-ttl 3600
max-cache-ttl 86400
```

These settings:
- `default-cache-ttl 3600`: Cache your passphrase for 1 hour (3600 seconds)
- `max-cache-ttl 86400`: Set the maximum cache time to 24 hours (86400 seconds)

Save the file by pressing `Ctrl+O`, then `Enter`, and exit nano with `Ctrl+X`.

### Step 3: Restart the GPG Agent

Restart the GPG agent to apply the changes:

```bash
gpgconf --kill gpg-agent
```

This command stops the current GPG agent process, which will automatically restart when needed.

## Testing Your GPG Setup

Let's make sure everything is working correctly:

```bash
echo "Test message" | gpg --encrypt --armor --recipient your.email@example.com | gpg --decrypt
```

This command:
1. Creates a test message
2. Encrypts it using your public key
3. Immediately decrypts it using your private key

You should be prompted for your passphrase (unless the agent has cached it), and then see "Test message" as the output.

## Conclusion

Congratulations! You've successfully:
- Installed GPG on your macOS system
- Generated a secure key pair
- Configured the GPG agent
- Tested your setup

Your GPG keys are now ready to be used with Pass. In the next guide, we'll install Pass and initialize your password store using these keys.

Remember to keep your private key secure and never share it with anyone. Also, make sure you don't forget your passphrase, as there's no way to recover it if lost.
