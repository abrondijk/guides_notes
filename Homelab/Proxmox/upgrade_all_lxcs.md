Example: upgrade all packages in all LXC's:
```sh
#!/bin/bash

for container in $(lxc-ls -1 --running); do
    lxc-attach -n "$container" -- /bin/bash -c "apt-get update && apt-get upgrade -y"
done
```

This way you don't need to open up your SSH port for Ansible.
A disadvantage is that this only runs in one container at a time so it's quite slow.
To run commands in parrallel you could use this:

```sh
for container in $(lxc-ls -1 --running); do
    lxc-attach -n "$container" -- /bin/bash -c "apt-get update && apt-get upgrade -y" &
done

wait
```

This does make it more difficult to debug if anything goes wrong though, and it's still not a full replacement for Ansible.

To upgrade LXC's that run a now unsupported version of Ubuntu, this guide is great:

[https://web.archive.org/web/20220206003458/https://www.barryodonovan.com/2022/01/31/upgrading-legacy-versions-of-ubuntu](https://web.archive.org/web/20220206003458/https://www.barryodonovan.com/2022/01/31/upgrading-legacy-versions-of-ubuntu)
