# PCIe Passthrough

## Details

- Proxmox VE 6.3-2

### UEFI Settings

- Enable IOMMU
  - VT-D on intel platforms

### Proxmox Setup

1. Update GRUB
    - Edit `/etc/defulat/grub`
    - Replace `GRUB_CMDLINE_LINUX_DEFAULT="quiet"` with `GRUB_CMDLINE_LINUX_DEFAULT="quiet intel_iommu=on"`
2. Add kernel modules
    - Edit `/etc/modules`
    - Append the file with:

    ```config
    vfio
    vfio_iommu_typ1
    vfio_pci
    vfio_virqfd
    ```

3. Reboot

#### Links

- <https://youtu.be/GoZaMgEgrHw?t=378>
