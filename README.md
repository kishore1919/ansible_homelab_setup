# Ansible Homelab Setup

Deploys a full homelab on Ubuntu: Docker stacks (SonarQube, n8n) via `basic_setup` role, multiple KVM VMs via `kvm_setup` role.

## Prerequisites
- Ubuntu 22.04+ target host.
- Ansible 2.14+ with `ansible-galaxy collection install community.libvirt community.docker`.
- Root/sudo access (`become: yes`).
- For KVM: `apt install libvirt-daemon-system qemu-utils virtinst bridge-utils` (add to basic_setup if needed).
- Inventory file (e.g. `inventory.ini`) with target host(s).

## Usage
```bash
ansible-playbook -i inventory.ini main.yml
```
- Runs `basic_setup`: installs Docker/cockpit-machines/python3-pip/git, deploys stacks to `/home/compose_file`.
- Runs `kvm_setup`: downloads Ubuntu 22.04 ISO to `/var/lib/libvirt/images`, creates `kvm_vms_counts` VMs (e.g. `ubuntu-server-22.04-1.qcow2`).

## Customization
- Edit `basic_setup/files/*.yml` for stacks (set `N8N_HOST`/`WEBHOOK_URL` to VM IP).
- Override vars in playbook: `roles: - { role: kvm_setup, kvm_vms_counts: 4, vm_ram_mb: 4096 }`.
- Access: Cockpit at `https://<vm-ip>:9090` (machines tab for KVM), SonarQube `http://<vm-ip>:9000`, n8n `http://<vm-ip>:5678`.

## Notes
- KVM images are qcow2 clones from ISO (use `virsh start <name>` or cockpit).
- Idempotent: re-runs skip existing images/VMs.
- Edit `kvm_setup/vars/main.yml` for VM specs.
