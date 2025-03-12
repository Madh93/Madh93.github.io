---
title: Implementing Full Disk Encryption on Linux
date: "2025-03-12"
summary: A comprehensive guide on setting up Manjaro Linux with full disk encryption using LVM on LUKS, based on five years of personal experience.
categories: [Linux, Encryption, LVM, LUKS, Manjaro]
---

Over the past five years, I've been running [Manjaro Linux](https://manjaro.org/) on my work laptop, utilizing full disk encryption with [LVM on LUKS](https://wiki.archlinux.org/title/Dm-crypt/Encrypting_an_entire_system#LVM_on_LUKS). This setup has provided a robust balance between security and flexibility. Below is a step-by-step guide to achieve this configuration.

# Understanding the Basics

## Why use Disk Encryption?

[Data-at-rest](https://en.wikipedia.org/wiki/Data_at_rest) encryption ensures all your files are encrypted while they're stored on your disk. This means that if someone gains unauthorized physical access, they'll only see gibberish. Your data is only decrypted when you're logged in and actively using the system. This is crucial for protection, even if your laptop is stolen or you discard an old hard drive.

## Types of Disk Encryption

There are [two primary methods](https://wiki.archlinux.org/title/Data-at-rest_encryption#Block_device_vs_stacked_filesystem_encryption) of disk encryption:

- **Filesystem-Level Encryption (FS Encryption):** Encrypts individual files or directories within the filesystem. While this allows for selective encryption, it may leave metadata (such as file names and sizes) exposed.
- **Block Device Encryption:** Encrypts entire disk partitions or volumes at the block level, making all data on the device inaccessible without proper authorization. This method is more comprehensive, as it typically encrypts all data, including metadata, providing a higher level of security.

For my setup, I've chosen the more comprehensive block device encryption, specifically using `dm-crypt` with LUKS.

## About `dm-crypt` and LUKS...

For implementing Block Device Encryption in Linux, `dm-crypt` is the standard tool.

[**dm-crypt**](https://en.wikipedia.org/wiki/Dm-crypt) is a feature within the Linux kernel that provides transparent block device encryption. Think of this as the core engine for encrypting your drives in Linux.

Meanwhile, [**LUKS**](https://en.wikipedia.org/wiki/Linux_Unified_Key_Setup) is a user-friendly layer built on top of `dm-crypt`. It makes it much easier to manage your encryption keys and settings. LUKS stores all the necessary information on your disk, simplifying the setup and improving security. Basically, LUKS makes using `dm-crypt` much more accessible.

## ... and LVM!

Beyond encryption, [**LVM (Logical Volume Management)**](https://en.wikipedia.org/wiki/Logical_Volume_Manager_(Linux)) provides an additional layer of powerful flexibility. LVM sits between your physical storage and the filesystem, allowing you to resize partitions, create snapshots, and manage storage more efficiently.

## Combining LVM on LUKS

By placing LVM on top of a LUKS-encrypted partition, you achieve both security and flexibility. The process involves encrypting a physical partition with LUKS and then setting up LVM within that encrypted space. This approach ensures that all logical volumes (e.g., `/root`, `/home`, `swap`) benefit from encryption, and only one passphrase is needed to unlock the entire volume group.

# Let's get started

## My Partition Scheme

In my setup, I configured the following partitions:

1. **EFI (`/boot/efi`):** A 512 MB partition formatted as `FAT32` to support UEFI boot.
2. **Boot (`/boot`):** A 1 GB {{< sidenote "partition" >}}While I understood that a separate boot partition wasn't always required, I've encountered issues installing Manjaro Architect without one. The process freezes at some point 🫤{{< /sidenote >}} formatted as `ext4` to hold the kernel and initial ramdisk.
3. **Encrypted Partition:** The remaining space is allocated for LUKS encryption, within which LVM manages:
 - **Root (`/`):** Initially allocated 100 GB for the operating system.
 - **Home (`/home`):** Assigned the remaining space for user data.

To visualize this, it would look something like this:

```
+----------------+----------------+-------------------------------------------+
| EFI Partition  | Boot Partition | Root Logical Volume | Home Logical Volume |
|                |                |                     |                     |
| /boot/efi      | /boot          | /                   | /home               |
|                |                |                     |                     |
|                |                | /dev/vgmanjaro/root | /dev/vgmanjaro/home |
| (may be on     | (may be on     |_ _ _ _ _ _ _ _ _ _ _|_ _ _ _ _ _ _ _ _ _ _|
| other device)  | other device)  |           Volume Group (vgmanjaro)        |
|                |                |                                           |
|                |                |  LUKS Encrypted Partition (cryptmanjaro)  |
| /dev/nvme0n1p1 | /dev/nvme0n1p2 |               /dev/nvme0n1p3              |
+----------------+----------------+-------------------------------------------+
```

{{< admonition >}}
I opted against a separate swap partition, choosing instead to use a [**swap file**](https://wiki.archlinux.org/title/Swap#Swap_file) within the root filesystem for greater flexibility.
{{< /admonition >}}

## Preparing the Installation Environment

Before starting, it's essential to prepare the installation environment:

1. **Download the Manjaro Architect ISO**: Although [Manjaro Architect](https://wiki.manjaro.org/index.php/Installation_with_Manjaro_Architect) has been discontinued, it's still possible to find previous versions in [community repositories](https://github.com/manjaro-architect/download/releases).
2. **Create a Bootable USB Drive**: Use tools like `dd` or [Etcher](https://www.balena.io/etcher/) to create a bootable USB drive with the downloaded ISO.
3. **Boot from the USB Drive**: Configure your system's BIOS/UEFI to boot from the USB drive.

## Partitioning the Disk

Once inside the Manjaro Architect environment, start by identifying the target disk. Use the following command to list available disks:

```sh
fdisk -l
```

Alternatively, you can use:

```sh
lsblk
```

This will display the available storage devices. Identify the target disk, for example, `/dev/nvme0n1`.

Next, launch the partitioning tool with:

```sh
cfdisk /dev/nvme0n1
```

In `cfdisk`, create the necessary partitions:

- **EFI Partition**: Set up a 512 MB partition and mark it as `EFI System`.
- **Boot Partition**: Create a 1 GB partition and set its type to `Linux filesystem`.
- **LUKS Partition**: Allocate the remaining space and set it as `Linux filesystem`.

Once the partitions are configured, write the changes to disk and exit `cfdisk`.

## Encrypting the LUKS Partition

To encrypt the main partition, initialize `LUKS` on it:

```shell
cryptsetup luksFormat /dev/nvme0n1p3
```

{{< admonition type="warning" >}}
Replace **/dev/nvme0n1p3** with your actual LUKS partition.
{{< /admonition >}}

You'll be prompted to confirm the action and set a passphrase. Once encrypted, open the partition to make it accessible:

```shell
cryptsetup open --type luks /dev/nvme0n1p3 cryptmanjaro
```

This command maps the encrypted partition to `/dev/mapper/cryptmanjaro`, allowing further operations.

{{< admonition type="tip" >}}
To change the passphrase without removing the partition, you can run the next command (once it's been opened):

```shell
cryptsetup luksChangeKey /dev/nvme0n1p3 -S 0
```
{{< /admonition >}}

## Setting Up LVM on the Encrypted Partition

With the encrypted partition unlocked, create a **Physical Volume (PV)**:

```shell
pvcreate /dev/mapper/cryptmanjaro
```

Next, set up a **Volume Group (VG)** named `vgmanjaro` (or your preferred name):

```shell
vgcreate vgmanjaro /dev/mapper/cryptmanjaro
```

Now, create **Logical Volumes (LV)** within the volume group.

First, allocate 100 GB for the `root` volume:

```shell
lvcreate -L 100G -n root vgmanjaro
```

And later, assign the remaining space to the `home` volume:

```shell
lvcreate -l +100%FREE -n home vgmanjaro
```

Finally, format the logical volumes with the `ext4` filesystem:

```shell
mkfs.ext4 /dev/vgmanjaro/root
mkfs.ext4 /dev/vgmanjaro/home
```

# That's All

With the partitions set up, you can proceed with the Manjaro installation by launching the Architect installer:

```shell
setup
```

Follow the prompts to install Manjaro Linux 🐧.

{{< admonition >}}
Once the installation is complete, you should consider to switch to the **stable** branch. Manjaro Architect initially configures the system to use the **unstable** branch, to change this just run:

```shell
sudo pacman-mirrors --api --set-branch stable
```
{{< /admonition >}}

# Wrapping Up

Setting up **LVM on LUKS** in provides a powerful balance between security and flexibility. With this setup:

- ✔ Your **data is fully encrypted** and protected from unauthorized access.
- ✔ LVM ensures **easy management of disk space**, allowing resizing and snapshots.

For further reading, check the [Arch Linux Wiki](https://wiki.archlinux.org/title/Main_page) on [Encrypting an entire system](https://wiki.archlinux.org/title/Dm-crypt/Encrypting_an_entire_system).
