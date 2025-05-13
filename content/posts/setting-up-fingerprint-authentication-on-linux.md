---
title: Setting Up Fingerprint Authentication on Linux
date: 2025-05-13
summary: A guide to configuring and using the fingerprint sensor on Manjaro Linux KDE Plasma 6, specifically for a Lenovo ThinkPad E14 Gen 5 14, including troubleshooting common issues.
categories: [Linux, Fingerprint, Authentication, Manjaro, KDE Plasma, ThinkPad]
---

[Fingerprint readers](https://en.wikipedia.org/wiki/Fingerprint_scanner) offer a convenient and secure way to authenticate on your Linux system. This guide walks through the process of setting up fingerprint authentication on Manjaro Linux with KDE Plasma 6, based on my experience with a [Lenovo ThinkPad E14 Gen 5 14](https://pcsupport.lenovo.com/us/en/products/laptops-and-netbooks/thinkpad-edge-laptops/thinkpad-e14-gen-5-type-21jk-21jl).

We'll cover identifying your sensor, installing drivers, enrolling fingerprints, and configuring it for `sudo` and the lock screen, including a workaround for a current [Plasma 6](https://kde.org/plasma-desktop/) bug.

# Understanding the Basics

## Why Use Fingerprint Authentication?

Fingerprint authentication provides a quick and often more convenient alternative to typing passwords. Instead of repeatedly entering complex passphrases, a simple touch can grant access to your system or elevate privileges. This is particularly useful for actions like unlocking the screen or using `sudo` commands.

# Let's Get Started

## Identifying Your Fingerprint Sensor

The first step is to identify the fingerprint sensor your laptop uses. Most fingerprint readers are connected via USB, even if integrated into the laptop's chassis. You can list USB devices connected to your system with the following command:

```shell
lsusb
```

Look for a line that indicates a fingerprint reader. In my case, on the [Lenovo ThinkPad E14 Gen 5 14](https://pcsupport.lenovo.com/us/en/products/laptops-and-netbooks/thinkpad-edge-laptops/thinkpad-e14-gen-5-type-21jk-21jl), this helped identify the specific model, which is crucial for finding the correct drivers. For my device, it's a Goodix sensor:

```shell
Bus 003 Device 011: ID 27c6:550a Shenzhen Goodix Technology Co.,Ltd. FingerPrint
```

## Installing Necessary Packages

Once you've identified your sensor, you'll need to install [fprintd](https://fprint.freedesktop.org/), the service that manages fingerprint authentication in Linux, and the appropriate driver for your hardware.

For my Goodix sensor, the driver [libfprint-2-tod1-goodix](https://aur.archlinux.org/packages/libfprint-2-tod1-goodix) is available in the [Arch User Repository (AUR)](https://aur.archlinux.org/). I used [yay](https://github.com/Jguer/yay) to install both `fprintd` and the driver:

```shell
yay -S fprintd libfprint-2-tod1-goodix
```

{{< admonition type="note" >}}
If you have a different fingerprint sensor, you'll need to search the AUR or official repositories for the correct `libfprint` driver.
{{< /admonition >}}

## Enrolling and Verifying Fingerprints

With the driver installed, you can now enroll your fingerprints. Use the `fprintd-enroll` command. You can enroll multiple fingers.

```shell
fprintd-enroll
```

The command will prompt you to scan your finger multiple times. Follow the on-screen instructions.

After enrolling, you can verify that the fingerprint is working correctly:

```shell
fprintd-verify
```

This command will ask you to scan your finger again and confirm if it matches a registered fingerprint.

## Using Fingerprint for `sudo` and `su`

To use your fingerprint for `sudo` commands or when switching users with `su`, you need to modify the relevant [PAM (Pluggable Authentication Modules)](https://www.redhat.com/en/blog/pluggable-authentication-modules-pam) configuration files.

{{< admonition type="warning" >}}
Always be careful when editing PAM files. An incorrect configuration can lock you out of your system. It's wise to keep a root terminal open while testing these changes.
{{< /admonition >}}

Edit `/etc/pam.d/sudo` and `/etc/pam.d/su`. Add the following lines at the beginning of the `auth` section, as recommended by the [Arch Wiki on Fprint](https://wiki.archlinux.org/title/Fprint#Login_configuration):

```
#%PAM-1.0

# BEGIN - Fingerprint support
auth      [success=1 default=ignore]  pam_succeed_if.so   service in sudo:su:su-l tty in :unknown
auth      sufficient pam_fprintd.so
# END - Fingerprint support

auth		  include		system-auth
account		include		system-auth
session		include		system-auth
```

Place these lines before other `auth` rules. The `sufficient` keyword means that if fingerprint authentication is successful, no further authentication methods (like prompting for a password) will be required. The `pam_succeed_if.so` line is an attempt to prevent fingerprint authentication for remote sessions, though its effectiveness can vary.

{{< admonition type="note" >}}
When fingerprint authentication is enabled for `sudo` and `su`, pressing `Ctrl+C` during the fingerprint prompt will allow you to enter your password as an alternative.
{{< /admonition >}}

## Fingerprint Authentication for KDE Plasma 6 Lock Screen

[KDE Plasma 6](https://kde.org/plasma-desktop/) has built-in support for using fingerprint authentication to unlock the screen, which is a very convenient feature. By default, if `fprintd` is set up and fingerprints are enrolled, Plasma should offer fingerprint unlock on the lock screen.

### Addressing the `pam_faillock` Bug in Plasma 6

However, there’s a known issue with KDE Plasma 6 (as of early 2025) where fingerprint authentication on the lock screen can trigger `pam_faillock` even when a correctly scanned fingerprint is provided. The bug causes these successful scans to be erroneously interpreted as failed login attempts. When enough of these 'false failures' accumulate within the time limit (e.g., three times in under 15 minutes), `pam_faillock` locks the account for a period (typically 10 minutes), as if there were actual brute-force attempts.

To mitigate this, you can adjust the PAM configuration for KDE's fingerprint service. The solution involves modifying `/etc/pam.d/kde-fingerprint`.

Here's the updated configuration for `/etc/pam.d/kde-fingerprint`, based on a solution found in the [Frame.work community forums](https://community.frame.work/t/guide-solved-sudo-and-login-with-fingerprint-reader-under-kde-arch-linux/37009/8):

```
#%PAM-1.0

auth       required                        pam_shells.so
auth       requisite                       pam_nologin.so
auth       requisite                       pam_faillock.so     preauth

# Default behaviour (commented out)
#-auth      required                        pam_fprintd.so
#auth      optional                        pam_permit.so

# Solution with faillock support
auth       [success=1 ignore=ignore module_unknown=die default=bad] pam_fprintd.so max-tries=3 timeout=65535
auth       [default=die]                   pam_faillock.so     authfail
auth       optional                        pam_permit.so
auth       required                        pam_faillock.so     authsucc

auth       required                        pam_env.so
account    include                         system-local-login
password   required                        pam_deny.so
session    include                         system-local-login
```

This configuration aims to integrate `pam_fprintd.so` more gracefully with `pam_faillock.so`:
- `pam_fprintd.so max-tries=3 timeout=65535`: Allows up to 3 fingerprint attempts with a very long timeout (effectively no immediate timeout by `pam_fprintd` itself for a single authentication attempt).
- The subsequent `pam_faillock.so` lines manage the actual locking behavior if fingerprint authentication ultimately fails after these tries.

# That's All

With these configurations in place, you should have a functional fingerprint authentication system on your Manjaro Linux with KDE Plasma 6. You can use your fingerprint for `sudo`, `su`, and unlocking your screen, with improved handling of potential misreads on the lock screen.

Always refer to the [Arch Wiki Fprint page](https://wiki.archlinux.org/title/Fprint) and relevant community forums for the latest information and troubleshooting tips, as software and configurations can evolve.

{{< admonition type="note" >}}
It's important to note that this configuration doesn't use fingerprint authentication for the initial login (e.g., after a reboot), only for unlocking a locked session. This is often a preferred security posture.
{{< /admonition >}}
