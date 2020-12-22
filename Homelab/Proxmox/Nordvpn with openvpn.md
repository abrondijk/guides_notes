# Nordvpn with openvpn

## Details

- Proxmox VE 6.3-2

## Prerequisites

- Container has to be priviliged (or mapped gid/uid)
- Allow for container create tun interface
  - Follow [this](https://www.hungred.com/how-to/setup-openvpn-on-proxmox-lxc/) guide:
    Enter the following in `/etc/pve/lxc/<lxc_id>.conf`:

    ```conf
    lxc.cgroup.devices.allow = c 10:200 rwm
    lxc.hook.autodev = sh -c "modprobe tun; cd ${LXC_ROOTFS_MOUNT}/dev; mkdir net; mknod net/tun c 10 200; chmod 0666 net/tun"
    ```

## Proxmox Container setup

1. Install `python3 pip`
    - `apt install python3-pip`
2. Install the [openpyn](https://github.com/jotyGill/openpyn-nordvpn) python package
3. Create symlink for the commands from `/usr/local/bin/` to `/bin/`

    ```bash
    ln -s /usr/local/bin/openpyn /bin/openpyn
    ln -s /usr/local/bin/openpyn-management /bin/openpyn-management
    ln -s /usr/local/bin/coloredlogs /bin/coloredlogs
    ln -s /usr/local/bin/humanfriendly /bin/humanfriendly
    ```

4. Initialize the `openpyn` package with `openpyn --init`
    - **Hint** The password is the actual password of nordvpn, not an application password
5. Test the `openpyn` package with `openypn nl --tcp`
6. **(Optional)** To enable the vpn on startup, create a `systemd` service
    - Create a new service file in `/etc/systemd/system/` with the following contents:

        ```service
        [Unit]
        Description=NordVPN connection manager
        Wants=network-online.target
        After=network-online.target
        After=multi-user.target

        [Service]
        Type=simple
        User=root
        WorkingDirectory=/usr/local/lib/python3.7/dist-packages/    openpyn/
        ExecStartPre=/bin/sleep 5
        ExecStart=/bin/openpyn nl --tcp
        ExecStop=/bin/openpyn --kill
        StandardOutput=syslog
        StandardError=syslog

        [Install]
        WantedBy=multi-user.target
        ```

7. **(Optional)** To allow the local network to access the container, add a new ip rule and apply it after the network interface comes online
    - Add the following to `/etc/network/interfaces`:

        ```bash
        ip rule add from <x.x.x.x> table 128
        ip route add table 128 to y.y.y.y/yy dev eth0
        ip route add table 128 default via z.z.z.z
        ```

        Where `x.x.x.x` is the container IP, `y.y.y.y/yy` is the    subnet and `z.z.z.z` is the default gateway

### Links

- [openpyn](https://github.com/jotyGill/openpyn-nordvpn)
- [Allow local network to access the container](https://serverfault.com/questions/659955/allowing-ssh-on-a-server-with-an-active-openvpn-client)
