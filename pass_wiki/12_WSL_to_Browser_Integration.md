# WSL to Browser Integration

## Introduction

Using Pass Password Manager in Windows Subsystem for Linux (WSL) presents unique challenges when it comes to browser integration. This guide explains how to bridge the gap between your Linux-based password store and Windows-based browsers, providing a seamless password management experience across the WSL/Windows boundary.

## Understanding the WSL-Windows Boundary

WSL and Windows operate as separate environments with different filesystems and process spaces. To integrate Pass with browsers running in Windows, we need to establish communication channels that cross this boundary.

### Key Concepts

1. **Native Messaging Hosts**: Browser extensions communicate with external applications through a protocol called Native Messaging. This requires a "host" application registered with the browser.

2. **WSL-Windows Interoperability**: WSL can access Windows files and Windows can access WSL files, but processes in one environment cannot directly communicate with processes in the other.

3. **Filesystem Access**: Windows can access WSL files through the `\\wsl$\` path, and WSL can access Windows drives through `/mnt/c/`.

## Integration Architecture

The integration between Pass in WSL and browsers in Windows follows this architecture:

```
┌─────────────────────┐      ┌─────────────────────┐
│     Windows         │      │        WSL          │
│  ┌───────────────┐  │      │  ┌───────────────┐  │
│  │    Browser    │  │      │  │     Pass      │  │
│  └───────┬───────┘  │      │  └───────┬───────┘  │
│          │          │      │          │          │
│  ┌───────┴───────┐  │      │  ┌───────┴───────┐  │
│  │   Extension   │  │      │  │  Password     │  │
│  └───────┬───────┘  │      │  │  Store        │  │
│          │          │      │  └───────────────┘  │
│  ┌───────┴───────┐  │      │          ▲          │
│  │ Native        │  │      │          │          │
│  │ Messaging Host├──┼──────┼──────────┘          │
│  └───────────────┘  │      │                     │
└─────────────────────┘      └─────────────────────┘
```

The key component is a "bridge" that allows the browser extension to communicate with Pass in WSL.

## Setting Up Browser Integration

### Prerequisites

1. Pass properly installed and configured in WSL
2. A Windows browser (Chrome, Firefox, or Edge)
3. Basic familiarity with both Windows and Linux command lines

### Firefox Integration with passff

Firefox offers the simplest integration with Pass in WSL through the passff extension.

#### Step 1: Install the Host Application in WSL

```bash
# In WSL
sudo apt install -y firefox-passff
```

#### Step 2: Install the Firefox Extension

Visit [passff on Firefox Add-ons](https://addons.mozilla.org/en-US/firefox/addon/passff/) and install the extension.

#### Step 3: Create a Windows-WSL Bridge

Create a batch file in Windows to bridge between Firefox and WSL:

1. Create a file named `passff-host.bat` in a location like `C:\Users\YourUsername\bin\` with:

```batch
@echo off
wsl /usr/lib/mozilla/native-messaging-hosts/passff.py
```

2. Create the native messaging host manifest for Firefox:

```bash
# In WSL
mkdir -p "/mnt/c/Users/YourUsername/AppData/Roaming/Mozilla/NativeMessagingHosts/"
```

3. Create a file at `/mnt/c/Users/YourUsername/AppData/Roaming/Mozilla/NativeMessagingHosts/passff.json` with:

```json
{
  "name": "passff",
  "description": "Host for communicating with pass",
  "path": "C:\\Users\\YourUsername\\bin\\passff-host.bat",
  "type": "stdio",
  "allowed_extensions": [
    "passff@invicem.pro"
  ]
}
```

Replace `YourUsername` with your actual Windows username.

### Chrome/Edge Integration with browserpass

Chrome and Edge require a more complex setup with browserpass.

#### Step 1: Install browserpass-native in WSL

```bash
# Install dependencies
sudo apt update
sudo apt install -y golang-go make

# Clone and build browserpass-native
git clone https://github.com/browserpass/browserpass-native.git
cd browserpass-native
make configure
make
sudo make install
```

#### Step 2: Install the Browser Extension

Visit the [Chrome Web Store](https://chrome.google.com/webstore/detail/browserpass-ce/naepdomgkenhinolocfifgehidddafch) to install the browserpass extension.

#### Step 3: Create a Windows-WSL Bridge

1. Create a batch file in Windows to bridge between Chrome and WSL:

Create a file named `browserpass-wsl.bat` in a location like `C:\Users\YourUsername\bin\` with:

```batch
@echo off
wsl /usr/bin/browserpass
```

2. Create the native messaging host manifest for Chrome:

```bash
# In WSL
mkdir -p "/mnt/c/Users/YourUsername/AppData/Local/Google/Chrome/User Data/NativeMessagingHosts/"
```

3. Create a file at `/mnt/c/Users/YourUsername/AppData/Local/Google/Chrome/User Data/NativeMessagingHosts/com.github.browserpass.native.json` with:

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

For Microsoft Edge, the path would be:
```
/mnt/c/Users/YourUsername/AppData/Local/Microsoft/Edge/User Data/NativeMessagingHosts/
```

## Troubleshooting Common Issues

### 1. "Cannot connect to native host" Error

This usually means the browser cannot find or execute the native messaging host.

**Solution:**
- Verify the batch file path in the manifest is correct
- Ensure the batch file has execute permissions
- Check that the path to the WSL executable in the batch file is correct

### 2. "Loading available logins" Indefinitely

This indicates the extension can connect to the native host, but the host cannot access your password store.

**Solution:**
- Create a configuration file in WSL:

```bash
mkdir -p ~/.config/browserpass
nano ~/.config/browserpass/config.json
```

Add:
```json
{
  "store": {
    "path": "/home/yourusername/.password-store",
    "mount": true
  }
}
```

### 3. GPG Passphrase Issues

If you encounter errors related to GPG being unable to prompt for your passphrase:

**Solution:**
- Install a GUI pinentry program in WSL:
```bash
sudo apt install -y pinentry-gtk2
```

- Configure GPG to use it:
```bash
echo "pinentry-program /usr/bin/pinentry-gtk2" >> ~/.gnupg/gpg-agent.conf
gpgconf --kill gpg-agent
```

## Alternative Approaches

### 1. X11 Forwarding with Native Linux Browsers

Instead of bridging WSL and Windows browsers, you can run Linux browsers directly in WSL with X11 forwarding:

1. Install an X server on Windows (like VcXsrv)
2. Configure WSL for X11 forwarding
3. Install and run Firefox in WSL
4. Install the passff extension directly in the WSL Firefox

### 2. Clipboard Integration

A simpler approach is to use Pass's clipboard functionality:

```bash
# Copy a password to clipboard
pass -c Category/service-name
```

This copies the password to your clipboard, which is shared between WSL and Windows.

## Navigation

- [README](README.md) - Wiki Home
- Previous: [GUI Clients and Browser Integration](11_GUI_Clients_and_Browser_Integration.md)
- Next: [Advanced Features and Extensions](13_Advanced_Features_and_Extensions.md)
- Related:
  - [Setting Up Pass on WSL Ubuntu](08_Setting_Up_Pass_on_WSL_Ubuntu.md)
  - [Security Best Practices](10_Security_Best_Practices.md)
  - [Troubleshooting Guide](14_Troubleshooting_Guide.md)
