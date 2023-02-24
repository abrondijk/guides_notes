# Ubuntu Server

Generic notes about ubuntu server

## User setup

```bash
# Create new user, add to sudoers and delete original
sudo adduser administrator
sudo usermod -aG sudo administrator
sudo userdel -r -f ubuntu

# Don't require password on sudo
echo "$USER ALL=(ALL) NOPASSWD: ALL" | sudo tee -a /etc/sudoers

```

## Netplan

```bash
# Netplan example
network:
  ethernets:
    eth0:
      addresses:
      - 10.0.20.63/24
      routes:
      - to: default
        via: 10.0.20.1
      nameservers:
        addresses:
        - 1.1.1.1
        search: []
  version: 2
```

## Cloud-init

### Disable cloud-init
```bash
sudo touch /etc/cloud/cloud-init.disabled
```

### Completely remove cloud-init
```bash
sudo apt-get purge cloud-init
sudo rm -rf /etc/cloud/; sudo rm -rf /var/lib/cloud/
sudo reboot
```

## Remove snap
```bash
sudo apt purge snapd
```

## Raspberry pi specific
```bash
# Disable wifi/bluetooth
# Add the following two lines in /boot/firmware/config.txt
dtoverlay=disable-wifi
dtoverlay=disable-bt

# Overclock, again in /boot/firmware/config.txt
over_voltage=4
arm_freq=1850

```

## Packages
```bash

sudo apt install lm-sensors ethtool net-tools keepalived bash curl open-iscsi nfs-common powertop

```

## Pihole on latest Ubuntu
https://github.com/pi-hole/pi-hole/issues/4693

```bash
sudo bash
curl -sSL https://install.pi-hole.net | PIHOLE_SKIP_OS_CHECK=true bash
```