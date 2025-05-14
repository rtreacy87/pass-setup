# Extending to Additional Platforms

## Introduction

One of the key advantages of using Pass for password management is its flexibility to work across multiple platforms. In this guide, we'll explore how to extend your password management system beyond macOS and WSL Ubuntu to include additional systems such as Linux desktops, more macOS systems, mobile devices, and servers.

## Adding Linux Desktop Systems

Linux is Pass's native environment, making it straightforward to set up.

### Installation on Linux

Most Linux distributions include Pass in their package repositories:

#### Debian/Ubuntu-based Systems

```bash
sudo apt update
sudo apt install -y pass gnupg2 git
```

#### Fedora/RHEL-based Systems

```bash
sudo dnf install pass gnupg2 git
```

#### Arch Linux

```bash
sudo pacman -S pass gnupg git
```

### Importing Your GPG Keys

Transfer your GPG keys to the Linux system using the same secure methods described in the "Exporting GPG Keys" guide:

1. Copy your encrypted key archive to the Linux system
2. Decrypt and extract the keys:
   ```bash
   gpg -d gpg-keys.tar.gz.gpg > gpg-keys.tar.gz
   tar -xzf gpg-keys.tar.gz
   ```

3. Import the keys:
   ```bash
   gpg --import public-key.asc
   gpg --import private-key.asc
   gpg --import-ownertrust trust.txt
   ```

4. Verify the import:
   ```bash
   gpg --list-secret-keys --keyid-format LONG
   ```

### Setting Up Pass

Initialize Pass with your imported GPG key:

```bash
pass init "YOUR_GPG_KEY_ID"
```

Then clone your password repository:

```bash
rm -rf ~/.password-store
git clone git@github.com:yourusername/password-store.git ~/.password-store
```

### Configuring Git Integration

Enable automatic Git operations:

```bash
echo 'export PASSWORD_STORE_ENABLE_GIT=true' >> ~/.bashrc
source ~/.bashrc
```

Configure Git in your password store:

```bash
cd ~/.password-store
git config user.name "Your Name"
git config user.email "your.email@example.com"
```

## Adding Additional macOS Systems

Adding another macOS system follows a similar process to your initial setup.

### Installation

Install the required software:

```bash
# Install Homebrew if not already installed
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# Install GPG, Pass, and Git
brew install gnupg pass git
```

### Importing Your GPG Keys

Transfer and import your keys as described in previous guides:

```bash
# Import keys
gpg --import public-key.asc
gpg --import private-key.asc
gpg --import-ownertrust trust.txt

# Verify import
gpg --list-secret-keys --keyid-format LONG
```

### Setting Up Pass

Initialize and clone your password store:

```bash
# Initialize Pass
pass init "YOUR_GPG_KEY_ID"

# Remove the empty store
rm -rf ~/.password-store

# Clone your repository
git clone git@github.com:yourusername/password-store.git ~/.password-store
```

### Configuring Git Integration

Enable automatic Git operations:

```bash
echo 'export PASSWORD_STORE_ENABLE_GIT=true' >> ~/.zshrc
# Or for Bash:
# echo 'export PASSWORD_STORE_ENABLE_GIT=true' >> ~/.bashrc
source ~/.zshrc
```

## Mobile Device Integration

### Android Integration with Password Store App

The Password Store app for Android provides a mobile interface to your Pass password store.

#### Installation

1. Install [Password Store](https://play.google.com/store/apps/details?id=dev.msfjarvis.aps) from Google Play
2. Install [OpenKeychain](https://play.google.com/store/apps/details?id=org.sufficientlysecure.keychain) for GPG support

#### Importing Your GPG Key to Android

1. Export your GPG key to your computer:
   ```bash
   gpg --export-secret-keys --armor YOUR_GPG_KEY_ID > mobile-key.asc
   ```

2. Transfer this file to your Android device (via USB, secure file transfer, or encrypted email)

3. In OpenKeychain:
   - Open the app
   - Tap "Import from file"
   - Navigate to and select your key file
   - Follow the prompts to import the key

#### Setting Up Password Store App

1. Open Password Store app
2. In the setup wizard:
   - Select "Clone from server"
   - Enter your Git repository URL
   - Choose SSH or HTTPS authentication
   - For SSH, you'll need to generate or import an SSH key
   - Select your imported GPG key

3. The app will clone your password repository and you can now access your passwords

#### Using Password Store on Android

- Browse your password hierarchy
- Search for passwords
- Copy passwords to clipboard
- Autofill passwords in other apps (requires setup)
- Synchronize with your Git repository

### iOS Integration with Pass for iOS

Pass for iOS provides similar functionality on Apple mobile devices.

#### Installation

1. Install [Pass for iOS](https://apps.apple.com/us/app/pass-password-store/id1205820573) from the App Store

#### Importing Your GPG Key to iOS

1. Export your GPG key to your computer:
   ```bash
   gpg --export-secret-keys --armor YOUR_GPG_KEY_ID > mobile-key.asc
   ```

2. Transfer this file to your iOS device (via AirDrop, iCloud Drive, or encrypted email)

3. In Pass for iOS:
   - Open the app
   - Go to Settings
   - Tap "Import PGP Key"
   - Select your key file
   - Enter your key passphrase

#### Setting Up Pass for iOS

1. In the app:
   - Go to Settings
   - Tap "Password Repository"
   - Select "Clone from Git Repository"
   - Enter your repository URL
   - Choose authentication method (SSH or HTTPS)
   - For SSH, you'll need to add your SSH key

2. The app will clone your password repository

#### Using Pass on iOS

- Browse your password hierarchy
- Search for passwords
- Copy passwords to clipboard
- Use the iOS share extension
- Synchronize with your Git repository

## Server Integration

Adding Pass to a server allows you to access your passwords in headless environments.

### Installation on a Server

SSH into your server and install the required packages:

```bash
# For Debian/Ubuntu
sudo apt update
sudo apt install -y pass gnupg2 git

# For CentOS/RHEL
sudo yum install -y pass gnupg2 git

# For Alpine Linux
apk add pass gnupg git
```

### Importing Your GPG Keys

Transfer your keys securely to the server:

```bash
# On your local machine, create an encrypted archive
tar -czf server-keys.tar.gz public-key.asc private-key.asc trust.txt
gpg -c server-keys.tar.gz

# Transfer the encrypted file to the server
scp server-keys.tar.gz.gpg user@server:~

# On the server, decrypt and import
gpg -d server-keys.tar.gz.gpg > server-keys.tar.gz
tar -xzf server-keys.tar.gz
gpg --import public-key.asc
gpg --import private-key.asc
gpg --import-ownertrust trust.txt

# Clean up
shred -u server-keys.tar.gz public-key.asc private-key.asc trust.txt
```

### Setting Up Pass on the Server

Initialize Pass and clone your repository:

```bash
# Initialize Pass
pass init "YOUR_GPG_KEY_ID"

# Remove the empty store
rm -rf ~/.password-store

# Clone your repository
git clone git@github.com:yourusername/password-store.git ~/.password-store
```

### Server-Specific Considerations

#### Non-Interactive GPG Usage

For scripts that need to access passwords without prompting for the passphrase:

1. Create a GPG batch file:
   ```bash
   echo "pinentry-mode loopback" > ~/.gnupg/gpg.conf
   echo "allow-loopback-pinentry" >> ~/.gnupg/gpg-agent.conf
   ```

2. For automated scripts, you can use a passphrase file (use with caution):
   ```bash
   # Create a secure passphrase file
   echo "YOUR_PASSPHRASE" > ~/.gpg-passphrase
   chmod 600 ~/.gpg-passphrase

   # Use it in a script
   GPG_OPTS="--batch --passphrase-file ~/.gpg-passphrase"
   pass show -c Website/example
   ```

#### Security Considerations for Servers

Servers are often more exposed than personal devices:

1. Use a separate GPG key for server use if possible
2. Consider using a read-only Git clone on servers
3. Implement strict file permissions
4. Use a separate password store with only the passwords needed on the server

## Continuous Integration Considerations

For CI/CD environments, you may need access to passwords for deployment or testing.

### Setting Up Pass in CI/CD Pipelines

#### GitHub Actions Example

```yaml
name: Deploy with Pass

on:
  push:
    branches: [ main ]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2

      - name: Set up GPG
        run: |
          echo "${{ secrets.GPG_PRIVATE_KEY }}" | base64 -d > private.key
          gpg --batch --import private.key
          echo "${{ secrets.GPG_PASSPHRASE }}" > passphrase.txt
          chmod 600 passphrase.txt

      - name: Set up Pass
        run: |
          sudo apt-get install -y pass
          git clone ${{ secrets.PASSWORD_STORE_REPO }} ~/.password-store

      - name: Use password in deployment
        run: |
          DB_PASSWORD=$(gpg --batch --passphrase-file passphrase.txt -d ~/.password-store/CI/database.gpg)
          echo "Using password to deploy (not showing password)"
          # Use $DB_PASSWORD in your deployment script
```

Store the GPG private key and passphrase as GitHub secrets.

#### GitLab CI Example

```yaml
deploy:
  stage: deploy
  script:
    - echo "$GPG_PRIVATE_KEY" | base64 -d > private.key
    - gpg --batch --import private.key
    - echo "$GPG_PASSPHRASE" > passphrase.txt
    - chmod 600 passphrase.txt
    - apt-get update && apt-get install -y pass
    - git clone $PASSWORD_STORE_REPO ~/.password-store
    - DB_PASSWORD=$(gpg --batch --passphrase-file passphrase.txt -d ~/.password-store/CI/database.gpg)
    - echo "Using password to deploy (not showing password)"
    # Use $DB_PASSWORD in your deployment script
  only:
    - main
```

Store the GPG private key and passphrase as GitLab CI/CD variables.

### Security Best Practices for CI/CD

1. Use a dedicated GPG key for CI/CD with limited access
2. Store secrets securely in your CI/CD platform
3. Consider using a separate password store with only CI/CD-related passwords
4. Regularly rotate CI/CD keys and passwords
5. Audit access to CI/CD secrets

## Synchronization Across Multiple Platforms

With Pass set up on multiple platforms, maintaining synchronization becomes more important.

### Synchronization Strategy

1. **Pull Before Use**: Always pull the latest changes before accessing passwords:
   ```bash
   pass git pull
   ```

2. **Push After Changes**: Always push changes after modifying passwords:
   ```bash
   pass git push
   ```

3. **Consider Automation**: Set up Git hooks or scheduled tasks to automate synchronization

4. **Handle Conflicts Carefully**: When conflicts occur, resolve them thoughtfully to avoid data loss

### Platform-Specific Sync Considerations

#### Mobile Devices

Mobile apps typically have sync buttons or settings:
- In Password Store for Android, pull from the menu
- In Pass for iOS, use the sync button

#### Servers

Consider setting up a cron job for regular synchronization:

```bash
# Add to crontab
*/30 * * * * cd ~/.password-store && git pull
```

This pulls changes every 30 minutes.

## Conclusion

By extending Pass to additional platforms, you create a truly cross-platform password management system that works wherever you need it. Whether you're on a desktop, mobile device, or server, you can securely access your passwords using the same familiar tools and workflows.

Key points to remember:
- The core process is similar across platforms: install Pass, import GPG keys, clone repository
- Mobile integration requires specialized apps but follows the same principles
- Server and CI/CD integration requires additional security considerations
- Consistent synchronization is essential across all platforms

With your password management system now extended to all your devices, you have a comprehensive solution that balances security, convenience, and flexibility.

In the next guide, we'll cover long-term maintenance and administration of your password management system.

## Navigation

- [README](README.md) - Wiki Home
- Previous: [Troubleshooting Guide](13_Troubleshooting_Guide.md)
- Next: [Maintenance and Administration](15_Maintenance_and_Administration.md)
- Related:
  - [GUI Clients and Browser Integration](11_GUI_Clients_and_Browser_Integration.md) - For desktop integration
  - [Synchronization Workflow Between Systems](09_Synchronization_Workflow_Between_Systems.md) - For sync details
  - [Security Best Practices](10_Security_Best_Practices.md) - For security considerations
