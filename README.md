# woodhead_ansible

Ansible playbooks for managing hosts on my network.

## Prerequisites

- Ansible 2.9 or later
- SSH access to target hosts
- sudo/root privileges on target hosts

## Files

- `apt-update.yml` - Playbook to update apt package lists and optionally upgrade packages
- `inventory.yml` - Inventory file defining hosts to manage
- `ansible.cfg` - Ansible configuration settings

## Usage

### 1. Update inventory.yml

Edit `inventory.yml` and replace the example hosts with your actual hostnames/IPs and usernames.

### 2. Run the playbook

**Update apt cache only:**
```bash
ansible-playbook apt-update.yml
```

**Update apt cache AND upgrade all packages:**
```bash
ansible-playbook apt-update.yml -e "upgrade_packages=true"
```

**Run on specific hosts:**
```bash
ansible-playbook apt-update.yml --limit host1
```

**Run on specific groups:**
```bash
ansible-playbook apt-update.yml --limit ubuntu_servers
```

## Testing Connections

Test your connection to hosts without running the playbook:
```bash
ansible all -m ping
```

## Additional Configuration

You can specify SSH keys, sudo passwords, or other settings:
- In `inventory.yml` for per-host settings
- In `ansible.cfg` for global settings
- Via command line: `ansible-playbook apt-update.yml -e "ansible_sudo_pass=your_password"`
