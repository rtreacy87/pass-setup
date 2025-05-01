# Understanding GPG Subkeys for Pass

## Introduction

When using the `pass` password manager across multiple devices, you need to access your encrypted passwords on each device. The standard approach is to copy your entire GPG key (including your master/primary key) to each device. However, this approach has security risks - if any of your devices is compromised, your master key could be stolen.

This guide introduces a more secure alternative: **GPG subkeys**. Using subkeys allows you to keep your master key secure (even offline) while still using `pass` on multiple devices.

## What Are GPG Subkeys?

Think of your GPG key setup like a master key ring:

- **Master/Primary Key**: This is your identity in the GPG system. It's like the main key to your house that can do everything.
- **Subkeys**: These are additional keys that are connected to your master key but have limited capabilities. They're like specialized keys that can only open certain doors.

The important security benefit is that you can keep your master key in a safe place (like a secure offline computer or USB drive) while only putting the subkeys on your everyday devices.

## Why Use Subkeys with Pass?

Using subkeys with `pass` offers several important benefits:

1. **Better Security**: If a device with only subkeys is compromised, attackers can't use those subkeys to create new subkeys or change your identity. You can simply revoke the compromised subkeys without affecting your master key.

2. **Damage Limitation**: If a subkey is compromised, you only need to revoke that specific subkey, not your entire GPG identity.

3. **Flexible Access Control**: You can create different subkeys for different devices, making it easier to manage access.

4. **Expiration Management**: You can set subkeys to expire, forcing regular rotation for better security.

## Types of GPG Subkeys

GPG supports three types of subkeys, each with a specific purpose:

1. **Encryption Subkey (E)**: Used to encrypt and decrypt data. This is the subkey that `pass` uses to encrypt and decrypt your passwords.

2. **Signing Subkey (S)**: Used to create digital signatures, which verify that content hasn't been tampered with. This is useful for signing Git commits when using `pass` with Git.

3. **Authentication Subkey (A)**: Used for authentication purposes, such as SSH authentication. This isn't directly used by `pass` but can be useful for related tasks.

## Which Subkeys Do You Need for Pass?

For using `pass` effectively across multiple devices:

- **Required**: Encryption subkey (E) - This is essential for encrypting and decrypting your passwords.
- **Recommended**: Signing subkey (S) - This is useful if you use Git with `pass` to track changes to your password store.
- **Optional**: Authentication subkey (A) - This is not directly used by `pass` but can be useful for SSH access to Git repositories.

## Subkeys vs. Full Key Export: A Comparison

Let's compare the two approaches for using `pass` on multiple devices:

### Full Key Export (Current Wiki Approach)

**Pros:**
- Simpler to set up
- Full functionality on all devices
- No need to access master key for future operations

**Cons:**
- Higher security risk if a device is compromised
- All-or-nothing approach to key management
- More difficult to recover from a compromise

### Subkey Approach (This Wiki Series)

**Pros:**
- Better security through isolation of the master key
- Easier to recover from device compromise
- More granular control over device access
- Supports key rotation and expiration

**Cons:**
- More complex initial setup
- Requires access to master key for certain operations (like creating new subkeys)
- Requires more planning and maintenance

## When to Use Subkeys vs. Full Key Export

Consider using subkeys if:

- You have high-security requirements
- You use `pass` on multiple devices, especially mobile or less secure devices
- You want to limit damage if a device is lost or stolen
- You're comfortable with a slightly more complex setup process

Stick with full key export if:

- You only use `pass` on one or two highly secure devices
- You prefer simplicity over maximum security
- You're new to GPG and want to start with the basics

## Visual Explanation: How Subkeys Work with Pass

```
SECURE LOCATION (e.g., offline computer, encrypted USB)
┌─────────────────────────────────────────────────┐
│                                                 │
│  Master/Primary Key                             │
│  - Creates your GPG identity                    │
│  - Can create/revoke subkeys                    │
│  - Should be kept highly secure                 │
│                                                 │
└───────────────┬─────────────────┬───────────────┘
                │                 │
                ▼                 ▼
┌─────────────────────┐ ┌─────────────────────┐
│  Encryption Subkey  │ │  Signing Subkey     │
│  - Encrypts/decrypts│ │  - Signs data       │
│  - Used by pass     │ │  - Used for Git     │
└─────────────────────┘ └─────────────────────┘
                │                 │
                ▼                 ▼
┌─────────────────────────────────────────────────┐
│                                                 │
│  EVERYDAY DEVICES                               │
│  - Only contain subkeys                         │
│  - Can use pass normally                        │
│  - Limited damage if compromised                │
│                                                 │
└─────────────────────────────────────────────────┘
```

## Conclusion

Using GPG subkeys with `pass` provides a significant security improvement over copying your entire GPG key to multiple devices. While it requires a bit more setup initially, the security benefits are substantial, especially if you use `pass` on multiple or potentially less secure devices.

In the next guide, we'll walk through the process of creating and managing GPG subkeys for use with `pass`.

## Navigation

- [README](../README.md) - Wiki Home
- Next: [Creating and Managing GPG Subkeys](17_Creating_and_Managing_GPG_Subkeys.md)
- Related: [Exporting GPG Keys for Cross-Platform Use](../05_Exporting_GPG_Keys_for_Cross_Platform_Use.md)
