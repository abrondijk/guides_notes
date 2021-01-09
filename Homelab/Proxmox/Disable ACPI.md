# Disable the ACPI bios errors at boot of proxmox

## Details

Not entirely sure why these errors show but this is how to disable ACPI all together to surpress those errors.
The errors themselves are not harmful, but they **might** stop the host from booting.

## Setup

1. Edit the grub file by changing `GRUB_CMDLINE_LINUX_DEFAULT="quiet"` to `GRUB_CMDLINE_LINUX_DEFAULT="quiet acpi=off"` in `/etc/default/grub`
2. Update grub: `update-grub`
3. Reboot

### Links

- <https://forum.proxmox.com/threads/acpi-error-stops-booting-with-latest-proxmox-update.80429/>
