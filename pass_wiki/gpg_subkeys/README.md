# Advanced GPG Key Management with Subkeys

This section of the wiki provides detailed guides on using GPG subkeys with the `pass` password manager for enhanced security. Using subkeys allows you to keep your master GPG key secure (even offline) while still using `pass` on multiple devices.

## Wiki Articles in This Section

1. [Understanding GPG Subkeys for Pass](16_Understanding_GPG_Subkeys_for_Pass.md) - Introduction to using GPG subkeys for enhanced security
2. [Creating and Managing GPG Subkeys](17_Creating_and_Managing_GPG_Subkeys.md) - Setting up and maintaining GPG subkeys
3. [Exporting GPG Subkeys for Other Devices](18_Exporting_GPG_Subkeys_for_Other_Devices.md) - Securely transferring subkeys to secondary devices
4. [Setting Up Pass with GPG Subkeys](19_Setting_Up_Pass_with_GPG_Subkeys.md) - Configuring pass to work with subkeys
5. [Subkey Rotation and Maintenance](20_Subkey_Rotation_and_Maintenance.md) - Managing subkey lifecycle and rotation
6. [Troubleshooting GPG Subkeys with Pass](21_Troubleshooting_GPG_Subkeys_with_Pass.md) - Solving common issues with subkeys

## Why Use Subkeys?

Using GPG subkeys with `pass` provides several important security benefits:

- **Keep your master key secure**: Your master key can be stored offline, reducing the risk of compromise
- **Limit damage from device compromise**: If a device with subkeys is compromised, you can revoke just those subkeys
- **Flexible access control**: Create different subkeys for different devices
- **Regular key rotation**: Set subkeys to expire, forcing regular rotation for better security

## How to Use This Section

1. Start with [Understanding GPG Subkeys for Pass](16_Understanding_GPG_Subkeys_for_Pass.md) to learn the concepts
2. Follow the guides in order for a complete setup
3. Refer to [Troubleshooting GPG Subkeys with Pass](21_Troubleshooting_GPG_Subkeys_with_Pass.md) if you encounter issues

## Navigation

- [Main Wiki Home](../README.md) - Return to the main wiki
