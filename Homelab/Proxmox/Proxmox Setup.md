# Proxmox VE Setup

All things initially done before setting up any containers/VMs

1. Add ssh key to the server:
    - `ssh-copy-id -i <public key> <user>@<server>`

2. Enable Proxmox updates without subscription
    1. Edit the `/etc/apt/sources.list` file
    2. Add `deb http://download.proxmox.com/debian/pve buster pve-no-subscription` to the file
    3. Remove the enterprise package list by deleting `/etc/apt/sources.list.d/pve-enterprise.list` or by uncommenting the only line in that file

3. `apt update && apt upgrade -y`

4. Enable Proxmox dark theme
    - `wget https://raw.githubusercontent.com/Weilbyte/PVEDiscordDark/master/PVEDiscordDark.py`
    - `python3 PVEDiscordDark.py`

5. Disable the notification for no valid subscription in the proxmox gui
    - `sed -Ezi.bak "s/(Ext.Msg.show\(\{\s+title: gettext\('No valid sub)/void\(\{ \/\/\1/g" /usr/share/javascript/proxmox-widget-toolkit/proxmoxlib.js && systemctl restart pveproxy.service`
    - [Source](https://johnscs.com/remove-proxmox51-subscription-notice/)

6. Setup email notifications
    - See `Enable Mail Notifications.md`
