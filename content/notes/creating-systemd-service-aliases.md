---
title: Creating systemd Service Aliases
date: 2025-09-18
summary: "Use the `Alias=` directive in a systemd unit file to create an additional name for a service. This is a cleaner, native alternative to manually creating symbolic links."
categories: [systemd, Linux, System Administration, CLI]
---

{{< admonition type="tip" title="TL;DR" >}}
Use the `Alias=` directive in a systemd unit file to create an additional name for a service. This is a cleaner, native alternative to manually creating symbolic links.
{{< /admonition >}}

I often find myself wanting to refer to a [**systemd**](https://systemd.io/) service with a shorter or different name, either for convenience or for compatibility reasons. For example, I might have `my-long-app-name.service` and want to control it using `app.service`.

The classic way to achieve this is by creating a symbolic link in `/etc/systemd/system/`:

```shell
sudo ln -s /etc/systemd/system/my-long-app-name.service /etc/systemd/system/app.service
```

While this works, it feels like a manual workaround that adds an extra file to manage directly on the filesystem. I recently learned there's a much cleaner, built-in way to do this.

## The systemd Way

The native solution is to use the [`Alias=`](https://www.freedesktop.org/software/systemd/man/latest/systemd.unit.html#Alias=) directive directly within the service's unit file. This option, specified in the `[Install]` section, tells systemd that the service can be referred to by another name.

Here’s how you would modify the unit file:

```ini
# /etc/systemd/system/my-long-app-name.service

[Unit]
Description=My Awesome Application

[Service]
ExecStart=/usr/local/bin/my-app-binary
# ... other service options

[Install]
WantedBy=multi-user.target
# Add the alias here
Alias=app.service
```

## How to Apply the Change

Unlike the manual `ln -s` approach, the alias is activated when the service is **enabled**. `systemd` creates and manages the necessary symbolic link for you as part of its standard lifecycle.

To apply the change to an existing service:

1. **Edit the unit file** to add the `Alias=` line:
    ```shell
    sudo systemctl edit my-long-app-name.service
    ```
2. **Reload the daemon** to pick up the change:
    ```shell
    sudo systemctl daemon-reload
    ```
3.  **Re-enable the service** to create the alias link:
    ```shell
    sudo systemctl reenable my-long-app-name.service
    ```

Now, you can start, stop, and check the status of the service using either name.

```shell
sudo systemctl status app.service
# ● my-long-app-name.service - My Awesome Application
#      Loaded: loaded (/etc/systemd/system/my-long-app-name.service; enabled; ...)
#      Active: active (running) since ...
```

## Conclusion

So, which method should you use? While a manual `ln -s` is a quick fix, the `Alias=` directive is the clear winner for any long-term setup.

It's far more **robust**, as it won't break during package updates that might move or rename the original service file. By keeping the configuration within `systemd` itself, your setup becomes self-documenting, cleaner, and adheres to **best practices** for system administration.

You can find more details in the official [`systemd.unit`](https://www.freedesktop.org/software/systemd/man/latest/systemd.unit.html) man page.
