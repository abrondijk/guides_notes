# General Notes

These are just random notes

## Manually mount zfs volumes

For whatever reason, zfs subvolumes will not be mounted automatically, resulting in containers failing to boot, without any helpfull error.

This might be fixed in newers proxmox builds.

Solution:

1. Start the container once (this will fail, but it is necessary)
2. Empty the subvolume (be careful when editing this command) `rm -rf <path_to_subvolumes>/subvol-<lxcid>-disk-<diskid>/dev/`
3. Manually mount the subvolume `zfs mount <path_to_subvolumes>/subvol-<lxcid>-disk-<diskid>`
4. Start the container `pct start <lxcid>`
