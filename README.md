# Ansible Homelab

A modular, layered Ansible project to deploy a complete media and home automation stack on a Debian-based server.

## 🏗 Architecture & Layer Breakdown
This project uses a modular **Layered Deployment** strategy. Each layer is a standalone playbook that builds upon the previous one.

<details>
<summary><b>[Layer 0] Storage</b></summary>

**Purpose:** Prepares the physical storage array.
- **Technologies:** MergerFS, `ext4`, `mount`.
- **Details:** Formats secondary drives, creates mount points, and establishes a pooled `storage_root` using MergerFS for transparent disk spanning.
</details>

<details>
<summary><b>[Layer 1] Base System</b></summary>

**Purpose:** Initial server hardening and utility setup.
- **Details:** Sets hostname, configures Timezone, updates APT cache, and installs core admin tools (`neovim`, `btop`, `htop`, `fastfetch`).
</details>

<details>
<summary><b>[Layer 2] Networking & DNS</b></summary>

**Purpose:** Ensures stable connectivity.
- **Details:** Configures static IP settings (if defined) and ensures DNS persistence.
</details>

<details>
<summary><b>[Layer 3] Security</b></summary>

**Purpose:** Perimeter and access security.
- **Technologies:** UFW (Uncomplicated Firewall), Fail2Ban.
- **Details:** Closes all ports except essential ones (SSH, HTTP/S), configures Fail2Ban to prevent brute-force attacks, and hardens SSH config.
</details>

<details>
<summary><b>[Layer 4] Docker Engine</b></summary>

**Purpose:** The containerization foundation.
- **Details:** Installs Docker Engine, Docker Compose, and **LazyDocker** for terminal-based container management.
</details>

<details>
<summary><b>[Layer 5] Caddy Reverse Proxy</b></summary>

**Purpose:** Edge routing and SSL management.
- **Container:** `lucaslorentz/caddy-docker-proxy`.
- **Details:** Automatically discovers other containers via Docker labels and provisions Let's Encrypt / ZeroSSL certificates.
</details>

<details>
<summary><b>[Layer 6] Pi-hole</b></summary>

**Purpose:** Network-wide ad and tracker blocking.
- **Container:** `pihole/pihole`.
- **Details:** Deploys Pi-hole with a customized web port and automated DNS health checks.
</details>

<details>
<summary><b>[Layer 7] Nextcloud AIO</b></summary>

**Purpose:** Private cloud and productivity suite.
- **Container:** `nextcloud/all-in-one`.
- **Details:** Deploys the Nextcloud AIO mastercontainer, enabling a complete personal cloud with high-performance backend.
</details>

<details>
<summary><b>[Layer 8] Mealie</b></summary>

**Purpose:** Recipe and meal management.
- **Container:** `hkotel/mealie`.
- **Details:** Provides a centralized database for recipes with a clean, mobile-friendly web UI.
</details>

<details>
<summary><b>[Layer 9] Jellyfin Stack</b></summary>

**Purpose:** Fully automated media management and streaming.
- **Containers:** 
  - **Networking:** `gluetun` (VPN Gateway for the stack).
  - **Streaming:** `jellyfin`.
  - **Management:** `sonarr`, `radarr`, `prowlarr`, `bazarr`, `flaresolverr`.
  - **Utilities:** `jellyseerr` (Requests), `wizarr` (Invites), `newtarr`.
  - **Downloads:** `qbittorrent`.
- **Details:** All traffic for downloaders and indexers is strictly routed through the `gluetun` VPN container to ensure privacy.
</details>


## 🚀 Quick Start

### 1. Prerequisites
- A fresh install of **Debian** (Tested with **Trixie**).
- SSH access configured.
- `ansible` installed on your control machine.

### 2. Configure Variables
1. Copy the secret template:
   ```bash
   cp group_vars/all/vault.yml.example group_vars/all/vault.yml
   ```
2. Encrypt your secrets with Ansible Vault:
   ```bash
   ansible-vault create group_vars/all/vault.yml
   ```
   *(Paste the contents of the example file and add your real credentials).*
3. Update `group_vars/all/vars.yml` with your preferred Timezone, Paths, and Ports.

### 3. Deploy
To deploy the entire stack:
```bash
ansible-playbook main.yml --ask-vault-pass -K
```

To deploy a specific layer:
```bash
ansible-playbook 9_jellyfin.yml --ask-vault-pass -K
```

## 🔐 Security & Variable Management
This project follows the **"Ansible Way"**:
- **Centralization**: All configurations live in `group_vars/all/vars.yml`.
- **Encryption**: Sensitive data (PIA passwords, etc.) is stored in an encrypted `vault.yml` file using **Ansible Vault**.
- **Smoke Tests**: Every layer includes automated health checks to verify service availability before proceeding.

## 🛠 Maintenance
- **View Status**: Use `lazydocker` for container monitoring and `btop` for system performance.
- **Edit Secrets**: `ansible-vault edit group_vars/all/vault.yml`
- **Update Stack**: Simply pull the latest changes and re-run the `main.yml` playbook.

---

## ⚖️ Disclaimer
This project is for **educational purposes only**. It is intended for managing personal media backups and home automation. The author does not condone or encourage any form of digital piracy or the illegal distribution of copyrighted material.

