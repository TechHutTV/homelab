_Work in Progress_

# Jellyfin Setup Guide
Welcome to the ultimate Jellyfin setup guide. 

## Installation
There are two options for installing Jellyfin. Both work great and it's all a matter of preference. I generally install Jellyfin directly on the LXC within Proxmox the contains all my data.

### System Installtion
Run the following command on your Ubuntu system, VM, or Proxmox LXC. You can learn about verify the script download integrity [here](https://jellyfin.org/docs/general/installation/linux/).
```
curl https://repo.jellyfin.org/install-debuntu.sh | sudo bash
```

### Docker Setup
Docker is another option to install and run Jellyfin. Checkout the compose.yaml for the full stack.

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


## Configuring Jellyfin
Open your web browser and navigate to your installed instance of Jellyfin using `http://IP:8096` and once there you can power through the initial setup by selecting your preferred language, then create an admin account with a secure username and password. Next, set up your media libraries by adding folders for movies, TV shows, or music. I tend to keep everything in my `/data` directory as shown in the media page on this repo.

## Hardware Transcoding
This focuses on trascoding with Intel QuickSync. In my experiance it is simply the best option. If you're running a AMD CPU you can pickup a Intel Arc GPU fairly cheap. If you have any issues or don't have access to a Intel CPU or an Arc GPU be sure to checkout the offical docs [here](https://jellyfin.org/docs/general/administration/hardware-acceleration/). If you're not doing this on Proxmox you can skip to the Ubuntu setup.

### Proxmox Setup
I recommend running this as a unprivlaged LXC that houses all your media. If you're running the Docker installation it's recommened to use a VM.

Add the lines below to your containers configuration under changing the ID to match the container you've installed Jellyfin on.
```
nano /etc/pve/lxc/100.conf
```
```
#Add these for Intel QuickSync
lxc.cgroup2.devices.allow: c 10:200 rwm
lxc.mount.entry: /dev/net dev/net none bind,create=dir
lxc.mount.entry: /dev/net/tun dev/net/tun none bind,create=file
```

### Ubuntu Setup
The following steps take place on the Ubuntu server, virtual machine or Proxmox LXC you're running Jellyfin on. Install the `jellyfin-ffmpeg7`. Remove the deprecated jellyfin meta package if it breaks the dependencies.
```
sudo apt update && sudo apt install -y jellyfin-ffmpeg7
```
Add jellyfin (and the user you're running jellyfin as that) to the render group.
```
sudo usermod -aG render jellyfin
sudo usermod -aG render brandon # since I'm running jellyfin as my user
sudo systemctl restart jellyfin
```
Now we can confirm hardware transcoding is ready by intstalling the `intel-gpu-tools` package and running the command `intel_gpu_top`.
```
sudo apt install intel-gpu-tools
intel_gpu_top
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
Now to expand functionality of Jellyfin I recommened these 3rd party tools that integrate well with Jellyfin.

### Jellystat
Find it [here](https://github.com/CyferShepard/Jellystat)

### Jellyseerr
Find it [here](https://github.com/fallenbagel/jellyseerr)

## DVR and Live TV
This will require some extra hardware and a paid service for the guide data. HDHomeRun is great and it's what I use. Go to _Dashboard → Live TV → TV Sources → Add Tuner Device_. Select your tuner type _HDHomeRun_. Enter the tuner’s IP address and click Save.

Next you want to setup guide data. _Guide Providers → Add Guide Provider_. Schedules Direct is a paid service, but their awesome. It's a non-profit and they commit to the Jellyfin code directly. Create a account and you can use it free for a week. Input your username and password and give it some time to update the data. I didn't need to but you can map the channels by going to _Channels → Map Channels_.

## Remote Connections
wip
