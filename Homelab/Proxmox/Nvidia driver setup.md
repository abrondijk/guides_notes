# Nvidia driver setup

## Details

- Proxmox VE 6.3-2

### Proxmox Host setup

1. Update header packeges `apt install pve-headers`
2. Remove and disable any previous nouveau installations before installing nvidia drivers
    - <https://docs.nvidia.com/cuda/cuda-installation-guide-linux/index.html#runfile-nouveau-debian>
3. Follow the installation instructions [here](https://docs.nvidia.com/cuda/cuda-installation-guide-linux/index.html#pre-installation-actions)
4. Verify that the installation was sucessful by running `nvidia-smi`
5. Reboot

### Get the GPU working in an LXC

1. Follow these 2 guides:
    - <https://forums.plex.tv/t/plex-hw-acceleration-in-lxc-container-anyone-with-success/219289/35>
    - <https://medium.com/@MARatsimbazafy/journey-to-deep-learning-nvidia-gpu-passthrough-to-lxc-container-97d0bc474957>

### Tips

- In case of failed install, remove any nvidia packages by following [this](https://docs.nvidia.com/cuda/cuda-installation-guide-linux/index.html#removing-cuda-tk-and-driver)
- Guide [here](https://medium.com/@MARatsimbazafy/journey-to-deep-learning-nvidia-gpu-passthrough-to-lxc-container-97d0bc474957)
