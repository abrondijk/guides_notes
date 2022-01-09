### Warning, headache ahead. This file is far from finished
Do *NOT* declare an IPV6 Domain entry if you use subdomains, all your shit will break while settings this up. You may do so when you're done.
  
  Hopefully

## NDPPD
  Should add NDP routes automatically but I haven't gotten it to work
 
## Docker daemon config
  Create `/etc/docker/daemon.json` and make it look something like this:
```json
{
        "ipv6": true,
        "fixed-cidr-v6": "2001:db8:ffff:ffff:aad::/80"
}
```
Make sure you allocate at least a /80 subnet or larger to the default docker bridge, smaller subnets don't seem to work for me.
  
## New docker network
  Containers on the default IPv6 bridge can't be assigned a static IPv6 address, so create a new bridge and connect your containers to that instead of the default one.
  
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
_note, this doesn't actually apply them on boot for me, more work needed_


