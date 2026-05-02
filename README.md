# Ansible
Some automation playbooks

## Bootstrap Linux Machines Ansible Playbook

This Ansible playbook automates the initial configuration and "hardening" of fresh Linux installations (Debian/Ubuntu and RedHat/AlmaLinux). It sets up a dedicated automation user, installs essential packages, configures time synchronization (Chrony), DNS (systemd-resolved), and adds internal Private CA certificates.

### Features

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

## Zabbix Agent 2 Ansible Playbook

This Ansible playbook automates the installation, configuration, and secure certificate-based authentication of Zabbix Agent 2 across your infrastructure.

### Features

- Installs Zabbix Agent 2 on supported distributions (Debian/Ubuntu, RedHat/AlmaLinux).
- Configures agent settings dynamically.
- **Automated PKI Management**:
  - Dynamically checks if an agent needs new certificates.
  - Generates Private Keys and Certificate Signing Requests (CSRs) locally.
  - Signs CSRs using a local, Vault-encrypted Certificate Authority (CA).
  - Securely deploys certificates to the remote agent.
  - **Security feature**: Automatically deletes the generated private key from the Ansible control node immediately after deployment.
