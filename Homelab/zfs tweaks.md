# ZFS tweaks

Most info can be found in the oracle docs.
Some basic, and actually readable info [here](https://www.thegeekdiary.com/zfs-tutorials-creating-zfs-pools-and-file-systems/).

## Some ZFS tweaks/settings

[Source](https://pve.proxmox.com/wiki/ZFS_on_Linux#_limit_zfs_memory_usage)

1. Change ZFS max ram allocation
    - Edit `/etc/modprobe.d/zfs.conf`
    - Insert `options zfs zfs_arc_max=<arc_size>` where the size is in bytes

2. Enable ZFS mail events
    - Make sure the package is installed `apt-get install zfs-zed`
    - Edit `/etc/zfs/zed.d/zed.rc`
    - Uncomment `ZED_EMAIL_ADDR="root"`

3. Setup a ZFS Scrub cronjob
    - `crontab -e`, insert `0 2 * * 1 /sbin/zpool scrub <poolname>`
    - Enable email notifications `MAILTO="<emailaddr>"`

## ZFS filesystems

Can be useful to split a pool in different sections and reserve space for said sections.
Remember that filesystems are **not** volumes and cannot be snapshot.

### Basic ZFS filesystem commands

```bash

#Create the action filesystem
zfs create <poolname>/<filesystemname>

#Changing filesystem settings
zfs set quota=500m <poolname>/<filesystemname>
zfs set reservation=200m <poolname>/<filesystemname>

```
