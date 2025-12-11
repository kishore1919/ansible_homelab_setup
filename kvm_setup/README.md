# KVM VM Provisioning with Ansible

This directory contains Ansible tasks for automating KVM virtual machine provisioning based on a custom-underlay YAML configuration.

## Overview

The system downloads VM images (supporting Google Cloud Storage URLs), converts them to qcow2 format, creates per-VM overlay disks for efficient storage, and provisions VMs with configurable networking and NAT rules.

## Directory Structure

```
kvm_setup/
├── defaults/
│   └── main.yml
├── tasks/
│   ├── main.yml
│   ├── build_vm_list.yml
│   ├── create_vm_single.yml
│   ├── configure_nat.yml
│   └── configure_connections.yml
└── README.md
```

## Requirements

- `libvirt-daemon-system`, `qemu-utils`, `virtinst`
- `community.libvirt` collection
- `become: yes`

## Dependencies

- `gsutil` (if using Google Cloud Storage image sources)
- `iptables` (for NAT rules)

## Example Playbook

```yaml
- hosts: kvm_hosts
  become: yes
  tasks:
    - name: "Include KVM setup"
      include_role:
        name: kvm_setup
```

## Configuration

This role uses the `cncloud__custom__vm_data` variable structure to define VMs. You can provide this structure via an external variable file:

```bash
ansible-playbook -i inventory.ini main.yml -e @sd-cloud.yml
```

The role will parse the `cncloud__custom__vm_data` structure and create a VM per entry found.

## Tasks

- `build_vm_list.yml`: Builds the list of VMs to create from custom-underlay data
- `create_vm_single.yml`: Creates a single VM from definition
- `configure_nat.yml`: Configures NAT rules for VMs (if defined)
- `configure_connections.yml`: Configures network connections between VMs (if defined)

## Variables

The role currently uses default settings from `defaults/main.yml` (which may need to be updated for KVM-specific defaults).

## Important Notes

- VM images can be sourced from local files, HTTP/HTTPS URLs, or Google Cloud Storage (gs://) URLs
- Base images are converted to qcow2 format and per-VM overlay disks are created for efficient storage
- VMs are created with Ubuntu 22.04 OS info by default
- NAT rules are configured using iptables and saved to `/etc/iptables.rules`
- Network connections rely on libvirt network configuration

## Issues to Address

- The `defaults/main.yml` file currently contains basic_setup settings and should be updated with KVM-specific defaults
- The `vars/main.yml` file referenced in the old README does not exist
- The `configure_connections.yml` task may have incorrect syntax for DHCP host configuration
- The NAT rules in `configure_nat.yml` may have incorrect destination format

## License

MIT-0
