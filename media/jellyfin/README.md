wow, such unfinished :(

working on this page for the updated part 2 guide.

# Jellyfin Setup Guide
wip

## Installation
wip

### System Installtion
wip

### Docker Setup
wip

## Permissions
wip

## General Setup
wip

## Hardware Transcoding
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


