# Cross-Platform Password Management with Pass

This wiki series provides comprehensive documentation for setting up and maintaining a cross-platform password management system using `pass` (the standard Unix password manager). The guides focus on command-line operations and are designed to be accessible to users with limited command-line experience.

## Wiki Articles

### Initial Setup (macOS)

1. [Introduction to Pass Password Management](01_Introduction_to_Pass.md) - Overview of Pass and cross-platform password management
2. [Setting Up GPG Keys on macOS](02_Setting_Up_GPG_Keys_on_macOS.md) - Installing GPG and generating keys on macOS
3. [Initializing Pass on macOS](03_Initializing_Pass_on_macOS.md) - Setting up Pass and basic password operations
4. [Git Integration for Pass](04_Git_Integration_for_Pass.md) - Configuring Git for version control and synchronization
5. [Exporting GPG Keys for Cross-Platform Use](05_Exporting_GPG_Keys_for_Cross_Platform_Use.md) - Securely exporting keys for use on other systems

### Windows Setup

6. [Setting Up WSL Ubuntu for Pass](06_Setting_Up_WSL_Ubuntu_for_Pass.md) - Installing and configuring WSL Ubuntu on Windows
7. [Importing GPG Keys to WSL Ubuntu](07_Importing_GPG_Keys_to_WSL_Ubuntu.md) - Importing your GPG keys to WSL
8. [Setting Up Pass on WSL Ubuntu](08_Setting_Up_Pass_on_WSL_Ubuntu.md) - Configuring Pass on WSL Ubuntu
9. [Synchronization Workflow Between Systems](09_Synchronization_Workflow_Between_Systems.md) - Managing passwords across multiple systems

### Security and Best Practices

10. [Security Best Practices](10_Security_Best_Practices.md) - Ensuring the security of your password management system

### Advanced Usage

11. [GUI Clients and Browser Integration](11_GUI_Clients_and_Browser_Integration.md) - Optional graphical interfaces and browser extensions
12. [WSL to Browser Integration](12_WSL_to_Browser_Integration.md) - Detailed guide for connecting Pass in WSL to Windows browsers
13. [Advanced Features and Extensions](13_Advanced_Features_and_Extensions.md) - Two-factor authentication, Git hooks, and more

### Troubleshooting and Maintenance

14. [Troubleshooting Guide](14_Troubleshooting_Guide.md) - Solving common issues with Pass, GPG, and Git
15. [Extending to Additional Platforms](15_Extending_to_Additional_Platforms.md) - Adding more systems to your password ecosystem
16. [Maintenance and Administration](16_Maintenance_and_Administration.md) - Long-term management of your password system
17. [Real-Time Password Synchronization](17_Real_Time_Password_Synchronization.md) - Implementing immediate password updates across systems

### Advanced GPG Key Management

18. [Understanding GPG Subkeys for Pass](gpg_subkeys/18_Understanding_GPG_Subkeys_for_Pass.md) - Introduction to using GPG subkeys for enhanced security
19. [Creating and Managing GPG Subkeys](gpg_subkeys/19_Creating_and_Managing_GPG_Subkeys.md) - Setting up and maintaining GPG subkeys
20. [Exporting GPG Subkeys for Other Devices](gpg_subkeys/20_Exporting_GPG_Subkeys_for_Other_Devices.md) - Securely transferring subkeys to secondary devices
21. [Setting Up Pass with GPG Subkeys](gpg_subkeys/21_Setting_Up_Pass_with_GPG_Subkeys.md) - Configuring pass to work with subkeys
22. [Subkey Rotation and Maintenance](gpg_subkeys/22_Subkey_Rotation_and_Maintenance.md) - Managing subkey lifecycle and rotation
23. [Troubleshooting GPG Subkeys with Pass](gpg_subkeys/23_Troubleshooting_GPG_Subkeys_with_Pass.md) - Solving common issues with subkeys
24. [Understanding GPG Modes](gpg_subkeys/24_Understanding_GPG_Modes.md) - Explaining GPG batch mode and other interaction methods

## How to Use This Wiki

1. Start with the Introduction to understand the overall system
2. Follow the guides in order for initial setup (articles 1-9)
3. Refer to the Security Best Practices to ensure your system is secure
4. Explore advanced topics as needed (articles 11-17)
5. For enhanced security with GPG subkeys, follow the Advanced GPG Key Management series (articles 18-24)
6. Use the Troubleshooting Guide when you encounter issues

### Navigation Tips

- Each wiki article contains navigation links at the bottom to move between related articles
- You can always return to this README page as a central hub
- Articles with prerequisites contain links to those prerequisite articles
- The Troubleshooting Guide (14) is linked from relevant articles for quick reference
- For Git-related issues, refer directly to article 4 (Git Integration) and article 14 (Troubleshooting)

## Features of This Documentation

- **Command-Line Focused**: Minimal GUI usage, with clear explanations of command-line operations
- **Detailed Explanations**: Each command is explained in detail
- **Beginner-Friendly**: Designed for users with limited command-line experience
- **Cross-Platform**: Covers macOS and Windows (WSL Ubuntu), with information on extending to other platforms
- **Security-Oriented**: Emphasizes best practices for secure password management

## Additional Resources

- [Pass Official Website](https://www.passwordstore.org/)
- [GPG Documentation](https://gnupg.org/documentation/)
- [Git Documentation](https://git-scm.com/doc)

## Contributing

If you find errors or have suggestions for improvements, please contribute by submitting issues or pull requests to the repository.

## License

This documentation is provided under the MIT License. Feel free to use, modify, and distribute it as needed.
