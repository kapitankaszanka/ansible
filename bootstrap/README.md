# Bootstrap Linux Machines Ansible Playbook

This Ansible playbook automates the initial configuration and "hardening" of fresh Linux installations (Debian/Ubuntu and RedHat/AlmaLinux). It sets up a dedicated automation user, installs essential packages, configures time synchronization (Chrony), DNS (systemd-resolved), and adds internal Private CA certificates.

## Features

- **Base System Setup**:
  - Creates a dedicated automation user with passwordless sudo access.
  - Installs a curated list of essential packages (vim, tmux, htop, git, etc.).
  - Configures user environments (bashrc, vimrc, tmux.conf).
- **Service Configuration**:
  - Configures **Chrony** for reliable time synchronization.
  - Configures **systemd-resolved** for DNS management (primarily Debian).
- **Security & Trust**:
  - Deploys and trusts internal **Private CA certificates** (RSA and EC).
- **System Maintenance**:
  - Includes a standalone playbook for full system upgrades.

## Prerequisites

Ensure your Ansible control node can connect to the target machines via SSH. The first run typically requires `root` access or a user with `sudo` privileges.

## Setup Instructions

### 1. Variables (`group_vars/all.yaml`)
Update the global variables to match your environment:

- `automation_user`: The name of the user Ansible will use for future tasks.
- `automation_ssh_keys`: List of public SSH keys for the automation user.
- `admin_users`: List of administrators to be created on the system.
- `dns_servers` & `ntp_servers`: Your infrastructure's core services.

### 2. OS-Specific Packages
Adjust the package lists in `group_vars/debian_like.yaml` and `group_vars/redhat_like.yaml` as needed.

### 3. Private CA Certificates
Place your root CA certificates in `roles/add_private_ca/files/`. The role automatically loops over all files ending with `*.crt` and installs them.

## Usage

### Initial Bootstrap
To run the full bootstrap process:

```bash
ansible-playbook site.yaml -u root --ask-pass
```
*Note: Once bootstrapped, you can use the `automation_user`. You can also run specific parts using tags, e.g., `--tags add_private_ca`.*

### System Upgrade
To upgrade all packages on the target systems:

```bash
ansible-playbook upgrade_all.yaml
```

## Roles Overview

| Role | Description |
|------|-------------|
| `common_bootstrap` | Creates automation user and sets up sudoers. |
| `services_configuration` | Configures Chrony and systemd-resolved. |
| `packages_installation` | Installs base packages from OS-specific lists. |
| `customization` | Deploys dotfiles and creates admin users. |
| `add_private_ca` | Adds internal CAs to the system trust store. Supports `add_private_ca` tag. |
| `upgrade_all_packages` | Performs a full system upgrade (`apt upgrade` or `dnf upgrade`). |
