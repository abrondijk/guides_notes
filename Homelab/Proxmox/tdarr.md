# TDarr installation
First attempt, rough guide.
Start by setting up Nvidia passtrough if you have a capable GPU. See guide in current folder.

## Create a user
This hooks into the guide for setting up correct ACL, from which I get the `-uid 1000` part, make sure you set this up correctly, because that's what grants access to your files. We also let `adduser` automatically setup a group.
```
adduser --uid 1000 tdarr
```
Fill in the prompts, and add it to the sudo group so that you may install packages:
```
usermod -aG sudo tdarr
```
Login as your new user, and we can get started.

## Dependencies
These I know for your that you'll need.
``` 
sudo apt-get install -y wget mkvtoolnix ccextractor handbrake-cli unzip 
```

## Set up FFMPEG
Got this from the dockerfile, works for me:
```
wget https://repo.jellyfin.org/releases/server/ubuntu/versions/jellyfin-ffmpeg/4.3.2-1/jellyfin-ffmpeg_4.3.2-1-focal_amd64.deb
sudo apt install -y ./jellyfin-ffmpeg_4.3.2-1-focal_amd64.deb
sudo ln -s /usr/lib/jellyfin-ffmpeg/ffmpeg /usr/local/bin/ffmpeg
```

## Downloading and running Tdarr itself
Download updater, See an overview of all versions here: [https://f000.backblazeb2.com/file/tdarrs/versions.json](https://f000.backblazeb2.com/file/tdarrs/versions.json), newest are lowest.
Also extract file, and run updater:
```
wget https://f000.backblazeb2.com/file/tdarrs/versions/2.00.12/linux_x64/Tdarr_Updater.zip
unzip Tdarr_Updater.zip
./Tdarr_Updater
```
Set permissions on the actual programs:
```
chmod +x Tdarr_Server/Tdarr_Server
chmod +x Tdarr_Node/Tdarr_Node
```

Run the server in a terminal (temporary, we'll set up a service later)
```
Tdarr_Server/./Tdarr_Server
```
This will give you an overview of what's working and what's not:
![2021-12-19T13:46:35_903x404_scrot](https://user-images.githubusercontent.com/20231417/146675341-5ada4f85-70f8-43f5-b2f7-26d8ee20dd78.png)

Do the same for the node:
```
Tdarr_Node/./Tdarr_Node
```
![2021-12-19T13:52:31_987x255_scrot](https://user-images.githubusercontent.com/20231417/146675533-603295dc-b234-4bbb-99b7-26f4cd28540a.png)

## Configuration

Close the node, and we'll get on to editing the configs.
Open `configs/Tdarr_Node_Config.json` and setup your media paths, this is also where you can edit the Node port and the target Server port.

Add the path to your media  **_`without a leading /`_**  to the server field, and the same but **_`with the leading /`_** for the node's path to the libary.

Also set your ffmpeg path to the one we installed earlier, or else your transcodes won't be able to use your Nvidia GPU.

In the end, my `configs/Tdarr_Node_Config.json` ended up looking like this:
```json
{
  "nodeID": "musty-moa",
  "nodeIP": "0.0.0.0",
  "nodePort": "8267",
  "serverIP": "0.0.0.0",
  "serverPort": "8266",
  "handbrakePath": "",
  "ffmpegPath": "/usr/local/bin/ffmpeg",
  "mkvpropeditPath": "",
  "pathTranslators": [
    {
      "server": "mnt/general",
      "node": "/mnt/general"
    }
  ]
}
```

If, like me, you're running the Node and the Server on the same host, there's no need to edit the IP addresses.
The `configs/Tdarr_Server_Config.json` contains just IP and port settings, along with handbrake and ffmpeg paths, but I haven't had to edit those.

## Setting up services
Create `/etc/systemd/system/tdarr_server.service` and fill it with the following content:
```
[Unit]
Description=Tdarr Server - Audio/Video Library Analytics & Transcode/Remux Automation
Documentation=https://github.com/HaveAGitGat/Tdarr

[Service]
Type=simple
User=tdarr
ExecStart=/home/tdarr/Tdarr_Server/Tdarr_Server
Restart=on-failure

[Install]
WantedBy=multi-user.target
```
Next create `/etc/systemd/system/tdarr_node.service` and make it look something like this:
```
[Unit]
Description=Tdarr Node - Audio/Video Library Analytics & Transcode/Remux Automation
Documentation=https://github.com/HaveAGitGat/Tdarr
After=tdarr_server.service

[Service]
Type=simple
User=tdarr
ExecStart=/home/tdarr/Tdarr_Node/Tdarr_Node
Restart=on-failure

[Install]
WantedBy=multi-user.target
```
Reload your services:
```
sudo systemctl daemon-reload
```
And you can enabel and start them:
```
sudo systemctl enable tdarr_server
sudo systemctl enable tdarr_node
sudo systemctl start tdarr_server
sudo systemctl start tdarr_node
```

## Finishing
And that's it, you can now visit your tdarr page at http://whatever.ip:8065, given that you didn't change the ports in the configuration step.
You may also remove the sudo rights from your tdarr user if you so desire:

```
usermod -G sudo tdarr
```
