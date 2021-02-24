# Telegraf exporting proxmox data to influxdb

## Prerequisites

### Influxdb installation

Easily setup with docker though that it is not a requirement. It is recommended to go with influxdb 2.0.
If both `influxdb` and `telegraf` are setup on docker, it's best to create a `docker network`.

```bash
docker run -d --name=InfluxDB \
-p 8086:8086 -v \
/[ath/to/persistent/data]:/root/.influxdbv2 \
quay.io/influxdb/influxdb:<latest_version>
```

To complete the setup, simply go to `http://<ipofthehost>:8086` and follow the setup.
After the setup is complete, be sure to create an access token for the bucket you've created.

## Proxmox and telegraf

There are 2 ways to go about this. You could either install telegraf on the host itself, or you could run telegraf on a container (docker or otherwise) and have telegraf collect the data over udp/tcp.

The latter can be done with:

```bash
docker run -d --name=Telegraf \
-v /dockerdata/telegraf:/etc/telegraf \
telegraf
```

But the (imo) best way to go about it, is by running it on the host. This allows to gather information that would otherwise not be available, such as `SMART` disk data, nvidia, memory usage, cpu usage, etc.

### Installation

```bash
# Before adding Influx repository, run this so that apt will be able to read the repository.
sudo apt-get update && sudo apt-get install apt-transport-https

# Add the InfluxData key
wget -qO- https://repos.influxdata.com/influxdb.key | sudo apt-key add -
source /etc/os-release
test $VERSION_ID = "7" && echo "deb https://repos.influxdata.com/debian wheezy stable" | sudo tee /etc/apt/sources.list.d/influxdb.list
test $VERSION_ID = "8" && echo "deb https://repos.influxdata.com/debian jessie stable" | sudo tee /etc/apt/sources.list.d/influxdb.list
test $VERSION_ID = "9" && echo "deb https://repos.influxdata.com/debian stretch stable" | sudo tee /etc/apt/sources.list.d/influxdb.list
test $VERSION_ID = "10" && echo "deb https://repos.influxdata.com/debian buster stable" | sudo tee /etc/apt/sources.list.d/influxdb.list

# Install telegraf
sudo apt-get update && sudo apt-get install telegraf
sudo systemctl start telegraf
```

This will install telegraf and create a configuration file at `/etc/telegraf/telegraf.conf`.

#### Telegraf confiruation

Add as many inputs as you like. The output to influxdb 2.0 should look like this:

```bash
# Configuration for sending metrics to InfluxDB
[[outputs.influxdb_v2]]
 ## The URLs of the InfluxDB cluster nodes.
 ##
 ## Multiple URLs can be specified for a single cluster, only ONE of the
 ## urls will be written to each interval.
 ##   ex: urls = ["https://us-west-2-1.aws.cloud2.influxdata.com"]
 urls = ["http://<influxdb_ip:8086"]

 ## Token for authentication.
 token = "<token created for the bucket>"

 ## Organization is the name of the organization you wish to write to; must exist.
 organization = "<organization name>"

 ## Destination bucket to write into.
 bucket = "<bucket name>"

 ## Timeout for HTTP messages.
  timeout = "5s"

 insecure_skip_verify = true
 ```

 Important here is that `insecure_skip_verify` is set to `true` if influxdb is available on http or on https with a self signed certificate.

#### Proxmox configuration

Now to create a user and an API token to gather data from the Proxmox API.

- `Datacenter->Permissions->Users`: Add a new user that uses the PAM authentication method.
- `Datacenter->Permissions->API Tokens`: Add a new API Token for the user that was just created. Save the `Token ID` and the `Token secret`.
- `Datacenter->Permissions`: Add a new permission for **both** the API Token **and** the user. The path should be root `/` and the role should be `PVEAuditor`.

#### Proxmox input in the telegraf configuration

The proxmox part of the telegraf config should look like this:

```bash
# Provides metrics from Proxmox nodes (Proxmox Virtual Environment > 6.2).
[[inputs.proxmox]]
  ## API connection configuration. The API token was introduced in Proxmox v6.2. Required permissions for user and toke$  base_url = "https://localhost:8006/api2/json"
  api_token = "<user>@pam!<tokenID>=<token_secret>"

  insecure_skip_verify = true

  # HTTP response timeout (default: 5s)
  response_timeout = "5s"
```

It's very important that the `api_token` look like that, or it will give you generic and useless JSON errors.

## Misc

To enable the `hddtemp` module, the daemon has to be enabled manually.

Edit `etc/default/hddtemp` to say `RUN_DAEMON="true"`.