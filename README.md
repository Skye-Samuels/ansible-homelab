# Ansible Homelab

A modular, layered Ansible project to deploy a complete media and home automation stack on a Debian-based server.

## 🏗 Architecture
This project uses a **Layered Deployment** strategy, allowing you to build your homelab step-by-step:

- **Layer 0: Storage** - MergerFS pool and disk formatting.
- **Layer 1: Base** - Hostname, Timezone, and core packages.
- **Layer 2: Networking** - Static IP and network config.
- **Layer 3: Security** - UFW Firewall and SSH hardening.
- **Layer 4: Docker** - Docker Engine, Compose, and LazyDocker.
- **Layer 5: Caddy** - Reverse proxy with automatic HTTPS.
- **Layer 6: Pi-hole** - Network-wide ad blocking.
- **Layer 7: Nextcloud** - Personal cloud and file sync (AIO).
- **Layer 8: Mealie** - Recipe management and meal planning.
- **Layer 9: Jellyfin Stack** - The "Full Arrs" media stack (Jellyfin, Sonarr, Radarr, Prowlarr, qBitTorrent + VPN).

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
