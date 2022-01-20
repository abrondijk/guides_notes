# Nvidia driver setup

## Proxmox Host setup

### On the host:

1. Add overlay to the host kernel modules: `modprobe overlay` and reboot
2. Create a sparsed zvol: `zfs create -s -V <size> <poolname>/<zvolname>`
3. Verify it was created and is sparse: `zfs get volsize,referenced`
4. Format the zvol with preferred filesystem: `mkfs.<fs> /dev/zvol/<poolname>/<zvolname>`
5. Mount it into a temp location to change permissions: 
    
    ```
    mkdir /tmp/zvol_tmp
    mount /dev/zvol/<poolname>/<zvolname> /tmp/zvol_tmp
    chown -R 100000:100000 /tmp/zvol_tmp
    umount /tmp/zvol_tmp
    ```

6. Add the zvol as mountpoint in the lxc conf file

### Links

- [Reddit thread](https://www.reddit.com/r/Proxmox/comments/lsrt28/easy_way_to_run_docker_in_an_unprivileged_lxc_on/)
