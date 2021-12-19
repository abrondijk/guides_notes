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
apt-get install -y mkvtoolnix ccextractor handbrake-cli unzip 
```

These may or may not be necessary, I got these from the Tdarr dockerfile and installed them before testing whether or not it worked without them, please update this guide if you find out more.
```
apt-get install -y software-properties-common git trash-cli \
                    curl autoconf automake autopoint appstream \
                    build-essential cmake git libass-dev libbz2-dev \
                    libfontconfig1-dev libfreetype6-dev libfribidi-dev \
                    libharfbuzz-dev libjansson-dev liblzma-dev \ 
                    libmp3lame-dev libnuma-dev libogg-dev libopus-dev \
                    libsamplerate-dev libspeex-dev libtheora-dev libtool \
                    libtool-bin libturbojpeg0-dev libvorbis-dev libx264-dev \
                    libxml2-dev libvpx-dev m4 make meson nasm ninja-build \
                    patch pkg-config python tar zlib1g-dev libva-dev libdrm-dev
```

## Downloading and running Tdarr itself
Download updater, See an overview of all versions here: [https://f000.backblazeb2.com/file/tdarrs/versions.json](https://f000.backblazeb2.com/file/tdarrs/versions.json), newest are lowest.
```
wget https://f000.backblazeb2.com/file/tdarrs/versions/LATEST_VERSION_HERE/linux_x64/Tdarr_Updater.zip
unzip Tdarr_Updater.zip
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

Close the node, and we'll get on to editing the configs.
Open `configs/Tdarr_Node_Config.json` and setup your media paths, this is also where you can edit the Node port and the target Server port.

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
ExecStart=/usr/bin/node /home/tdarr/Tdarr_Server/Tdarr_Server
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
ExecStart=/usr/bin/node /home/tdarr/Tdarr_Node/Tdarr_Node
Restart=on-failure

[Install]
WantedBy=multi-user.target
```
Reload your services:
```
systemctl daemon-reload
```
And you can enabel and start them:
```
systemctl enable tdarr_server
systemctl enable tdarr_node
systemctl start tdarr_server
systemctl start tdarr_node
```
