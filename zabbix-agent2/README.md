# Zabbix Agent 2 Ansible Playbook

This Ansible playbook automates the installation, configuration, and secure certificate-based authentication of Zabbix Agent 2 across your infrastructure.

## Features

- Installs Zabbix Agent 2 on supported distributions (Debian/Ubuntu, RedHat/AlmaLinux).
- Configures agent settings dynamically.
- **Automated PKI Management**:
  - Dynamically checks if an agent needs new certificates.
  - Generates Private Keys and Certificate Signing Requests (CSRs) locally.
  - Signs CSRs using a local, Vault-encrypted Certificate Authority (CA).
  - Securely deploys certificates to the remote agent.
  - **Security feature**: Automatically deletes the generated private key from the Ansible control node immediately after deployment.

## Prerequisites

Ensure your Ansible control node has the required dependencies installed:

```bash
# Install required Ansible collection
ansible-galaxy collection install community.crypto

# Install required Python library
pip install cryptography
```

## Setup Instructions

### 1. Inventory (`inventory.yml`)
Add your target machines to the `inventory.yml` file. Example:

```yaml
---
all:
  vars:
    ansible_user: your_ssh_user
    ansible_ssh_private_key_file: /path/to/private/key

almalinux:
  hosts:
    alma-01:
      ansible_host: 10.0.1.1

debian:
  hosts:
    debian-01:
      ansible_host: 10.0.1.2

```

### 2. Variables (`group_vars/all.yaml`)
Update the global variables in `group_vars/all.yaml` to match your environment, specifically the Zabbix Server address:

```yaml
zabbix_agent2:
  mandatory:
    server: zabbix.yourdomain.com
  server_active: zabbix.yourdomain.com
  # ... other configurations
```

### 3. Certificate Authority (CA) & Vault
This playbook relies on a local CA stored in `zabbix-pki/ca/`. The CA's private key password is encrypted using Ansible Vault and stored in `group_vars/all/vault.yaml`.

#### How to create a local CA
If you don't have a CA yet, you can create one manually on your Ansible control node:

```bash
# 1. Create the directory structure
mkdir -p zabbix-pki/ca
chmod 700 zabbix-pki/ca

# 2. Generate the CA private key (you will be prompted for a passphrase)
openssl genrsa -aes256 -out zabbix-pki/ca/ca.key 4096

# 3. Create the self-signed CA certificate
openssl req -x509 -new -nodes -key zabbix-pki/ca/ca.key -sha256 -days 3650 -out zabbix-pki/ca/ca.crt
```

After creating the CA, ensure you add the passphrase to your vault:
```bash
ansible-vault edit group_vars/all/vault.yaml
# Add: ca_passphrase: "your_chosen_passphrase"
```

## Usage

To run the playbook and deploy Zabbix Agent 2 to all hosts in your inventory, simply execute:

```bash
ansible-playbook zabbix-agent2.yaml --ask-vault-pass
```

## Certificate Workflow (Under the Hood)
1. Checks the remote agent for an existing `agent2.crt`.
2. Compares it against the local copy stored in `zabbix-pki/hosts/<hostname>/agent.crt`.
3. If missing or mismatched, it generates a new `.key` and `.csr` locally.
4. Signs the `.csr` with the local CA.
5. Pushes the `ca.crt`, `agent2.crt`, and `agent2.key` to the target machine (`/etc/zabbix/pki/`).
6. Permanently deletes the local `agent.key` from the Ansible server for security.

## Supported tags
1. `repository` - Manage Zabbix official repositories.
2. `install`    - Install the Zabbix Agent 2 package.
3. `configure`  - Handle full agent configuration, including PKI and service state.
4. `pki`        - Specifically manage certificate generation and distribution.

