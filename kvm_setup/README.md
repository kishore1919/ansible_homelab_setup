# KVM Setup Role

Creates multiple KVM VMs from Ubuntu ISO in `/var/lib/libvirt/images`.

## Requirements
- libvirt-daemon-system, qemu-utils, virtinst.
- `community.libvirt` collection.
- `become: yes`.

## Role Variables
From `vars/main.yml` (high priority, override in playbook):
- `kvm_image_url`: ISO download URL.
- `kvm_image_pool`: `/var/lib/libvirt/images`.
- `base_image_name`: `ubuntu-server-22.04` (VM prefix).
- `kvm_vms_counts`: Number of VMs (1-10 typical).
- `vm_ram_mb`: 2048, `vm_vcpus`: 2, `vm_disk_size`: "20G".
- `os_variant`: "ubuntu22.04", `network`: "default".

Defaults in `defaults/main.yml`.

## Dependencies
- None.

## Example Playbook
```yaml
- hosts: kvm_hosts
  become: yes
  roles:
    - role: kvm_setup
      kvm_vms_counts: 3
      vm_ram_mb: 4096
```

## License
MIT-0
