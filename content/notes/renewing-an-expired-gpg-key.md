---
title: Renewing an Expired GPG Key
date: 2026-03-05
summary: To renew an expired GPG key, update its date via `gpg --edit-key`. Finally, distribute the updated public key to keyservers (`gpg --send-keys`) and sync your other machines (`gpg --recv-keys`).
categories: [GPG, Encryption, Security, CLI]
---

{{< admonition type="tip" title="TL;DR" >}}
To renew an expired GPG key, update its date via `gpg --edit-key`. Finally, distribute the updated public key to keyservers (`gpg --send-keys`) and sync your other machines (`gpg --recv-keys`).
{{< /admonition >}}

I like to set an expiration date on my GPG keys (usually 2 years) as a security best practice. However, because this is something I only do every 2 years, I always end up forgetting the exact steps when the key finally expires. 

It is important to understand that when you extend an expiration date, **your underlying private and public keys do not change**. You are simply adding a new self-signature to your public key that says: *I authorize this key to be valid for another X years.*

If you run `gpg --list-keys` and see that your key is marked as `[ expired ]`, here is the explicit, step-by-step process to renew it and sync it across your devices.

## Edit the Expiration Date

First, open the GPG interactive menu for your specific key using its ID or email address:

```shell
gpg --edit-key 87638F555971C9470EE0970292798C20EED0C422
```

Once inside the `gpg>` prompt, change the expiration date for your primary key:

1. Type `expire` and press Enter.
2. Specify the new validity period (e.g., type `2y` for two years).
3. Confirm the new date when prompted.

{{< admonition type="note">}}
If you use subkeys for encryption or signing, you must select them first by typing `key 1` (or the corresponding index) and then run the `expire` command again).
{{< /admonition >}}

Finally, type `save` to apply the changes and exit the prompt.

## Publish the Updated Key

Extending the expiration date locally is not enough. You must distribute this new expiration signature so that the rest of the world (and services like [GitHub](https://github.com)) knows your key is valid again and stops rejecting your signatures.

Push the updated key to the standard public keyservers:

```shell
gpg --keyserver keyserver.ubuntu.com --send-keys 87638F555971C9470EE0970292798C20EED0C422
gpg --keyserver keys.openpgp.org --send-keys 87638F555971C9470EE0970292798C20EED0C422
gpg --keyserver pgp.mit.edu --send-keys 87638F555971C9470EE0970292798C20EED0C422
```

## Syncing the Renewal to Your Other Devices

If you use your GPG key on multiple machines (e.g., a desktop and a laptop), your other devices already have your private key, but their local keyring still thinks the key is expired. You do not need to copy your private key again; you only need to import the updated **public key** to refresh the expiration date.

You have two options to achieve this.

### Option A: Fetch from a Keyserver (Recommended)

If you have pushed your key to a keyserver, this is the most elegant method. Go to your other machine and simply pull the updated public key:

```shell
gpg --recv-keys 87638F555971C9470EE0970292798C20EED0C422
```

GPG will download the new expiration signature from the internet and automatically apply it to your local keyring.

### Option B: Manual Import

If keyservers are down or you prefer a direct approach, you can export the public key manually to a file:

```shell
gpg --armor --export 87638F555971C9470EE0970292798C20EED0C422 > pubkey.asc
```

Later, copy the `pubkey.asc` file to your other machine (via SSH, a USB drive, etc.) and run the next command to import it:

```shell
gpg --import pubkey.asc
```

GPG will detect that you already possess the private key and will simply update the expiration date of the associated public key.
