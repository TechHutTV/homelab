# Self-Hosted Media Server and Aggregation

This is currently a work in progress. Make sure to review everything here and if you have any issues please submit it or suggest any edits. Also, checkout the [Servarr Docker Setup](https://wiki.servarr.com/docker-guide) for more details on installing the stack.


## Navigation
* [Apps](https://github.com/TechHutTV/homelab/tree/main/apps)
* [Home Assistant](https://github.com/TechHutTV/homelab/tree/main/homeassistant)
* [__Media Server__](https://github.com/TechHutTV/homelab/tree/main/media)
  - [Companion Video](#companion-video)
    * [Updates Since Video Publish](#updates-since-video-publish)
  - [Media Server](#media-server)
  - [Data Directory](#data-directory)
    * [Folder Mapping](#folder-mapping)
    * [Network Share](#network-share)
  - [User Permissions](#user-permissions)
  - [Gluetun VPN](#gluetun-vpn)
    * [Testing Gluetun Connectivity](#testing-gluetun-connectivity)
    * [Passing Through Containers](#passing-through-containers)
    * [External Container to Gluetun](#external-container-to-gluetun)
    * [Gluetun Proxmox LXC Setup](#gluetun-proxmox-fix)
    * [Reduce Gluetun Ram Usage](#reduce-gluetun-ram-usage)
  - [Download Clients](#download-clients)
    * [NZBGet](#nzbget)
      + [Fix "directory does not appear" error in Sonarr/Radarr](#fix-directory-does-not-appear-to-exist-inside-the-container-error)
    * [qBittorrent](#qbittorrent)
      + [qBittorrent Stalls with VPN Timeout](#qbittorrent-stalls-with-vpn-timeout)
  - [arr Apps](#arr-apps)
* [Server Monitoring](https://github.com/TechHutTV/homelab/tree/main/monitoring)
* [Surveillance System](https://github.com/TechHutTV/homelab/tree/main/surveillance)
* [Storage](https://github.com/TechHutTV/homelab/tree/main/storage)
* [Proxy Managment](https://github.com/TechHutTV/homelab/tree/main/proxy)

## Companion Video
[![my NEW Proxmox Media Server - Full Walkthrough Guide Pt.2 (Jellyfin, Sonarr, Gluetun, and MORE)](https://i3.ytimg.com/vi/Uzqf0qlcQlo/mqdefault.jpg)](https://www.youtube.com/watch?v=Uzqf0qlcQlo)

### Updates Since Video Publish
* Set my networking interface to ```tun0``` and added the ```HEALTH_VPN_DURATION_INITIAL=120s``` enviroment variable to gluetun. 
* Added the [deunhealth](https://github.com/qdm12/deunhealth/tree/main) container to restart qbittorrent if it becomes unhealth due to a VPN timeout. See details [here](https://github.com/TechHutTV/homelab/blob/main/media/README.md#qBittorrent).
* Added static IPs with a custom docker networking making it easier to configure Prowlarr with internal docker IP addresses. See details [here](https://github.com/TechHutTV/homelab/pull/14).
* Removed Readarr and added [ytdl-sub](https://ytdl-sub.readthedocs.io/en/latest/) to the arr-compose.

## Media Server
Media Servers have their own guides! Check the link below and it will take you to the folder for the guides.

- [Jellyfin](https://github.com/TechHutTV/homelab/tree/main/media/jellyfin)
- [Plex](https://github.com/TechHutTV/homelab/tree/main/media/plex)

## Data Directory
### Folder Mapping
It's good practise to give all containers the same access to the same root directory or share. This is why all containers in the compose file have the bind volume mount ```/data:/data```. It makes everything easier, plus passing in two volumes such as the commonly suggested /tv, /movies, and /downloads makes them look like two different file systems, even if they are a single file system outside the container. See my current setup below. 
```
data
├── books
├── downloads
│   ├── qbittorrent
│   │   ├── completed
│   │   ├── incomplete
│   │   └── torrents
│   └── nzbget
│       ├── completed
│       ├── intermediate
│       ├── nzb
│       ├── queue
│       └── tmp
├── movies
├── music
├── shows
└── youtube
```
Here is a easy command to create the download directory scheme.
```
mkdir -p downloads/qbittorrent/{completed,incomplete,torrents} && mkdir -p downloads/nzbget/{completed,intermediate,nzb,queue,tmp}
```

### Network Share (VM)
I generally install Docker on the same LXC that I have my media server on as well as all my data. This; however, is [not recommened by Proxmox](https://pve.proxmox.com/pve-docs/pve-admin-guide.html#chapter_pct). Going forward you should create a seperate VM for all your docker containers and mount the data directory we created in the storage guide with the share. 

Within the VM install `cifs-utils` 
```
sudo apt install cifs-utils
```
Now, edit the fstab file and add the following lines editing them to match your information.
```
sudo nano /etc/fstab
//10.0.0.100/data /data cifs uid=1000,gid=1000,username=user,password=password,iocharset=utf8 0 0
```
Storing the user creditentials within this file it's the best idea. Check out [this question](https://unix.stackexchange.com/questions/178187/how-to-edit-etc-fstab-properly-for-network-drive) on Stack Exchange to learn more.

Now reload the configuration and mount the shares with the following commands.
```
sudo systemctl daemon-reload
sudo mount -a
```

## User Permissions
Using bind mounts (path/to/config:/config) may lead to permissions conflicts between the host operating system and the container. To avoid this problem, you we specify the user ID (PUID) and group ID (PGID) to use within some of the containers. This will give your user permission to read and write configuration files, etc.

In the compose files I use PUID=1000 and PGID=1000, as those are generally the default ID's in most Linux systems, but depending on your setup you may need to chage this.

```
id your_user
uid=1000(brandon),gid=1003(brandon),groups=1000(data-share),988(docker)
```
In the example output above, If using a network share I would need to edit the compose.yaml with gid=1003. If you are using a network share mounted though ```/etc/fstab``` match the permissions there. I use Cockpit with a custom group fpr shares so my permissions are ```uid=1000(brandon),gid=1000(data-share)```.
If you run into errors, after creating all the folders you can assign the permissions using chmod. For example,
```
sudo chown -R 1000:1000 /data
```
Also, I like to store all my Docker configurations in a root /docker directory on my Linux system. These can go whereever you prefer whether that be your home directory or somewhere else. Do note, many Docker apps may have issues if you're trying to store you Docker configurations in a SMB network share.
```
mkdir /docker
sudo chown -R 1000:1000 /docker
```
## Gluetun VPN
### Testing Gluetun Connectivity 
Once your containers are up and running, you can test your connection is correct and secured. This assumes you keeo the gluetun container name. Learn more at the [gluetun wiki](https://github.com/qdm12/gluetun-wiki/blob/main/setup/test-your-setup.md).
```
docker run --rm --network=container:gluetun alpine:3.18 sh -c "apk add wget && wget -qO- https://ipinfo.io"
```
If you'd like to test Gluetun Connectivity from a container using the service jump into the Exec console and run the wget command below. Tested with nzbget, deluge, and prowlarr. Ensure you open the ports through the the gluetun container.
```
docker exec -it conatiner_name bash
wget -qO- https://ipinfo.io
```
### Passing Through Containers 
When containers are in the same docker compose they all you need to add is a ```network_mode: service:container_name``` and open the ports through the the gluetun container. See example from the compose.yaml below.
```
services:
  gluetun: # This config is for wireguard only tested with AirVPN
    image: qmcgaw/gluetun
    container_name: gluetun
    ...
    ports:
      - 8888:8112 # deluge web interface
      - 58846:58846 # deluge RPC
  deluge:
    image: linuxserver/deluge:latest
    container_name: deluge
    ...
    network_mode: service:gluetun
```
### External Container to Gluetun
Add the following when launching the container, provided Gluetun is already running on the same machine. 
```
--network=container:gluetun
``` 
If the container is in another docker compose.yaml, assuming Gluetun is already running add the following network mode. Ensure you open the ports through the the gluetun container.
```
network_mode: "container:gluetun"
```
### Arr apps wont connect to prowlarr

I like to run Prowlarr though a VPN as it is the service that will fetch magnet links and skim torrent sites. If you have issues with connections between arr apps and prowlarr you may need to [allow gluetun to access your lan](https://github.com/qdm12/gluetun-wiki/blob/main/setup/connect-a-lan-device-to-gluetun.md#access-your-lan-through-gluetun). You can do this by adding the following to your docker compose file under the glutun enviromental varibles. 
``` yaml
- FIREWALL_OUTBOUND_SUBNETS=192.168.1.0/24 #change to your specific subnet
```

see issue [here](https://github.com/TechHutTV/homelab/issues/9)

### Gluetun Proxmox LXC Setup

"cannot Unix Open TUN device file: operation not permitted and cannot create TUN device file node: operation not permitted" May happen if you're running this on LXC containers.

Find your container number, for example mine is 101

Edit `/etc/pve/lxc/101.conf` and add:
```
lxc.cgroup2.devices.allow: c 10:200 rwm
lxc.mount.entry: /dev/net dev/net none bind,create=dir
lxc.mount.entry: /dev/net/tun dev/net/tun none bind,create=file
```
Make sure you pass through the tun device (/dev/net/tun:/dev/net/tun) as shown in my compose file.

### Reduce Gluetun Ram Usage
As mentioned in this [issue](https://github.com/TechHutTV/homelab/issues/12) there is [feature request](https://github.com/qdm12/gluetun/issues/765#issuecomment-1019367595) on the Glutun Github page to help reduce ram usage. Gluetun bundles a recursive caching DNS resolver called unbound for handling domain name requests securely. Over time the cache size, which rests in RAM, can balloon to gigabytes.

You can do this by adding the following to your docker compose file under the glutun enviromental varibles.
```
BLOCK_MALICIOUS=off
```
This may not be an issue as [DNS over HTTPS in Go to replace Unbound](https://github.com/qdm12/gluetun/issues/137) is impletmented, but it's worth the mention.

## Download Clients

### NZBGet

#### Fix directory does not appear to exist inside the container error
This error may appear within Sonarr and Radarr. Once NZBGet is setup go to settings and under **INCOMING NZBS** change the **AppendCategoryDir** to **No**. This will prevent some potential mapping issues and save on unnessesary directories.

### qBittorrent

### Download Directories
If following my /data:/data directory scheme and used the command to setup the download directories open the qBitttorent Web UI and do under Settings > Downloads and change the paths.

__Default Save Path:__ `/data/downloads/qbittorrent/completed`

__Keep incomplete torrents in:__ `/data/downloads/qbittorrent/incomplete`

__Copy .torrent files to:__ `/data/downloads/qbittorrent/torrents`

#### qBittorrent Login Credentials
When you first launch qBittorrent it will be givin a random password. To find this password you can stop the stack and run without detached mode.
```
docker compose up
```
You will find the password in the console. Keep it running and login with 'admin' and the random short string password it generated.

Now, go to your settings and setup a new username and password under WebUI > Authentication.

#### qBittorrent Stalls with VPN Timeout
qBittorrent stalls out if there is a timeout or any type of interuption on the VPN. This is good becuase it drops connection, but we need it to fire back up when the connection is restored without manually restarting the container.

__Solution #1:__ Within the WebUI of qbittorrent head over to advanced options and select ```tun0``` as the networking interface. See image below for example.

![Set Network Interface to tun0](https://github.com/TechHutTV/homelab/blob/main/media/images/qbittorrent_tun0.jpeg)

Next, I added ```HEALTH_VPN_DURATION_INITIAL=120s``` to my glutun enviroment varibles as [per this issue](https://github.com/qdm12/gluetun/issues/1832). I updated my compose.yaml above with this varible so you may already have this enabled. You can learn more about this on their [wiki](https://github.com/qdm12/gluetun-wiki/blob/main/faq/healthcheck.md). If you continue to have issues continue to next solution.

__Solution #2:__ Another solution, that can be used in conjection with __Solution #1__ is using the [deunhealth](https://github.com/qdm12/deunhealth/tree/main) container to automatically restart qbittorrent when it give an unheathly status. We've added this to our compose.yaml for this stack.
```
  deunhealth:
    image: qmcgaw/deunhealth
    container_name: deunhealth
    network_mode: "none"
    environment:
      - LOG_LEVEL=info
      - HEALTH_SERVER_ADDRESS=127.0.0.1:9999
      - TZ=America/Los_Angeles
    restart: always
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
```

Next we need to add a health check and label to our qbittorrent container. We add ```deunhealth.restart.on.unhealthy=true``` as a label and a simple ping health check as shown below.

```
  qbittorrent:
    image: lscr.io/linuxserver/qbittorrent:latest
    container_name: qbittorrent
    restart: unless-stopped
    labels:
      deunhealth.restart.on.unhealthy=true # Label added for deunhealth monitoring
    ...
```
Relevent Resources: [DBTech video on deunhealth](https://www.youtube.com/watch?v=Oeo-mrtwRgE), [gluetun/issues/2442](https://github.com/qdm12/gluetun/issues/2442) and [gluetun/issues/1277](https://github.com/qdm12/gluetun/issues/1277#issuecomment-1352009151)

## arr Apps

When connecting your *arr applications be sure to use the new configured IP addresses in the servarrnetwork. We will soon update this section with more text documentation.
