# How to create cloudfare tunnels for services
## Notes
 - It might be a better idea to use a generic `/etc/.cloudflared/config.yml`
   - In that case using `cloudflared service install` is easier than setting up your own systemd service
   - Adding more services is as simple as adding another `-hostname` entry to the `config.yaml` and restarting your service
 - Updating is done using `cloudflared update`
 - Nextcloud relies on an array of trusted domain names [https://help.nextcloud.com/t/howto-add-a-new-trusted-domain/26]()
## Setup
First, download `cloudflared` from 
 - [https://developers.cloudflare.com/cloudflare-one/connections/connect-apps/install-and-setup/installation]().

"Install" it by moving the downloaded binary: 
 ```
 sudo mv cloudflared-linux-amd64 /usr/local/bin/cloudflared
 ``` 

and adding execute permissions:
 ```
 sudo chmod +x /usr/local/bin/cloudflared
 ```


Create a new user to run cloudflared, safety first, kids:
 ```
 sudo useradd cloudflared -md /home/cloudflared
 ```
 ```
 sudo passwd cloudflared
 ```

Login to your new user, and we can begin setting up the actual tunnel.
```
su cloudflared
```

This step only needs to be done once for all of your tunnels:
```
cloudflared login
```
Simply follow the promts and it will generate a cert.perm that contains a login certificate

## Standard HTTP Tunnel setup
From here you can create a tunnel, I will be making one for nextcloud:
```
cloudflare tunnel create nextcloud
```
This will give you a json named by a UUID, copy its name because you will need it in a followup step.


You can verify the tunnels' existance if you'd like with this command:
```
cloudflared tunnel list
```

Add the DNS route to cloudflare using the following command:
```
cloudflared tunnel route dns nextcloud next.abchost.nl
```

Add a `~/.cloudfared/nextcloud.yaml` to contain something like the following, adjust settings where neccesary:
``` 
tunnel: JSON-UUID HERE WITHOUT .json extension
credentials-file: /home/cloudflared/.cloudflared/JSON-COPIED-IN-PREVIOUS-STEP-HERE.json

ingress:
  - hostname: next.abchost.nl
    service: http://192.168.2.217:80
    originRequest:
      connectTimeout: 60s
      originServerName: next.abchost.nl
      noTLSVerify: true
  - service: http_status:404

warp-routing:
  enabled: true 
```

Verify your config file:
```
cloudflared tunnel --config .cloudflared/nextcloud.yaml ingress validate
```
  
From here you can actually test your tunnel, do note that it is important that the subdomain you set in the `.yaml` does _not_ already exist on your domain.
```
cloudflared tunnel --config .cloudflared/nextcloud.yaml run nextcloud
```
Once verified that it's working, you can `ctrl+c` it because running this in a terminal that you need to keep open is not very practical.

Therefore we will be adding a simple `systemd` file to run it in the background.

Exit your `cloudfared` user and we can go right ahead by creating `/etc/systemd/system/tunnel-nextcloud.service` and giving it the following contents:
```
# /lib/systemd/system/tunnel-nextcloud.service
[Unit]
Description=cloudflared tunnel
After=syslog.target network-online.target

[Service]
Type=simple
User=cloudflared
ExecStart=/usr/local/bin/cloudflared tunnel --config /home/cloudflared/.cloudflared/nextcloud.yaml run nextcloud
Restart=on-failure
RestartSec=10
KillMode=process

[Install]
WantedBy=multi-user.target
```

Now simply enable and start your service, and verify that everything is working:
```
sudo systemctl enable tunnel-nextcloud
```
```
sudo systemctl start tunnel-nextcloud
```
