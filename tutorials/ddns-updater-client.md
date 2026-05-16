# Infrastructure Playbook: Rootless Podman Service Deployment via Systemd Quadlets

This document details the standardized deployment procedure for running isolated, rootless container services under dedicated, unprivileged system accounts. By leveraging systemd user lingering and dynamic variables (`%h`, `%U`), this playbook can be reused for any service without hardcoding specific User IDs (UIDs).

---

## Prerequisites (Run as Root or Sudo User)

Ensure your host system is updated, and then that systemd-container, and Podman is installed:

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install systemd-container -y
sudo apt install podman -y

```

---

## Step 1: Create the Dedicated Service Account

Define the target username as an environment variable in your terminal session, create the unprivileged user account, and enable background lingering so the services boot automatically on system startup.

```bash
# 1. Define the service account name
SVC_USER_DDNS="ddns-svc"

# 2. Create the system user with a default home directory and bash shell
sudo useradd -m -s /bin/bash "$SVC_USER_DDNS"

# 3. Enable systemd lingering for this specific user
sudo loginctl enable-linger "$SVC_USER_DDNS"

```

---

## Step 2: Set Up the Deployment Files

Switch into the newly created service user's isolated environment using `machinectl`. This ensures that all native systemd environment variables (like `XDG_RUNTIME_DIR`) are cleanly instantiated.

```bash
# Safely spawn a shell session inside the service user account
sudo machinectl shell "${SVC_USER_DDNS}@"

```

*(Note: Your terminal prompt will now reflect that you are operating as the service user).*

Next, build the configuration and data directory paths:

```bash
# Create the standard directory paths inside the user's home folder
mkdir -p ~/.config/containers/systemd/
mkdir -p ~/data

```

Create the Quadlet container definition file:

```bash
nano ~/.config/containers/systemd/ddns-updater.container

```

Paste the following completely dynamic Quadlet layout:

```ini
[Unit]
Description=DDNS Updater Service (Dynamic Rootless)
After=network-online.target

[Container]
Image=docker.io/qmcgaw/ddns-updater
ContainerName=ddns-updater
PublishPort=8000:8000/tcp
# %h automatically resolves to the current user's home directory
Volume=%h/data:/updater/data:Z,U
# %U dynamically injects the runtime user's specific UID/GID
Environment=TZ=Europe/Oslo PUID=%U PGID=%U
AutoUpdate=registry

[Service]
Restart=always

[Install]
WantedBy=default.target

```

*(Save and exit via `Ctrl+O`, `Enter`, `Ctrl+X`)*

---

## Step 3: Add the Application Configuration

Drop your actual DDNS operational variables or configuration data into the mapped data space before firing up the engine:

```bash
nano ~/data/config.json

```

**Paste your application's JSON configuration payload here, save, and exit.**

example config:

```JSON
{
    "settings": [
        {
        "provider": "desec",
        "domain": "bergmal.net",
        "token": "totallySafeToken@public.not...dont.upload.the.token",
        "ip_version": "ipv4",
        "ipv6_suffix": ""
        }
    ]
}
```

## Step 4: Initialize the Service

Force the user-level systemd manager to scan the new Quadlet file and automatically compile it into a rootless service, then start it.

```bash
# Reload systemd to parse the Quadlet definition
systemctl --user daemon-reload

# Start the newly generated background service
systemctl --user start ddns-updater

```

Verify that the service has cleanly spun up, initialized its internal engine, and bound to the container ports without any permission blocks:

```bash
journalctl --user -u ddns-updater -f

```

To close out the service account's terminal session and return to your primary user, execute:

```bash
exit

```

---

## Reference Commands for Administrators

Once deployment is finished, the service runs completely detached. Any administrative user with host `sudo` privileges can interact with the user session externally using the explicit account name string instead of tracking down numerical UIDs:

* **View Runtime Status:**

```bash
sudo systemctl --machine=ddns-svc@ --user status ddns-updater

```

* **Follow Live Operational Logs:**

```bash
    sudo journalctl --machine=ddns-svc@ --user -u ddns-updater -f

```

* **Force Cycle Restart:**

```bash
    sudo systemctl --machine=ddns-svc@ --user restart ddns-updater

```
