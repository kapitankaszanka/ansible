# Bootstrap

Bootstrap playbooks and roles used to prepare a fresh machine (or user
environment) with baseline packages, dotfiles, and system settings.

## What this does

Typical tasks include:
- Installing base packages and CLI tools
- Configuring shell / dotfiles (e.g., `.bashrc`, aliases)
- Placing templated config files using Jinja2
- Applying OS-level settings required by the rest of the stack

> Check the playbook and role files in this folder to see the exact tasks.

## Prerequisites

- **Ansible** (recommended: 2.15+)
- SSH access to the target host(s) or running locally
