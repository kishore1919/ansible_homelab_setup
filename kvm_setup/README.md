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
VMs: `{{ kvm_vm_name }}-01.qcow2` etc. Login: ubuntu/ubuntu (SSH/console). Hostnames match names.

## Provision from sd-cloud.yml (cncloud::custom::vm_data)

You can use the `sd-cloud.yml` structure to provision multiple VMs defined under the `cncloud::custom::vm_data` mapping. Copy or add the `sd-cloud.yml` variables as a var file and include it at runtime:

```bash
ansible-playbook -i hosts main.yml -e @sd-cloud.yml
```

This role will parse the `cncloud::custom::vm_data` structure and create a VM per `trafficgen2`, `trafficgen3` entries found. Make sure to install `gsutil` if your images are provided as `gs://...` URLs, or update image paths to `http(s)`.

### Run once behavior

This role honors the global homelab 'run once' marker files. A marker file is created in `/var/lib/ansible-homelab-setup/markers/kvm_setup` after the role successfully runs; future playbook runs will skip this role unless you pass `force_reapply=true` or remove the marker file.

```bash
ansible-playbook -i inventory.ini main.yml -e force_reapply=true
# or to reset the marker on a host:
ssh root@<host> rm /var/lib/ansible-homelab-setup/markers/kvm_setup
```

## License
MIT-0
