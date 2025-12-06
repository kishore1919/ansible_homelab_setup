# Ansible Homelab Setup

An Ansible playbook that deploys a complete homelab infrastructure on Ubuntu, featuring containerized services and KVM virtual machines.

## 🎯 Features

### Basic Setup Role
- **System Preparation**: Updates packages, disables automatic upgrades, installs essential tools
- **Docker Environment**: Installs Docker CE, Docker Compose plugin, and required dependencies
- **Web Management**: Deploys Cockpit web-based management interface
- **Containerized Services**:
  - **SonarQube Community**: Code quality and security analysis platform
  - **n8n**: Workflow automation and integration platform

### KVM Setup Role
- **VM Infrastructure**: Creates multiple Ubuntu 22.04 LTS virtual machines
- **Cloud-init Integration**: Configures VMs with user accounts and SSH access
- **Flexible Configuration**: Customizable VM count, memory, CPU, and disk size

## 📋 Prerequisites

### Target System
- Ubuntu 22.04 LTS or newer
- Root or sudo access required
- Minimum 8GB RAM recommended (for VM creation)
- 50GB+ free disk space

### Ansible Requirements
- Ansible 2.14 or newer
- Required collections:
  ```bash
  ansible-galaxy collection install community.libvirt community.docker
  ```

### Host System Dependencies
- `libvirt-daemon-system`
- `qemu-utils`
- `virtinst`
- `bridge-utils`

**Note**: The `basic_setup` role does not automatically install KVM dependencies. Install them separately before running the KVM setup.

## 🏗️ Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    Target Host (Ubuntu 22.04)               │
├─────────────────────────────────────────────────────────────┤
│  ┌─────────────────┐  ┌─────────────────┐  ┌──────────────┐ │
│  │   Cockpit UI    │  │   SonarQube     │  │     n8n      │ │
│  │   :9090         │  │   :9000         │  │    :5678     │ │
│  └─────────────────┘  └─────────────────┘  └──────────────┘ │
│                           Docker Engine                     │
├─────────────────────────────────────────────────────────────┤
│  ┌─────────────────┐  ┌─────────────────┐  ┌──────────────┐ │
│  │ ubuntu-vm-01    │  │ ubuntu-vm-02    │  │ ubuntu-vm-N  │ │
│  │ 192.168.122.X   │  │ 192.168.122.X   │  │ 192.168.122.X│ │
│  └─────────────────┘  └─────────────────┘  └──────────────┘ │
│                         KVM/libvirt                         │
└─────────────────────────────────────────────────────────────┘
```

## 🚀 Usage

### Basic Usage
```bash
# Run full homelab setup
ansible-playbook -i inventory.ini main.yml
```

### Selective Execution
```bash
# Run only basic setup (Docker + services)
ansible-playbook -i inventory.ini main.yml --tags basic_setup

# Run only KVM setup
ansible-playbook -i inventory.ini main.yml --tags kvm_setup
```

## ⚙️ Configuration

### Inventory Configuration
Edit `inventory.ini` to match your environment:

```ini
[gcp_server]
34.58.59.52 ansible_user=root ansible_password='your-password'
```

### Customization Options

#### Docker Services Configuration
Edit compose files in `basic_setup/files/`:
- `sonarqube-docker-compose.yml`: SonarQube + PostgreSQL configuration
- `n8n-docker-compose.yml`: n8n workflow automation

**Important**: Update n8n configuration with your actual IP/domain:
```yaml
environment:
  - N8N_HOST=your-actual-ip-or-domain
  - WEBHOOK_URL=http://your-actual-ip-or-domain:5678/
```

#### VM Configuration
Override role variables in the playbook:

```yaml
---
- name: Basic Setup Role
  hosts: all
  become: yes
  gather_facts: yes
  roles:
    - basic_setup

- name: KVM Setup Role
  hosts: gcp_server
  become: yes
  gather_facts: no
  collections:
    - community.libvirt
  roles:
    - role: kvm_setup
      kvm_vms_counts: 3          # Number of VMs to create
      vm_ram_mb: 4096            # RAM per VM in MB
      vm_vcpus: 2                # vCPUs per VM
      vm_disk_size: "30G"        # Disk size per VM
      user_password: "custom-password"  # VM user password
```

## 📊 Role Variables

### basic_setup Role
| Variable | Default | Description |
|----------|---------|-------------|
| `basic_setup_compose_dir` | `/home/compose_file` | Docker compose files location |
| `basic_setup_compose_files` | `[sonarqube-docker-compose.yml, n8n-docker-compose.yml]` | Services to deploy |

### kvm_setup Role
| Variable | Default | Description |
|----------|---------|-------------|
| `kvm_vms_counts` | - | Number of VMs to create (required) |
| `vm_ram_mb` | 2048 | Memory per VM in MB |
| `vm_vcpus` | 2 | vCPUs per VM |
| `vm_disk_size` | "20G" | Disk size per VM |
| `kvm_image_url` | Ubuntu cloud image URL | Source image for VMs |
| `network` | "default" | libvirt network name |
| `user_password` | "ubuntu" | VM user password |

## 🌐 Access Information

### Web Interfaces
- **Cockpit Management**: `https://<host-ip>:9090`
  - Monitor system resources, manage containers, access VMs
- **SonarQube**: `http://<host-ip>:9000`
  - Default credentials: admin/admin
- **n8n Workflows**: `http://<host-ip>:5678`
  - Initial setup wizard will guide you through configuration

### Virtual Machines
- **Network**: Default libvirt network (192.168.122.0/24)
- **Access**: `ssh ubuntu@<vm-ip-address>`
- **Default Password**: ubuntu (configurable via `user_password` variable)
- **VM Names**: ubuntu-server-vm-01, ubuntu-server-vm-02, etc.

### VM Management Commands
```bash
# List VMs
virsh list --all

# Start a VM
virsh start ubuntu-server-vm-01

# Stop a VM
virsh shutdown ubuntu-server-vm-01

# Access VM console
virsh console ubuntu-server-vm-01
```

## 🔧 Advanced Configuration

### Custom VM Images
Modify `kvm_setup/vars/main.yml` to use different base images:
```yaml
base_image_name: "focal-server-cloudimg-amd64"  # For Ubuntu 20.04
kvm_image_url: "https://cloud-images.ubuntu.com/focal/current/{{ base_image_name }}.img"
```

### Adding Additional Services
1. Create new compose file in `basic_setup/files/`
2. Add filename to `basic_setup_compose_files` list in `basic_setup/vars/main.yml`
3. Redeploy with basic_setup role

### Network Customization
For custom networking, modify the network variable and ensure libvirt network is properly configured.

## 🛠️ Troubleshooting

### Common Issues

#### Docker Services Not Starting
```bash
# Check service status
docker compose -f /home/compose_file/sonarqube-docker-compose.yml ps

# View logs
docker compose -f /home/compose_file/sonarqube-docker-compose.yml logs
```

#### VM Creation Failures
- Ensure sufficient disk space in `/var/lib/libvirt/images`
- Check libvirt daemon status: `systemctl status libvirtd`
- Verify KVM hardware support: `kvm-ok`

#### n8n Webhook Issues
- Update `N8N_HOST` and `WEBHOOK_URL` in compose file
- Ensure firewall allows inbound connections on port 5678

### Idempotency
Both roles are designed to be idempotent:
- Docker services: Existing containers are not recreated
- VMs: Skips creation if qcow2 files already exist
- Safe to re-run without side effects

## 📁 Project Structure
```
├── main.yml                          # Main playbook
├── inventory.ini                     # Target hosts configuration
├── README.md                         # This file
├── basic_setup/                      # Basic system setup role
│   ├── tasks/
│   │   ├── main.yml                 # Task orchestration
│   │   ├── install_packages.yml     # Package installation
│   │   └── configure_docker.yml     # Docker configuration
│   ├── files/                       # Docker compose files
│   │   ├── sonarqube-docker-compose.yml
│   │   ├── n8n-docker-compose.yml
│   │   └── multi-media-compse.yml
│   ├── defaults/main.yml            # Default variables
│   └── vars/main.yml                # Role-specific variables
└── kvm_setup/                       # KVM VM creation role
    ├── tasks/main.yml               # VM creation tasks
    ├── defaults/main.yml            # Default variables
    ├── vars/main.yml                # VM configuration variables
    └── README.md                    # KVM role documentation
```

---

**Happy Homelabbing! 🏠✨**

## Provision from sd-cloud.yml

You can provision multiple VMs using the `sd-cloud.yml` variables (the file contains `cncloud::custom::vm_data`) using the supplied sample playbook:

```bash
ansible-playbook -i inventory.ini sd-cloud-provision.yml -e @sd-cloud.yml
```

This will parse the `cncloud::custom::vm_data` entries and create the VMs described in the file.

### Run each role only once by default

To prevent re-running roles every time you execute the playbook, this project uses per-role markers stored on the target hosts in `/var/lib/ansible-homelab-setup/markers`. Each role will only run if its marker file is missing. If you want to force re-run a role, pass `force_reapply=true` when running the playbook:

```bash
ansible-playbook -i inventory.ini main.yml -e force_reapply=true
```

Marker files are created automatically after a role runs. To reset a role (force it to run the next time without using `force_reapply`), remove the corresponding file on the target host, e.g.:

```bash
ssh root@<host> rm /var/lib/ansible-homelab-setup/markers/kvm_setup
```

