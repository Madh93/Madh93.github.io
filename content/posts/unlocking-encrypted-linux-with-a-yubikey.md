---
title: Unlocking Encrypted Linux with a YubiKey
date: 2025-09-02
summary: A step-by-step guide to upgrading a LUKS-encrypted Linux system to use a YubiKey for FIDO2-based unlocking, moving beyond traditional passphrases.
categories: [Linux, Security, YubiKey, FIDO2, LUKS, Manjaro]
---

A few months ago, I wrote about my process for {{< backlink "implementing-full-disk-encryption-with-lvm-on-luks" >}} using [LVM on LUKS](https://wiki.archlinux.org/title/Dm-crypt/Encrypting_an_entire_system#LVM_on_LUKS). That setup has served me well for years, providing a solid foundation of security. However, I recently decided to push that security even further. A passphrase is strong, but it's still just "something you know." What if we could add "something you have"?

That's where hardware security keys come in. I recently picked up a pair of [YubiKey 5 Series](https://www.yubico.com/products/yubikey-5-overview/) keys and set out to integrate them into my boot process. My goal: to unlock my encrypted disk not with a passphrase, but with a PIN combined with a physical touch of my YubiKey.

This guide details the process of upgrading an existing LVM on LUKS installation to use FIDO2 for decryption at boot.

# Understanding the Basics

## Passwords vs. Hardware Keys

A traditional passphrase, while secure, can be vulnerable to keyloggers or shoulder-surfing. By switching to a FIDO2-compliant key, we adopt a phishing-resistant multi-factor authentication method right at the boot level.

The process relies on [systemd-cryptenroll](https://wiki.archlinux.org/title/Systemd-cryptenroll), a powerful tool that allows enrolling various authentication methods, what it calls "tokens", into the `LUKS2` header. Instead of just a passphrase, we can now have:

- A **FIDO2 Security Key** (like a YubiKey). The cryptographic secret never leaves the hardware device, making it incredibly secure.
- A **TPM 2.0 Chip**.
- A **Recovery Key** as a last-resort backup.

To make this work, we need to shift our boot process from the traditional [mkinitcpio](https://wiki.archlinux.org/title/Mkinitcpio) `encrypt` hook to the more modern `systemd`-based [initramfs](https://wiki.archlinux.org/title/Arch_boot_process#initramfs). This new environment understands how to communicate with hardware devices like a YubiKey.

## A Quick Refresher of Boot Process

Before we change any files, it helps to understand the key players in the Linux boot process and why we need to modify them. Think of it as a relay race:

1. **GRUB (The Bootloader)**: This is the very first program that runs. Its main job is to find and load the Linux kernel into memory. It's the one that gives you the menu to choose your operating system. In our setup, we'll tell it to pass fewer instructions to the kernel because our new `initramfs` will handle the details.
    
2. **`initramfs` (Initial RAM File System)**: Once the kernel is loaded, it needs a set of tools to perform crucial early tasks, like finding and unlocking your main encrypted disk. The `initramfs` is a tiny, temporary operating system loaded entirely into RAM for this purpose. It contains just enough software to mount the real root filesystem.
    
3. **`mkinitcpio` (The Builder)**: This tool isn't part of the boot process itself; it's the utility we use to *build* the `initramfs`. The `HOOKS` array inside its configuration file (`/etc/mkinitcpio.conf`) is like a shopping list that tells `mkinitcpio` what tools and drivers to pack into that tiny OS.

### Why We Are Changing Things

Our goal is to swap out the old, simple `initramfs` for a new, more powerful one based on **systemd**.

- The **old `encrypt` hook** we were using is simple: it knows how to ask for a passphrase and not much else.
- The **new `sd-encrypt` hook** builds a `systemd`-based `initramfs`. This modern environment is much more flexible and, crucially, knows how to communicate with hardware devices using protocols like FIDO2.

So, in essence, we're telling `mkinitcpio` to pack a new toolkit (`systemd` and `sd-encrypt`) into the `initramfs`. This new toolkit has the right tools to talk to our YubiKey at boot, allowing us to unlock the system before the main operating system even loads.

# Let's Get It Done

## Install Prerequisites

First, `systemd-cryptenroll` relies on `libfido2` to communicate with FIDO2 devices. Let's make sure it's installed.

```shell
sudo pacman -S libfido2
```

## Enroll Your Keys

This is the most critical part. We'll add three new ways to unlock the disk: a recovery key (as a failsafe) and our two YubiKeys.

{{< admonition type="warning" title="Identify Your LUKS Partition" >}}
All these commands target your main LUKS partition. In my case, it's **/dev/nvme0n1p3**. Make sure to replace this with your own partition path.
{{< /admonition >}}

First, generate and enroll a **recovery key**. This is a high-entropy key that acts as a secure, one-time-use password in case you lose all your YubiKeys. The system will generate it and display it on the screen.

```shell
sudo systemd-cryptenroll /dev/nvme0n1p3 --recovery-key
```

{{< admonition >}}
Store this recovery key somewhere extremely safe and offline, like in a password manager's secure notes or a physical safe. Do not lose it!
{{< /admonition >}}

Next, enroll your first YubiKey. The `fido2-device=auto` flag will automatically detect any connected FIDO2 key.

```shell
sudo systemd-cryptenroll /dev/nvme0n1p3 --fido2-device=auto
```

The system will prompt you to enter a PIN for the key (you'll set one if it's new) and then ask you to touch the key's flashing sensor to confirm its presence.

Repeat the exact same command for your second (backup) YubiKey.

## Migrate the Boot Process to `systemd`

Now we need to tell the system to use the new `systemd`-based `initramfs` which knows how to handle FIDO2 devices.

### Create a `crypttab` for the `initramfs`

Instead of passing complex instructions via the kernel command line, the cleaner method is to create a configuration file that the `initramfs` will read. This file is `/etc/crypttab.initramfs`.

Create the file and add the following line:

```ini
# /etc/crypttab.initramfs
# <name>        <device>                                     <password>  <options>
cryptmanjaro    UUID=955cc284-fafb-4132-a2b9-4893e4254b4d    none        fido2-device=auto
```

This tells `systemd`:

- **`cryptmanjaro`**: The name for the unlocked device (`/dev/mapper/cryptmanjaro`).
- **`UUID=...`**: Which LUKS partition to target. Use the UUID of your LUKS partition.
- **`none`**: Don't use a password or keyfile.
- **`fido2-device=auto`**: Use an auto-detected FIDO2 device as the unlock method.

### Update Kernel Parameters in GRUB

Since `/etc/crypttab.initramfs` now contains all the necessary instructions, we can clean up our GRUB configuration.

Edit `/etc/default/grub` and find the `GRUB_CMDLINE_LINUX` line. It used to contain the `cryptdevice=` parameter. We can now clear it.

```shell
# /etc/default/grub

# Before:
# GRUB_CMDLINE_LINUX="cryptdevice=UUID=955cc284-fafb-4132-a2b9-4893e4254b4d:cryptmanjaro"

# After:
GRUB_CMDLINE_LINUX=""
```

{{< admonition type="tip" >}}
Setting the line to an empty string (`""`) is a safe way to clear it. You could also delete the line entirely, but leaving it empty clearly shows it was an intentional change.
{{< /admonition >}}

### Switch to `systemd` hooks in `mkinitcpio`

Finally, we instruct `mkinitcpio` to build a `systemd`-based `initramfs` by changing the hooks.

Edit `/etc/mkinitcpio.conf` and modify the `HOOKS` array:

```shell
# /etc/mkinitcpio.conf

# Before:
# HOOKS=(base udev autodetect ... block encrypt lvm2 ...)

# After:
HOOKS=(base systemd autodetect ... block sd-encrypt lvm2 ...)
```

The key changes are replacing `udev` with `systemd` and `encrypt` with `sd-encrypt`. I also swapped `keymap` for `sd-vconsole` for better `systemd` integration.

## Apply All Changes

The configuration files are now set up. The final step is to generate the new `initramfs` and update the GRUB configuration file.

```shell
# Generate the initramfs images
sudo mkinitcpio -P

# Generate the GRUB config file
sudo grub-mkconfig -o /boot/grub/grub.cfg
```

# The New Boot Experience

That's it! After a reboot, you'll be greeted with a simple text prompt asking for your FIDO2 PIN. After entering it, the system will ask you to touch your YubiKey. Once you do, the disk will decrypt, and the boot process will continue as normal.

Welcome to the future of secure boot! 🚀

# Wrapping Up

Switching from a passphrase to a YubiKey for disk encryption adds a powerful layer of physical security to any Linux machine.
With this setup:

- ✔ Your drive is unlocked with a hardware-backed key.
- ✔ The process is resistant to phishing and keylogging.
- ✔ You have a secure recovery key and a backup hardware key for peace of mind.

For further reading, check the [Arch Linux Wiki](https://wiki.archlinux.org/title/Main_page) on [Unlocking in early userspace](https://wiki.archlinux.org/title/Dm-crypt/System_configuration#Unlocking_in_early_userspace).
