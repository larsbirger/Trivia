# Deployment Procedure: Rootless Podman on ZFS

This guide documents the setup of a rootless Podman environment on an
Ubuntu system using the ZFS filesystem.

## 1. System Installation

Install the required packages. `uidmap` provides the tools for
user namespace mapping, and `fuse-overlayfs` is required to run Podman storage
on ZFS.

```bash
# Update and install system dependencies
sudo apt update
sudo apt install python3-pip uidmap fuse-overlayfs -y

# Install podman-compose globally via pip
sudo pip3 install podman-compose
```

## 2. Service User Setup

Create a dedicated user to isolate the container services.

```bash
# Create the service user
sudo adduser --disabled-password --gecos "" [podman-container-user-name]

# Enable lingering so containers start on boot and stay running after logout
sudo loginctl enable-linger [podman-container-user-name]
```

- **FYI:**

  the command:

  ```Bash
  sudo adduser --disabled-password --gecos "" 
  ```

  - `--disabled-password` prevents the need for password, as it is not supposed
    to be accessed through ssh.

  - `--gecos "" ` sets general user info like name and phone number to be `""`
    since it is not a "user" as in a "human" user.

## 3. Configure Sub-UID and Sub-GID

Define the range of UIDs and GIDs the service user is allowed to use for its
rootless containers.

```bash
# Assign 65,536 sub-IDs to the user and group
sudo usermod --add-subuids 100000-165535 [podman-container-user-name]
sudo usermod --add-subgids 100000-165535 [podman-container-user-name]

# Verify mapping was written correctly
cat /etc/subuid | grep [podman-container-user-name]

```

- **FYI:**
  
    the command:

    ```bash
    sudo usermod --add-subuids 100000-165535 [podman-container-user-name]
    ```  

    follows the convention:  

    ```bash
    sudo usermod --add-subuids [start number]-[count] [podman-container-user-name]
    ```

    this is similar for both `add-subguids` and `add-subuids`. (do note the `g`
    differentiating them.)

## 4. User-Space Configuration

Log in as the service user to configure the container storage driver and
registry settings.

```bash
# Switch to service user session
sudo -u [podman-container-user-name] -i

# Create configuration directory
mkdir -p ~/.config/containers

# Configure Podman to use fuse-overlayfs for ZFS compatibility
cat > ~/.config/containers/storage.conf << 'EOF'
[storage]
driver = "overlay"

[storage.options.overlay]
mount_program = "/usr/bin/fuse-overlayfs"
mountopt = "nodev,metacopy=on"
EOF

# Set default image search registry to Docker Hub
echo 'unqualified-search-registries = ["docker.io"]' > ~/.config/containers/registries.conf
```

## 5. Folder Permissions & ID Mapping

Because Podman is rootless, the internal container user (UID 1000)
must be mapped to the host directory permissions.

```bash
# Create the project data directory
mkdir -p ~/desec-updater/data

# Use podman unshare to set ownership for the container's internal UID
podman unshare chown -R 1000:1000 ~/desec-updater/data

# Ensure directory execution permissions
chmod 755 ~/desec-updater
```

## 6. Deployment

Initialize the environment and launch the container.

```bash
# Navigate to project directory
cd ~/desec-updater

# Refresh Podman internal mapping state
podman system migrate

# Start the container in detached mode
podman-compose up -d

# Verify status
podman ps
podman logs -f ddns-updater
```
