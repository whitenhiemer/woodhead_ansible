# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is an Ansible project for managing a homelab infrastructure. The infrastructure consists of multiple host groups running on Proxmox virtualization, including a Kubernetes cluster (named after TMNT characters), monitoring stack, and home automation services.

## Architecture

### Host Groups (inventory.yml)

- **turtles**: Kubernetes cluster nodes (leonardo, donatello, raphael, michealangelo)
  - User: bwoodwar
  - Python: /usr/bin/python3
- **proxmox**: Proxmox virtualization hosts (master, thinkcentre1-3)
  - User: root
  - Python: /usr/bin/python3
- **homeassistant**: Home Assistant server
  - User: root
  - Python: /usr/bin/python3
- **monitoring**: Monitoring infrastructure (graphana, influxdb)
  - User: root
  - Python: /usr/bin/python3

All hosts use SSH key authentication via `~/.ssh/id_ansible`.

### Playbook Organization

All playbooks are located in the `playbooks/` directory as standalone YAML files (no roles). Each playbook is self-contained with its own vars section for configuration.

**Categories:**
- **Kubernetes**: `k8s-install.yml`, `k8s-init.yml`
- **Package Installation**: `install-apache.yml`, `install-vim.yml`, `install-zsh-git.yml`, `install-gnupg.yml`
- **System Management**: `apt-update.yml`, `poweroff.yml`
- **Storage**: `mount-nfs.yml`

## Common Commands

### Running Playbooks

```bash
# Run a playbook on all applicable hosts
ansible-playbook playbooks/<playbook-name>.yml

# Run on specific host(s)
ansible-playbook playbooks/<playbook-name>.yml --limit <hostname>

# Override variables
ansible-playbook playbooks/<playbook-name>.yml -e "variable_name=value"

# Example: Install Kubernetes with specific version
ansible-playbook playbooks/k8s-install.yml -e "kubernetes_version=1.28.0-00"
```

### Testing and Debugging

```bash
# Test connectivity to all hosts
ansible all -m ping

# Test connectivity to specific group
ansible turtles -m ping

# Check playbook syntax
ansible-playbook playbooks/<playbook-name>.yml --syntax-check

# Dry run (check mode)
ansible-playbook playbooks/<playbook-name>.yml --check

# Run with verbose output
ansible-playbook playbooks/<playbook-name>.yml -v    # or -vv, -vvv for more verbosity
```

### Inventory Management

```bash
# List all hosts
ansible-inventory --list

# List hosts in a specific group
ansible-inventory --list | grep -A 10 "turtles"

# Graph inventory structure
ansible-inventory --graph
```

## Configuration Files

- **ansible.cfg**: Global Ansible configuration
  - Sets inventory file location
  - Disables host key checking
  - Uses YAML output callback for readable results
  - Disables retry files

- **inventory.yml**: Defines all hosts, groups, and connection settings
  - Each group has specific vars for authentication and Python interpreter
  - Hosts may be commented out when offline (see leonardo example)

## Creating New Playbooks

When creating new playbooks:

1. Place in `playbooks/` directory
2. Use `.yml` extension
3. Include a descriptive `name` at the top
4. Specify `hosts` (use group names from inventory or `all`)
5. Set `become: yes` if root privileges needed
6. Define configurable parameters in `vars` section with sensible defaults
7. Use fully qualified collection names (e.g., `ansible.builtin.apt`)
8. Include debug tasks at the end to report results

Example structure:
```yaml
---
- name: Descriptive playbook name
  hosts: all  # or specific group
  become: yes
  vars:
    # Configurable variables with defaults
    some_variable: "default_value"

  tasks:
    - name: Clear task description
      ansible.builtin.module:
        parameter: value
```

## Kubernetes-Specific Notes

The Kubernetes installation is a two-step process:
1. `k8s-install.yml` - Installs Kubernetes packages and container runtime (containerd) on all nodes
2. `k8s-init.yml` - Initializes the control plane (run after updating inventory with control_plane group)

Default versions and settings:
- Kubernetes version: 1.29.0-00
- Container runtime: containerd
- Pod network CIDR: 10.244.0.0/16

## Working with This Codebase

- All playbooks target Debian/Ubuntu hosts with apt package manager
- SSH authentication uses key-based auth (no password prompts expected)
- Most playbooks are idempotent and safe to re-run
- Use `--limit` flag to target specific hosts when testing new playbooks
- Check git history for examples of recent changes to playbooks
