# Fixing bindmounts `nobody nogroup` in unprivileged LXCs

## Details

This was a pain to set up, tried almost everything until I finally found a guide that almost worked. Added a few things and finally there was the desired result.

The ID's you setup in the proxmox host will be passed onto the LXC, however 10.000 (ten-thousand) will be subtracted from it.

This means that creating a user in the host with PUID 101000 will be presented in the LXC as 1000.

## Setup

1. Add the mount to the LXC with `mpX: /path/on/host/,mp=/path/on/container/`
2. Create a new group on the proxmox host with a gid over 101000: `addgroup --gid <GID (ie."101000")> <GroupName (ie."container-data")>`
3. Create a new user on the proxmox host with e uid over 101000: `useradd --uid <GID (ie."101000")> <UserName (ie."container")>`
4. Add the new user to the just created group: `usermod -aG <GroupName> <User>`
5. Do the following on the proxmox host:
    ```
    chgrp -R <GroupName> <path>
    chown -R <UserName> <path>
    chmod -R 2775 <path>
    ```
    The 2775 mask will make sure that all newly created files and directories, will have the same mask

6. ***Optional:*** Allow for ACL to be used on ZFS: `zfs set acltype=posixacl pool_name`
7. Do the following on the proxmox host:
    ```
    setfacl -Rm g:<GID>:rwx,d:g:<GID>:rwx <path>
    setfacl -Rm u:<UID>:rwx,d:u:<UID>:rwx <path>
    ```
    _note, `apt-get install acl` to get the setfacl command_
8. Now finally inside the LXC, check whether the bindmount exists and is owned by a uid and gid other than 65536: `ls -ln`
9. Add a user with the same uid and gid as the bindmount: `useradd --uid <UID> <username>`

By now you should be able to read and write to the directory.
### Links

- <https://gist.github.com/ajmassi/e6862294d114467b46f9b7f073921352/>
- <https://pve.proxmox.com/wiki/Unprivileged_LXC_containers/>
