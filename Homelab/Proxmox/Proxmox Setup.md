# Proxmox VE Setup

All things initially done before setting up any containers/VMs

1. Enable Proxmox updates without subscription
    1. Edit the `/etc/apt/sources.list` file
    2. Add `deb http://download.proxmox.com/debian/pve buster pve-no-subscription` to the file

2. Setup email notifications
    1. Edit `/etc/postfix/main.cf`
    2. 
2. Setup a ZFS Scrub cronjob
    1. 