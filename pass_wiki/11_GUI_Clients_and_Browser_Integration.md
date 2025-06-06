# GUI Clients and Browser Integration

## Introduction

While Pass is primarily a command-line tool, there are several graphical user interfaces (GUIs) and browser extensions available that can make it more convenient to use. This guide will cover the available options for macOS, Windows/WSL, and browsers, focusing on command-line setup whenever possible.

## macOS GUI Options

### QtPass

QtPass is a cross-platform GUI for Pass that works well on macOS.

#### Installation

Install QtPass using Homebrew:

```bash
brew install --cask qtpass
```

This command downloads and installs the QtPass application.

#### Configuration

When you first launch QtPass (from your Applications folder), you'll need to configure it:

1. Select "Use pass/gpg" as the backend
2. Set the path to your password store (default: `~/.password-store`)
3. Set the path to the `pass` executable (default: `/usr/local/bin/pass` or `/opt/homebrew/bin/pass`)
4. Set the path to the `gpg` executable (default: `/usr/local/bin/gpg` or `/opt/homebrew/bin/gpg`)

You can find the correct paths by running these commands in Terminal:

```bash
which pass
which gpg
```

#### Usage

QtPass provides a graphical interface for:
- Viewing your password structure
- Generating new passwords
- Editing existing passwords
- Copying passwords to clipboard
- Searching for passwords

### Pass for macOS

Pass for macOS is a native macOS application for Pass.

#### Installation

Install it using Homebrew:

```bash
brew install --cask pass
```

#### Configuration

When you first launch Pass (from your Applications folder), it will automatically detect your password store and GPG configuration.

#### Usage

Pass for macOS integrates well with macOS and provides:
- A menu bar icon for quick access
- Native macOS notifications
- Spotlight integration for searching passwords
- Touch ID support (on compatible Macs)

## Windows/WSL GUI Options

### QtPass for Windows

QtPass also works on Windows and can be configured to work with your WSL Ubuntu Pass installation.

#### Installation

1. Download the Windows installer from the [QtPass website](https://qtpass.org/)
2. Run the installer and follow the prompts

#### Configuration for WSL Integration

Configuring QtPass to work with WSL requires some additional steps:

1. In QtPass, go to Options
2. Select "Use pass/gpg" as the backend
3. Set the path to your password store: `\\wsl$\Ubuntu\home\yourusername\.password-store`
4. For the `pass` executable, create a batch file wrapper:

   Create a file named `pass.bat` in a location like `C:\Users\YourUsername\bin\` with the following content:
   ```batch
   @echo off
   wsl pass %*
   ```

5. For the `gpg` executable, create a similar wrapper:

   Create a file named `gpg.bat` in the same location with:
   ```batch
   @echo off
   wsl gpg %*
   ```

6. In QtPass, set the paths to these batch files

This configuration allows QtPass to call into WSL to execute the actual Pass and GPG commands.

### WSL Ubuntu with X Server

For a more integrated experience, you can run Linux GUI applications directly from WSL using an X server.

#### Installing an X Server on Windows

1. Download and install VcXsrv from [SourceForge](https://sourceforge.net/projects/vcxsrv/)
2. Launch XLaunch from the Start menu
3. Choose display settings (defaults are fine)
4. Select "Disable access control" and finish

#### Configuring WSL for X11 Forwarding

Add these lines to your `~/.bashrc` in WSL Ubuntu:

```bash
export DISPLAY=$(grep -m 1 nameserver /etc/resolv.conf | awk '{print $2}'):0
export LIBGL_ALWAYS_INDIRECT=1
```

Then apply the changes:

```bash
source ~/.bashrc
```

#### Installing QtPass in WSL Ubuntu

```bash
sudo apt update
sudo apt install -y qtpass
```

#### Running QtPass from WSL

With VcXsrv running on Windows, you can now launch QtPass from WSL:

```bash
qtpass &
```

The QtPass window should appear on your Windows desktop, running from WSL but displaying through the X server.

## Browser Integration

Browser integration allows you to automatically fill passwords on websites.

### Firefox Integration with passff

#### Installing the Extension

1. Install the host application:

   On macOS:
   ```bash
   brew install passff-host
   ```

   On WSL Ubuntu:
   ```bash
   sudo apt install -y firefox-passff
   ```

2. Install the Firefox extension by visiting [passff on Firefox Add-ons](https://addons.mozilla.org/en-US/firefox/addon/passff/)

#### Configuration

The extension should automatically detect your Pass installation. If not, you can configure it in the extension settings:

1. Click the passff icon in Firefox
2. Click the gear icon to open settings
3. Set the path to your password store if needed

#### Usage

With passff installed:
1. Navigate to a website's login page
2. Click the passff icon in the toolbar
3. Select the appropriate password entry
4. The extension will fill in your username and password

### Chrome/Chromium Integration with browserpass

#### Installing the Host Application

On macOS:
```bash
# Add the required tap first
brew tap amar1729/formulae
brew install browserpass

# Configure for your browser (the installation will print similar instructions)
# For Firefox:
PREFIX='/opt/homebrew/opt/browserpass' make hosts-firefox-user -f '/opt/homebrew/opt/browserpass/lib/browserpass/Makefile'
# For Chrome:
# PREFIX='/opt/homebrew/opt/browserpass' make hosts-chrome-user -f '/opt/homebrew/opt/browserpass/lib/browserpass/Makefile'
```

Note: The `PREFIX` path may vary depending on your Homebrew installation. If you're using an Intel Mac, the path might be `/usr/local/opt/browserpass` instead.

On WSL Ubuntu:
```bash
# Clone the repository
git clone https://github.com/browserpass/browserpass-native.git
cd browserpass-native

# Build and install
make configure
make
sudo make install
```

#### Troubleshooting Browserpass Installation

##### macOS Installation Issues

If you encounter an error like `Warning: No available formula with the name "browserpass"` when trying to install browserpass on macOS, you need to use a user-contributed tap:

1. **Add the required tap and install browserpass**:
   ```bash
   brew tap amar1729/formulae
   brew install browserpass
   ```

2. **Configure browser integration**:
   After installation, you need to configure browserpass for your browser. The installation will print instructions, but typically you'll run:

   For Firefox:
   ```bash
   PREFIX='/opt/homebrew/opt/browserpass' make hosts-firefox-user -f '/opt/homebrew/opt/browserpass/lib/browserpass/Makefile'
   ```

   For Chrome/Chromium:
   ```bash
   PREFIX='/opt/homebrew/opt/browserpass' make hosts-chrome-user -f '/opt/homebrew/opt/browserpass/lib/browserpass/Makefile'
   ```

   Note: Run these commands in your terminal. The `PREFIX` path may vary depending on your Homebrew installation. If you're using an Intel Mac, the path might be `/usr/local/opt/browserpass` instead.

3. **Verify installation**:
   ```bash
   ls -la ~/.mozilla/native-messaging-hosts/
   # Should show com.github.browserpass.native.json for Firefox

   # Or for Chrome:
   ls -la ~/Library/Application\ Support/Google/Chrome/NativeMessagingHosts/
   # Should show com.github.browserpass.native.json
   ```

4. **Install the browser extension**:
   - [Firefox Add-ons](https://addons.mozilla.org/en-US/firefox/addon/browserpass-ce/)
   - [Chrome Web Store](https://chrome.google.com/webstore/detail/browserpass-ce/naepdomgkenhinolocfifgehidddafch)

##### WSL Installation Issues

If you encounter errors like `env: 'go': No such file or directory` when trying to build browserpass-native, follow these steps:

1. **Install Go on WSL Ubuntu**:
   ```bash
   sudo apt update
   sudo apt install -y golang-go
   ```

2. **Verify Go installation**:
   ```bash
   go version
   ```
   This should display the installed Go version.

3. **Retry building browserpass-native**:
   ```bash
   cd browserpass-native
   make configure
   make
   sudo make install
   ```

4. **If you encounter permission issues**, ensure proper permissions:
   ```bash
   chmod -R 755 ~/.config/browserpass
   ```

5. **Verify installation**:
   ```bash
   which browserpass
   # Should output: /usr/bin/browserpass
   ```

After successful installation, continue with the browser extension setup as described above.

##### GPG Binary Detection Issues

If you encounter an error like `Unable to fetch and parse login fields: Error: {"status":"error","code":22,"version":3001000,"params":{"action":"fetch","error":"Unable to detect the location of the gpg binary to use","message":"Unable to detect the location of the gpg binary"}}`, browserpass is having trouble finding your GPG installation. Here's how to fix it:

1. **Create or edit the browserpass configuration file**:

   For macOS:
   ```bash
   mkdir -p ~/.config/browserpass
   nano ~/.config/browserpass/config.json
   ```

2. **Add the full path to your GPG binary**:
   ```json
   {
     "gpgPath": "/opt/homebrew/bin/gpg"
   }
   ```

   Note: The path may be different on your system. To find the correct path, run:
   ```bash
   which gpg
   ```

   On Intel Macs, it might be `/usr/local/bin/gpg` instead.

3. **Save the file** (in nano: Ctrl+O, Enter, then Ctrl+X)

4. **Restart your browser** to apply the changes

5. **If the issue persists**, you can also try creating a `.browserpass.json` file in the root of your password store:
   ```bash
   nano ~/.password-store/.browserpass.json
   ```

   With the same content:
   ```json
   {
     "gpgPath": "/opt/homebrew/bin/gpg"
   }
   ```

This configuration tells browserpass exactly where to find your GPG binary, which should resolve the detection issue.

##### GPG Passphrase Input Issues

If you encounter an error like `Unable to fetch and parse login fields: Error: {"status":"error","code":24,"version":3001000,"params":{"action":"fetch","error":"Error: exit status 2, Stderr: gpg: Sorry, we are in batchmode - can't get input\n"...`, GPG is unable to prompt for your passphrase. This is typically caused by issues with the pinentry program. Here's how to fix it:

1. **Install a GUI pinentry program** if you don't have one:

   On macOS:
   ```bash
   brew install pinentry-mac
   ```

2. **Configure GPG to use the GUI pinentry**:
   ```bash
   echo "pinentry-program /opt/homebrew/bin/pinentry-mac" >> ~/.gnupg/gpg-agent.conf
   ```

   Note: The path may be different on your system. To find the correct path, run:
   ```bash
   which pinentry-mac
   ```

   On Intel Macs, it might be `/usr/local/bin/pinentry-mac` instead.

3. **Restart the GPG agent**:
   ```bash
   gpgconf --kill gpg-agent
   ```

4. **Test that GPG can prompt for your passphrase**:
   ```bash
   echo "test" | gpg --encrypt --recipient YOUR_KEY_ID | gpg --decrypt
   ```

   Replace `YOUR_KEY_ID` with your actual GPG key ID. This should prompt for your passphrase with a GUI dialog.

5. **Restart your browser** and try browserpass again.

6. **If issues persist**, you can try adding these lines to your `~/.gnupg/gpg.conf` file:
   ```bash
   use-agent
   batch
   no-tty
   ```

7. **For more advanced troubleshooting**, you can enable browserpass debug mode by adding to your `~/.config/browserpass/config.json`:
   ```json
   {
     "gpgPath": "/opt/homebrew/bin/gpg",
     "debug": true
   }
   ```

   Then check the browser console for more detailed error messages.

#### Installing the Browser Extension

1. Visit the [Chrome Web Store](https://chrome.google.com/webstore/detail/browserpass/naepdomgkenhinolocfifgehidddafch) to install the extension
2. After installation, the extension will guide you through connecting to the host application

#### Configuration for WSL (Advanced)

For Chrome to work with WSL, you need to create a custom native messaging host manifest:

1. Create a directory for the manifest:
   ```bash
   mkdir -p ~/.config/browserpass
   ```

2. Create a configuration file:
   ```bash
   nano ~/.config/browserpass/config.json
   ```

3. Add the following content:
   ```json
   {
     "store": {
       "path": "/home/yourusername/.password-store",
       "mount": true
     }
   }
   ```

4. Create a Windows batch script to bridge between Chrome and WSL:

   Create a file named `browserpass-wsl.bat` in a location like `C:\Users\YourUsername\bin\` with:
   ```batch
   @echo off
   wsl ~/.local/bin/browserpass-native
   ```

5. Create a native messaging host manifest for Chrome:

   Create a file in `%LOCALAPPDATA%\Google\Chrome\User Data\NativeMessagingHosts\com.github.browserpass.native.json` with:
   ```json
   {
     "name": "com.github.browserpass.native",
     "description": "Browserpass native component for WSL",
     "path": "C:\\Users\\YourUsername\\bin\\browserpass-wsl.bat",
     "type": "stdio",
     "allowed_origins": [
       "chrome-extension://naepdomgkenhinolocfifgehidddafch/"
     ]
   }
   ```

This configuration is complex and may require troubleshooting.

#### Troubleshooting Browserpass WSL-to-Windows Chrome Communication

If your browserpass Chrome extension shows "Loading available logins" indefinitely when using WSL, the issue is likely with the communication bridge between WSL and Windows Chrome:

1. **Verify the WSL-Windows bridge is properly configured**:

   The most common issue is that Chrome in Windows cannot communicate with browserpass in WSL. Follow these steps to fix it:

   a. Create a Windows batch script to bridge between Chrome and WSL:

   Create a file named `browserpass-wsl.bat` in a location like `C:\Users\YourUsername\bin\` with:
   ```batch
   @echo off
   wsl ~/.local/bin/browserpass-native
   ```

   b. Create a native messaging host manifest for Chrome in Windows:

   Create a file in `%LOCALAPPDATA%\Google\Chrome\User Data\NativeMessagingHosts\com.github.browserpass.native.json` with:
   ```json
   {
     "name": "com.github.browserpass.native",
     "description": "Browserpass native component for WSL",
     "path": "C:\\Users\\YourUsername\\bin\\browserpass-wsl.bat",
     "type": "stdio",
     "allowed_origins": [
       "chrome-extension://naepdomgkenhinolocfifgehidddafch/"
     ]
   }
   ```

   Make sure to replace `YourUsername` with your actual Windows username and verify the path to the batch file is correct.

2. **Check the path to browserpass in WSL**:

   The batch file assumes browserpass is installed at `~/.local/bin/browserpass-native`. Verify the actual location:
   ```bash
   which browserpass
   ```

   If it's in a different location, update the batch file accordingly:
   ```batch
   @echo off
   wsl /usr/bin/browserpass
   ```

3. **Verify the Chrome extension ID**:

   The `allowed_origins` in the manifest must match your installed extension ID. To check:
   - Go to Chrome Extensions (chrome://extensions/)
   - Enable Developer mode (toggle in top-right)
   - Find the ID for Browserpass and ensure it matches the one in your manifest

4. **Test the batch file manually**:

   Open Command Prompt and run:
   ```
   C:\Users\YourUsername\bin\browserpass-wsl.bat
   ```

   You should see some output or an error message that can help diagnose the issue.

5. **Check Windows permissions**:

   Ensure the batch file and manifest have appropriate permissions:
   ```
   Right-click browserpass-wsl.bat → Properties → Security
   ```

   Make sure your user account has read and execute permissions.

6. **Restart Chrome**:

   After making these changes, completely close Chrome (check Task Manager to ensure all Chrome processes are terminated) and restart it.

#### Usage

With browserpass installed:
1. Navigate to a website's login page
2. Click the browserpass icon in the toolbar
3. Select the appropriate password entry
4. The extension will fill in your username and password

## Mobile Applications

While this guide focuses on macOS and Windows/WSL, it's worth mentioning mobile options for completeness.

### Android: Password Store

Password Store is an Android app that works with Pass.

To set it up:
1. Install [Password Store](https://play.google.com/store/apps/details?id=dev.msfjarvis.aps) from Google Play
2. Configure it to use your Git repository
3. Import your GPG key

### iOS: Pass for iOS

Pass for iOS is an iOS app that works with Pass.

To set it up:
1. Install [Pass for iOS](https://apps.apple.com/us/app/pass-password-store/id1205820573) from the App Store
2. Configure it to use your Git repository
3. Import your GPG key

Setting up mobile applications requires additional steps beyond the scope of this guide.

## Command-Line Alternatives to GUI Integration

If you prefer to stick with command-line tools, here are some alternatives to GUI integration.

### Clipboard Integration

You can use Pass's built-in clipboard functionality:

```bash
# Copy a password to clipboard
pass -c Category/service-name
```

This copies the password to your clipboard for 45 seconds.

### Custom Scripts for Browser Integration

You can create custom scripts to fill passwords in browsers:

#### Example: Simple Autotype Script for macOS

Create a file named `pass-autotype.sh`:

```bash
#!/bin/bash
# Simple script to autotype username and password

# Get the password entry
ENTRY=$(pass | grep -v "Password Store" | sed 's/└── \|├── \|    //' | dmenu -i -p "Pass Entry:")

if [ -n "$ENTRY" ]; then
    # Get username and password
    PASSWORD=$(pass "$ENTRY" | head -n 1)
    USERNAME=$(pass "$ENTRY" | grep -i "username:" | cut -d ' ' -f 2-)

    # Type username, tab, password, enter
    osascript -e "tell application \"System Events\" to keystroke \"$USERNAME\""
    osascript -e "tell application \"System Events\" to keystroke tab"
    osascript -e "tell application \"System Events\" to keystroke \"$PASSWORD\""
    osascript -e "tell application \"System Events\" to keystroke return"
fi
```

Make it executable:

```bash
chmod +x pass-autotype.sh
```

You can bind this script to a keyboard shortcut using macOS's Automator or a tool like Karabiner-Elements.

## Conclusion

While Pass is primarily a command-line tool, these GUI clients and browser extensions can make it more convenient to use in your daily workflow. Choose the options that best fit your needs and preferences, keeping in mind that each additional integration point potentially increases the attack surface.

For maximum security, stick with the command-line interface and clipboard integration. For convenience, the browser extensions provide a good balance of security and usability.

In the next guide, we'll explore advanced features and extensions to enhance your password management system.

## Navigation

- [README](README.md) - Wiki Home
- Previous: [Security Best Practices](10_Security_Best_Practices.md)
- Next: [WSL to Browser Integration](12_WSL_to_Browser_Integration.md)
- Related:
  - [Setting Up Pass on WSL Ubuntu](08_Setting_Up_Pass_on_WSL_Ubuntu.md) - For WSL integration
  - [Extending to Additional Platforms](14_Extending_to_Additional_Platforms.md) - For mobile setup
