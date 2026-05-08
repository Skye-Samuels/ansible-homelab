# Ansible Homelab

A modular, layered Ansible project to deploy a complete media and home automation stack on a Debian-based server.

## 🏗 Architecture & Layer Breakdown
This project uses a modular **Role-based** deployment strategy. Each layer is an Ansible Role orchestrated by `main.yml`.

<details>
<summary><b>[Layer 0] Storage</b></summary>

- **Purpose:** Prepares the physical storage array.
- **Technologies:** MergerFS, `ext4`, `mount`.
- **Details:** Formats secondary drives, creates mount points, and establishes a pooled `storage_root` using MergerFS for transparent disk spanning.
</details>

<details>
<summary><b>[Layer 1] Base System</b></summary>

- **Purpose:** Initial server hardening and utility setup.
- **Details:** Sets hostname, configures Timezone, updates APT cache, and installs core admin tools (`neovim`, `btop`, `htop`, `fastfetch`).
</details>

<details>
<summary><b>[Layer 2] Networking & DNS</b></summary>

- **Purpose:** Ensures stable connectivity.
- **Details:** Configures static IP settings (if defined) and ensures DNS persistence.
</details>

<details>
<summary><b>[Layer 3] Security</b></summary>

- **Purpose:** Perimeter and access security.
- **Technologies:** UFW (Uncomplicated Firewall), Fail2Ban.
- **Details:** Closes all ports except essential ones (SSH, HTTP/S), configures Fail2Ban to prevent brute-force attacks, and hardens SSH config.
</details>

<details>
<summary><b>[Layer 4] Docker Engine</b></summary>

- **Purpose:** The containerization foundation.
- **Details:** Installs Docker Engine, Docker Compose, and **LazyDocker** for terminal-based container management.
</details>

<details>
<summary><b>[Layer 5] Caddy Reverse Proxy</b></summary>

- **Purpose:** Edge routing and SSL management.
- **Container:** `lucaslorentz/caddy-docker-proxy`.
- **Details:** Automatically discovers other containers via Docker labels and provisions Let's Encrypt / ZeroSSL certificates.
</details>

<details>
<summary><b>[Layer 6] Pi-hole</b></summary>

- **Purpose:** Network-wide ad and tracker blocking.
- **Container:** `pihole/pihole`.
- **Details:** Deploys Pi-hole with a customized web port and automated DNS health checks.
</details>

<details>
<summary><b>[Layer 7] Nextcloud AIO</b></summary>

- **Purpose:** Private cloud and productivity suite.
- **Container:** `nextcloud/all-in-one`.
- **Details:** Deploys the Nextcloud AIO mastercontainer, enabling a complete personal cloud with high-performance backend.
</details>

<details>
<summary><b>[Layer 8] Mealie</b></summary>

- **Purpose:** Recipe and meal management.
- **Container:** `hkotel/mealie`.
- **Details:** Provides a centralized database for recipes with a clean, mobile-friendly web UI.
</details>

<details>
<summary><b>[Layer 9] Jellyfin Stack</b></summary>

- **Purpose:** Fully automated media management and streaming.
- **Containers:** 
  - **Networking:** `gluetun` (VPN Gateway for the stack).
  - **Streaming:** `jellyfin`.
  - **Management:** `sonarr`, `radarr`, `prowlarr`, `bazarr`, `flaresolverr`.
  - **Utilities:** `jellyseerr` (Requests), `wizarr` (Invites), `newtarr`.
  - **Downloads:** `qbittorrent`.
- **Details:** All traffic for downloaders and indexers is strictly routed through the `gluetun` VPN container to ensure privacy.
</details>


## 🚀 Quick Start

Follow these steps to get your homelab up and running.

### Step 1: Prepare your Control Machine
Ensure you have Ansible installed, then install the required dependencies:
```bash
ansible-galaxy collection install -r requirements.yml
```

### Step 2: Configure Your Environment
You need to customize three main files before deploying:

1.  **Inventory**: Open the `inventory` file and replace the placeholder IP with your server's IP address.
2.  **Secrets**:
    ```bash
    cp group_vars/all/vault.yml.example group_vars/all/vault.yml
    ansible-vault encrypt group_vars/all/vault.yml
    ```
    *Use `ansible-vault edit group_vars/all/vault.yml` to add your real PIA and Pi-hole passwords.*
3.  **Variables**: Open `group_vars/all/vars.yml` and review all settings.
    *   **Identity**: Update `hostname_desired`, `timezone`, and `domain`.
    *   **Feature Flags**: Enable or disable services (Nextcloud, Mealie, Jellyfin) by setting `enable_*` flags to `true` or `false`.
    *   **Docker Config**: Ensure `puid` and `pgid` match your server's user (usually `1000`).
    *   **Service Ports**: Review the default ports and VPN region.
    *   **Storage**: Verify `disk_1_source` and `disk_2_source`.

    > ⚠️ **CRITICAL: DATA LOSS WARNING**
    > The storage role will **format** these disks if `format_disks` is set to `true`.
    > **If your disks already contain data (movies, backups, etc.), set `format_disks: false` before running the playbook.**

### Step 3: Deploy
Run the playbook from the root of the project:
```bash
ansible-playbook main.yml --ask-vault-pass -K
```
---

## 🔐 Security & Variable Management
- **Centralization**: All configurations live in `group_vars/all/vars.yml`.
- **Encryption**: Sensitive data (PIA passwords, etc.) is stored in an encrypted `vault.yml` file using **Ansible Vault**.
- **Modernization**: Uses Fully Qualified Collection Names (FQCN) and dynamic fact detection for cross-platform compatibility.

## 🛠 Maintenance & Uninstallation
- **View Status**: Use `lazydocker` for container monitoring and `btop` for system performance.
- **Edit Secrets**: `ansible-vault edit group_vars/all/vault.yml`
- **Soft Teardown**: To remove a service but keep its data, set its `enable_*` flag to `false` in `vars.yml` and run `main.yml`.
- **Hard Purge**: To permanently delete a service's persistent data, run:
  ```bash
  ansible-playbook purge_data.yml
  ```
  *(This will prompt you for the service name and a confirmation).*
- **Update Stack**: Simply pull the latest changes and re-run the `main.yml` playbook.

---

## ⚖️ Disclaimer
This project is for **educational purposes only**. It is intended for managing personal media backups and home automation. The author does not condone or encourage any form of digital piracy or the illegal distribution of copyrighted material.
