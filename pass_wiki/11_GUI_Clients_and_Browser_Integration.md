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
brew install browserpass
```

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
