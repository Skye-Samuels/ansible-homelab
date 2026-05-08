# Full Setup Guide

This guide provides a comprehensive, step-by-step walkthrough for users starting from scratch.

## 1. Debian Installation
- **ISO**: I recommend the Debian 13 (trixie) ISO as that is what I have tested it on.
- **Base System**: During installation, select only Standard System Utilities and SSH Server. A GUI is not recommended for this server stack.

## 2. Server Account Setup
Before configuring SSH, you must create the necessary accounts on your server. SSH into your server as `root` and run the following:

### Install Sudo & Create Automation User (Ansible)
On minimal Debian installs, `sudo` is not installed by default. Run these as `root`:
```bash
apt update && apt install sudo -y

# Create the automation user
adduser ansible
usermod -aG sudo ansible
echo "ansible ALL=(ALL) NOPASSWD:ALL" > /etc/sudoers.d/ansible
```

### Create Primary Admin User
This is your personal account for manual management. Run these as `root`:

> **IMPORTANT**: The username you use here **MUST** match the `admin_user` value you set in `group_vars/all/vars.yml`.

```bash
# Replace 'sysadmin' with your chosen admin_user from vars.yml
adduser sysadmin
usermod -aG sudo sysadmin
```

## 3. SSH Configuration
Now, generate and push SSH keys from your **control machine** to these accounts.

### Automation Key (Ansible)
1. **Generate Key**:
    ```bash
    ssh-keygen -t ed25519 -f ~/.ssh/ansible_key
    ```
    *Press Enter when prompted for a passphrase to keep it empty for automation.*
2. **Copy Key to Server**:
    ```bash
    ssh-copy-id -i ~/.ssh/ansible_key ansible@your-server-ip
    ```

### Manual Admin Key
1. **Generate Key**:
    ```bash
    ssh-keygen -t ed25519 -f ~/.ssh/sysadmin_key
    ```
    *Enter a strong passphrase when prompted.*
2. **Copy Key to Admin User**: Replace `sysadmin` with your chosen `admin_user`:
    ```bash
    ssh-copy-id -i ~/.ssh/sysadmin_key sysadmin@your-server-ip
    ```

## 4. Verify Connection
Test that you can log in to both accounts successfully.

- **Ansible**: `ssh -i ~/.ssh/ansible_key ansible@your-server-ip` (Should log in immediately).
- **Admin**: `ssh -i ~/.ssh/sysadmin_key sysadmin@your-server-ip` (Should prompt for your passphrase).

## 5. Preparing the Control Machine
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

## 6. Configuration
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

## 7. Deployment
To deploy the entire stack:
```bash
ansible-playbook main.yml --ask-vault-pass -K
```

To deploy or update a specific service:
```bash
ansible-playbook main.yml --tags <service_name> --ask-vault-pass -K
```

## 8. Application Configuration
After the containers are successfully running, you will need to perform some manual configuration inside each application's web interface.

### Jellyfin Stack
- **Jellyfin**: Access at `https://jellyfin.<your-domain>`. Add your libraries (mapped to `/data` inside the container) and set up your admin account.
- **Management (Sonarr, Radarr, Prowlarr)**: 
    - **Prowlarr**: Add your indexers and sync them to Sonarr/Radarr.
    - **Sonarr/Radarr**: Configure your download client (qBitTorrent) and set your media paths.
- **Jellyseerr**: A request management and media discovery tool. Connect it to your Jellyfin, Sonarr, and Radarr instances.
- **Bazarr**: Automatically searches for and downloads subtitles for your media libraries.
- **Wizarr**: A user-friendly invite system to easily share your Jellyfin server with others.
- **Newtarr**: Enhances your stack with automated list management and media notifications.
- **FlareSolverr**: A proxy server to bypass Cloudflare and DDoS protection for your indexers.

### Nextcloud
- **Admin Setup**: Follow the Nextcloud AIO interface at `https://cloud.<your-domain>:8081` to finalize the installation and create your admin account.

## 9. Maintenance & Troubleshooting
- **Logs**: View real-time logs for any container: `docker logs -f <container_name>`
- **Status**: Run `lazydocker` on the server for a comprehensive terminal UI dashboard.
- **Updates**: Pull the latest repository changes and re-run the playbook to apply updates.
- **Storage Issues**: If the MergerFS pool is not visible, verify your disks are mounted correctly: `mount | grep /mnt/disk`

