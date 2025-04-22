wow, such unfinished :(

working on this page for the updated part 2 guide.

# Jellyfin Setup Guide
wip

## Installation
wip

### System Installtion
Run the following command on your Ubuntu system, VM, or Proxmox LXC. You can learn about verify the script download integrity [here](https://jellyfin.org/docs/general/installation/linux/).
```
curl https://repo.jellyfin.org/install-debuntu.sh | sudo bash
```

### Docker Setup
wip

Paste in this compose file and make any edits if needed.
```
services:
  jellyfin:
    image: lscr.io/linuxserver/jellyfin:latest
    container_name: jellyfin
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=America/Los_Angeles
      - JELLYFIN_PublishedServerUrl=http://10.0.0.100 #optional
    volumes:
      - ./config:/config
      - /data:/data
    devices:
      - /dev/dri:/dev/dri #Use for Intel QuickSync
    ports:
      - 8096:8096
      - 7359:7359/udp #Service Discovery
      - 1900:1900/udp #Client Discovery
    restart: unless-stopped
```
[https://github.com/NVIDIA/nvidia-container-toolkit](https://github.com/NVIDIA/nvidia-container-toolkit)

## Permissions
If you're running this with docker, you can skip these steps!

Stop the Jellyfin service.
```sudo systemctl jellyfin stop```
Edit the Jellyfin service configuration to reflext your user.
```sudo nano /lib/systemd/system/jellyfin.service```
Now change the user and group to your main user. You can run the `id` command if you're unsure.
```                      
[Unit]
Description = Jellyfin Media Server
After = network-online.target

[Service]
Type = simple
EnvironmentFile = /etc/default/jellyfin
User = brandon
Group = brandon
WorkingDirectory = /var/lib/jellyfin
ExecStart = /usr/bin/jellyfin $JELLYFIN_WEB_OPT $JELLYFIN_FFMPEG_OPT $JELLYFIN_SERVICE_O>
Restart = on-failure
TimeoutSec = 15
SuccessExitStatus=0 143

[Install]
WantedBy = multi-user.target
```
Now change the permissions of the Jellyfin files and folders
```
chown -R brandon:brandon /etc/default/jellyfin
chown -R brandon:brandon /usr/bin/jellyfin
chown -R brandon:brandon /var/lib/jellyfin/
chown -R brandon:brandon /etc/jellyfin/
chown -R brandon:brandon /var/log/jellyfin/
chown -R brandon:brandon /var/cache/jellyfin/
chown -R brandon:brandon /usr/share/jellyfin
chown -R brandon:brandon /usr/share/jellyfin-ffmpeg
chown -R brandon:brandon /usr/lib/jellyfin/
chown -R brandon:brandon /usr/lib/jellyfin-ffmpeg/
```
Reload the daemon and restart jellyfin
```
systemctl daemon-reload
systemctl restart jellyfin
```
Check the user that is running Jellyfin
```
ps -aux | grep jellyfin
```
Source: [https://github.com/tteck/Proxmox/discussions/286](https://github.com/tteck/Proxmox/discussions/286)


## General Setup
wip

## Hardware Transcoding
Be sure to checkout the offical docs [here](https://jellyfin.org/docs/general/administration/hardware-acceleration/).

The following steps take place on the system, virtual machine or Proxmox LXC you're running Jellyfin on. Install the jellyfin-ffmpeg7. Remove the deprecated jellyfin meta package if it breaks the dependencies.
```
sudo apt update && sudo apt install -y jellyfin-ffmpeg7
```
Add jellyfin (and the user you're running jellyfin as that) to the render group.
```
sudo usermod -aG render jellyfin
sudo usermod -aG render brandon # since I'm running jellyfin as my user
sudo systemctl restart jellyfin
```
### Proxmox Setup (Intel QuickSync)
I recommend running this as a unprivlaged LXC that houses all your media. 

Add this your your containers configuration under using  `nano /etc/pve/lxc/100.conf` changing the ID to match the container you've installed Jellyfin on.
```
#Add these for Intel QuickSync
lxc.cgroup2.devices.allow: c 10:200 rwm
lxc.mount.entry: /dev/net dev/net none bind,create=dir
lxc.mount.entry: /dev/net/tun dev/net/tun none bind,create=file
```
Now we can confirm hardware transcoding is ready by intstalling the `intel-gpu-tools` package and running the command `intel_gpu_top`

### Proxmox Setup (NVIDIA)
wip 

Grab the download link for the driver [here](https://www.nvidia.com/en-us/drivers/unix/). Click the link for the _Latest Production Branch Version_ and right click the download link and copy the link. Within the Proxmox shell for your node run a `wget` command and paste in the link to download.

...

If running with nvidia you'd add this and remove the `/dev/dri` devices lines in the docker compose above.
```
    runtime: nvidia
       deploy:
         resources:
           reservations:
             devices:
               - driver: nvidia
                 count: all
                 capabilities: [gpu]
```


## Plugins
wip

1. [Intro Skipper](https://github.com/intro-skipper/intro-skipper)
2. [Fan Art](https://github.com/jellyfin/jellyfin-plugin-fanart)
3. [Jellyfin Newsletter](https://github.com/Cloud9Developer/Jellyfin-Newsletter-Plugin)
4. [TMDb Box Sets](https://github.com/jellyfin/jellyfin-plugin-tmdbboxsets)
5. [TheTVDB](https://github.com/jellyfin/jellyfin-plugin-tvdb)
6. [SkinManager](https://github.com/danieladov/jellyfin-plugin-skin-manager)
7. [Media Bar](https://github.com/IAmParadox27/jellyfin-plugin-media-bar)

## Other Tools
wip

### Jellystat
Find it [here](https://github.com/CyferShepard/Jellystat)

### Jellyseerr
Find it [here](https://github.com/fallenbagel/jellyseerr)

## DVR and Live TV
wip
