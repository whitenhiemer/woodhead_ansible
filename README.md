# woodhead_ansible

Ansible playbooks for managing hosts on my network.

## Prerequisites

- Ansible 2.9 or later
- SSH access to target hosts
- sudo/root privileges on target hosts
- Ubuntu/Debian hosts (for Kubernetes installation)

## Files

- `k8s-install.yml` - Playbook to install Kubernetes with apt
- `k8s-init.yml` - Playbook to initialize Kubernetes cluster
- `inventory.yml` - Inventory file defining hosts to manage
- `ansible.cfg` - Ansible configuration settings

## Kubernetes Installation

### 1. Update inventory.yml

Edit `inventory.yml` and replace the example hosts with your actual hostnames/IPs and usernames.

### 2. Install Kubernetes packages

**Install on all hosts:**
```bash
ansible-playbook k8s-install.yml
```

**Install with specific Kubernetes version:**
```bash
ansible-playbook k8s-install.yml -e "kubernetes_version=1.28.0-00"
```

**Install on specific hosts:**
```bash
ansible-playbook k8s-install.yml --limit leonardo
```

### 3. Initialize Kubernetes cluster

**First, update inventory.yml to define control plane:**
```yaml
turtles:
  hosts:
    leonardo:
      ansible_host: 192.168.86.238
    donatello: null
    raphael: null
    michealangelo: null
  children:
    control_plane:
      hosts:
        - leonardo
    workers:
      hosts:
        - donatello
        - raphael
        - michealangelo
```

**Initialize the cluster:**
```bash
ansible-playbook k8s-init.yml
```

**Join worker nodes:**
Use the join command displayed after cluster initialization.

## Testing Connections

Test your connection to hosts without running the playbook:
```bash
ansible all -m ping
```

## Additional Configuration

You can specify SSH keys, sudo passwords, or other settings:
- In `inventory.yml` for per-host settings
- In `ansible.cfg` for global settings
- Via command line: `ansible-playbook k8s-install.yml -e "ansible_sudo_pass=your_password"`

## Kubernetes Variables

- `kubernetes_version`: Kubernetes version to install (default: "1.29.0-00")
- `container_runtime`: Container runtime to use (default: "containerd")
- `pod_network_cidr`: Pod network CIDR for cluster initialization (default: "10.244.0.0/16")
