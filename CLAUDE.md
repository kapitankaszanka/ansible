# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository layout

Two independent Ansible projects, each self-contained (own `group_vars`, `roles`, playbooks, README). Run commands from inside the project directory — paths like `{{ playbook_dir }}/zabbix-pki` and role lookup depend on it. There is no `ansible.cfg` and no inventory in git (inventory is gitignored / supplied locally with `-i`).

- `bootstrap/` — first-run setup of fresh Debian/Ubuntu and RedHat/AlmaLinux hosts.
- `zabbix-agent2/` — install/configure Zabbix Agent 2 with certificate (TLS) auth.

## Commands

```bash
# bootstrap (first run as root, afterwards as automation_user)
cd bootstrap
ansible-playbook -i <inventory> site.yaml -u root --ask-pass
ansible-playbook -i <inventory> upgrade_all.yaml
ansible-playbook -i <inventory> site.yaml --tags add_private_ca   # run one part

# zabbix-agent2 (needs: ansible-galaxy collection install community.crypto; pip install cryptography)
cd zabbix-agent2
ansible-playbook -i <inventory> playbook.yaml --ask-vault-pass
ansible-playbook -i <inventory> playbook.yaml --tags pki --ask-vault-pass

# validation (no linters or tests are configured; ansible-lint/yamllint are not installed)
ansible-playbook -i <inventory> <playbook> --syntax-check
ansible-playbook -i <inventory> <playbook> --check --diff --limit <host>
```

## Conventions

- Every task carries `tags`; READMEs document the tag set per project — keep them in sync when adding tasks. Bootstrap tags: `bootstrap, update, sudo, user, customization, package, dns/resolved, ntp/chrony, add_private_ca, upgrade`. Zabbix tags: `repository, install, configure, pki` (`remove` for the `remove_config` role, which is commented out in `playbook.yaml`).
- OS branching is done inline with `when: ansible_os_family == "Debian"|"RedHat"` and with ternary defaults (e.g. `superuser_group`, `chrony_service_name`, `root_ca_cert_path`) rather than per-OS vars files. Zabbix roles additionally fail early if `ansible_distribution` is not in `supported_distros` (Debian, Ubuntu, RedHat, AlmaLinux) — this list is duplicated in each role's `defaults/main.yaml`.
- Use FQCN module names (`ansible.builtin.*`, `ansible.posix.*`, `community.crypto.*`).
- File extensions are mostly `.yaml`; a few zabbix vars files use `.yml`.

## Architecture notes

### bootstrap
- `site.yaml` applies roles in order: `common_bootstrap` (automation user + validated NOPASSWD sudoers drop-in) → `services_configuration` → `packages_installation` → `customization` (admin users + dotfiles from `roles/customization/files/`) → `add_private_ca`.
- Package lists come from `group_vars/debian_like/` and `group_vars/redhat_like/`; a `group_by` pre-task in `site.yaml` assigns hosts to these groups by `ansible_os_family`.
- Tags on `include_tasks` only tag the include itself — pass them via `apply: tags:` too (see `services_configuration/tasks/main.yaml`), or `--tags` won't reach the included tasks.
- `services_configuration` is skipped entirely inside containers (`not_container`); systemd-resolved is configured only on Debian family, chrony on both.
- `add_private_ca` installs every `*.crt` in `roles/add_private_ca/files/` via `with_fileglob` and triggers the trust-store update handler.

### zabbix-agent2
- Roles: `add_repository` (official Zabbix repo; on RedHat excludes `zabbix*` from EPEL) → `install` → `configure`.
- Agent `Hostname` is built in `group_vars/all/main.yml` from `zabbix_agent_suffixes` (per-host in `host_vars/`, default `['linux']`) → e.g. `dns01-linux,dns01-docker`. Config is rendered from `roles/configure/templates/zabbix_agent2.conf.j2` using the nested `zabbix_agent2` dict.
- PKI pipeline in `roles/configure/tasks/main.yaml`, run on the control node via `delegate_to: localhost`:
  - Local CA lives in `zabbix-pki/ca/` (`ca.crt`, passphrase-protected `ca.key`); `zabbix-pki/` is gitignored and must never be committed. `ca_passphrase` comes from the Ansible Vault file `group_vars/all/vault.yaml` (gitignored; template in `vault.yaml.example`).
  - Per host, `zabbix-pki/hosts/<inventory_hostname>/agent.crt` is compared by checksum with the remote `/etc/zabbix/pki/agent2.crt`; on mismatch/missing, a new key+CSR is generated, signed by the CA, pushed, and the local private key is deleted.
  - Hosts in the `zabbix_servers` inventory group get a parallel `server.crt`/`server.key` pipeline that writes alongside (not over) the agent certs.
