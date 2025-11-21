# KVM Setup Role

Clones Ubuntu 22.04 cloud image to create `kvm_vms_counts` KVM VMs in default pool.

## Requirements
- libvirt-daemon-system, qemu-utils, virtinst.
- `community.libvirt` collection.
- `become: yes`.

## Role Variables
From `vars/main.yml` (high priority, override in playbook):
- `kvm_image_url`: ISO download URL.
- `kvm_image_pool`: `/var/lib/libvirt/images`.
- `base_image_name`: `jammy-server-cloudimg-amd64`.
- `kvm_image_checksum`: SHA256 for image (update from SHA256SUMS).
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

## Notes
VMs: `{{ kvm_vm_name }}-01.qcow2` etc. Login: ubuntu/ubuntu (SSH/console). Hostnames match names. Password auth enabled via cloud-init.

## License
MIT-0
