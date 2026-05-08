# Full Setup Guide

This guide provides a comprehensive, step-by-step walkthrough for users starting from scratch.

## 1. Debian Installation
- **ISO**: I recommend the Debian 13 (trixie) ISO as that is what I have tested it on.
- **Base System**: During installation, select only "Standard System Utilities" and "SSH Server." A GUI is not recommended for this server stack.

## 2. SSH Configuration
Before Ansible can connect, you must prepare the server for management.

1.  **Create Ansible User**: SSH into your new server as root or your initial user and create the management account:
    ```bash
    sudo adduser ansible
    sudo usermod -aG sudo ansible
    echo "ansible ALL=(ALL) NOPASSWD:ALL" | sudo tee /etc/sudoers.d/ansible
    ```
2.  **Generate SSH Keys** (on your control machine):
    ```bash
    ssh-keygen -t ed25519 -f ~/.ssh/ansible_key
    ```
3.  **Copy Key to Server**:
    ```bash
    ssh-copy-id -i ~/.ssh/ansible_key ansible@your-server-ip
    ```

## 3. Preparing the Control Machine
1.  **Install Ansible**: Follow the official guide for your OS (macOS, Linux, or WSL2).
2.  **Clone this Repository**:
    ```bash
    git clone https://github.com/Skye-Samuels/ansible-homelab.git
    cd ansible-homelab
    ```
3.  **Install Collections**:
    ```bash
    ansible-galaxy collection install -r requirements.yml
    ```

## 4. Configuration
You need to customize three main files before deploying:

1.  **Inventory**: Open the `inventory` file and replace the placeholder IP with your server's IP address.
2.  **Secrets**:
    ```bash
    cp group_vars/all/vault.yml.example group_vars/all/vault.yml
    ansible-vault encrypt group_vars/all/vault.yml
    ```
    *Use `ansible-vault edit group_vars/all/vault.yml` to add your real PIA and Pi-hole passwords.*
3.  **Variables**: Open `group_vars/all/vars.yml` and review all settings.
    *   **Identity**: Update hostname_desired, timezone, and domain.
    *   **Feature Flags**: Enable or disable services (Nextcloud, Mealie, Jellyfin) by setting enable_* flags to true or false.
    *   **Docker Config**: Ensure puid and pgid match your server's user (usually 1000).
    *   **Service Ports**: Review the default ports and VPN region.
    *   **Storage**: Verify disk_1_source and disk_2_source.

    > **CRITICAL: DATA LOSS WARNING**
    > The storage role will format these disks if format_disks is set to true.
    > **If your disks already contain data (movies, backups, etc.), set format_disks: false before running the playbook.**

## 5. Deployment
To deploy the entire stack:
```bash
ansible-playbook main.yml --ask-vault-pass -K
```

To deploy or update a specific service:
```bash
ansible-playbook main.yml --tags <service_name> --ask-vault-pass -K
```

## 6. Application Configuration
After the containers are successfully running, you will need to perform some manual configuration inside each application's web interface.

### Jellyfin Stack
- **Jellyfin**: The core media server. Add your libraries (mapped to `/data` inside the container) and set up your admin account.
- **Management (Sonarr, Radarr, Prowlarr)**: 
    - **Prowlarr**: Add your indexers and sync them to Sonarr/Radarr.
    - **Sonarr/Radarr**: Configure your download client (qBitTorrent) and set your media paths.
- **Jellyseerr**: A request management and media discovery tool. Connect it to your Jellyfin, Sonarr, and Radarr instances.
- **Bazarr**: Automatically searches for and downloads subtitles for your media libraries.
- **Wizarr**: A user-friendly invite system to easily share your Jellyfin server with others.
- **Newtarr**: Enhances your stack with automated list management and media notifications.
- **FlareSolverr**: A proxy server to bypass Cloudflare and DDoS protection for your indexers. You must add this as a "Proxy" in Prowlarr's indexer settings.

### Nextcloud
- **Admin Setup**: Follow the Nextcloud AIO interface at `https://cloud.<your-domain>:8081` to finalize the installation and create your admin account.

