# Nvidia driver setup for LXC use with Proxmox

## Details

- Proxmox VE 7.2-4

### Proxmox Host setup

1. Add backports to your `/etc/apt/sources.list`

```sh
echo "deb http://deb.debian.org/debian buster main contrib non-free" | tee /etc/apt/sources.list
```

2. Install nvidia drivers:
```sh
curl -s -L https://nvidia.github.io/nvidia-container-runtime/gpgkey | apt-key add -
curl -s -L https://nvidia.github.io/nvidia-container-runtime/$(. /etc/os-release;echo $ID$VERSION_ID)/nvidia-container-runtime.list | tee /etc/apt/sources.list.d/nvidia-container-runtime.list
apt update
apt -y install nvidia-container-runtime nvidia-smi cuda # This may take a while
```
A reboot is recommended, but not necessary; you can check your driver installation and version by using `nvidia-smi`

### LXC setup
1. Add the following line to your `/etc/pve/lxc/<container-id>.conf`
```
lxc.hook.pre-start: sh -c '[ ! -f /dev/nvidia-uvm ] && /usr/bin/nvidia-modprobe -c0 -u'
lxc.environment: NVIDIA_VISIBLE_DEVICES=all
lxc.environment: NVIDIA_DRIVER_CAPABILITIES=all
lxc.hook.mount: /usr/share/lxc/hooks/nvidia
```
And that's it, after rebooting your LXC you should now be able to use `nvidia-smi` in there as well.

### Tips

- In case of failed install, remove any nvidia packages by following [this](https://docs.nvidia.com/cuda/cuda-installation-guide-linux/index.html#removing-cuda-tk-and-driver)
- Guide [here](https://medium.com/@MARatsimbazafy/journey-to-deep-learning-nvidia-gpu-passthrough-to-lxc-container-97d0bc474957)
