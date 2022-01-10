# Setting up docker for use with IPv6 on Ubuntu
Even though docker _should_ technically support IPv6 with the same ease as it does IPv4, there are some tweaks that I needed to do to actually get it working.

## NDPPD
  Should add NDP routes automatically but I haven't gotten it to work
 
## Docker daemon config
  Create `/etc/docker/daemon.json` and make it look something like this:
```json
{
        "ipv6": true,
        "fixed-cidr-v6": "2001:db8:0:0:aad::/80"
}
```
Make sure you allocate at least a /80 subnet or larger to the default docker bridge, smaller subnets don't seem to work for me.
I found [https://www.vultr.com/resources/subnet-calculator-ipv6/](https://www.vultr.com/resources/subnet-calculator-ipv6/) to be a handy tool to calculate the correct CIDR notation.
 
## New docker network
  Containers on the default IPv6 bridge can't be assigned a static IPv6 address, so create a new bridge and connect your containers to that instead of the default one.
  
  ```sh
  docker network create --ipv6 --subnet=2001:db8:ffff:ffff:aaf::/80 bridge6
  ```
  
  And yes, setting the subnet in `daemon.json` is still necessary to make sure docker boots with IPv6 enabled at all.
 
## Sysctl settings


Add the following to `/etc/sysctl.conf`:

```
# change `interfaceName` to your docker host NIC name (the ndp one is the most important it seems)
net.ipv6.conf.interfaceName.proxy_ndp=1
net.ipv6.conf.interfaceName.accept_ra=2
net.ipv6.conf.default.forwarding=1
net.ipv6.conf.all.forwarding=1
```
Add this to the root users' crontab to actually apply the settings in there:
```
@reboot sysctl --system
```
Not sure if that's actually needed for every system but for mine it was.

You might also want to cal `sysctl --system` yourself to apply the settings right now.

Now you just have to setup an IPv6 compatible container and give it a go. Do note that Docker port mapping doesn't apply to IPv6. Whatever port the container uses internally is what will be published on IPv6. This is because NAT isn't used for IPv6.
