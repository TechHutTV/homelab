# Self-Hosted Media Server and Aggregation

This is currently a work in progress. Make sure to review everything here and if you have any issues please submit it or suggest any edits. Also, checkout the [Servarr Docker Setup](https://wiki.servarr.com/docker-guide) for more details on installing the stack.


## Navigation
* [Apps](https://github.com/TechHutTV/homelab/tree/main/apps)
* [Home Assistant](https://github.com/TechHutTV/homelab/tree/main/homeassistant)
* [__Media Server__](https://github.com/TechHutTV/homelab/tree/main/media)
  - [Companion Video](#companion-video)
    * [Updates Since Video Publish](#updates-since-video-publish)
  - [Data Directory](#data-directory)
    * [Folder Mapping](#folder-mapping)
    * [Network Share](#network-share)
  - [User Permissions](#user-permissions)
  - [Gluetun VPN](#gluetun-vpn)
    * [Testing Gluetun Connectivity](#testing-gluetun-connectivity)
    * [Passing Through Containers](#passing-through-containers)
    * [External Container to Gluetun](#external-container-to-gluetun)
    * [Container in another docker-compose.yml](#container-in-another-docker-composeyml)
    * [Arr apps wont connect to prowlarr](#arr-apps-wont-connect-to-prowlarr)
    * [Gluetun Proxmox Fix](#gluetun-proxmox-fix)
    * [Reduce Gluetun Ram Usage](#reduce-gluetun-ram-usage)
    * [Testing Other Containers](#testing-other-containers)
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
Easy command to create the download directory scheme.
```
mkdir -p downloads/qbittorrent/{completed,incomplete,torrents} && mkdir -p downloads/nzbget/{completed,intermediate,nzb,queue,tmp}
```

### Network Share (VM)
I generally install Docker on the same LXC that I have my media server on as well as all my data. This; however, is [not recommened by Proxmox](https://www.reddit.com/r/Proxmox/comments/1afslhs/should_i_use_lxc_or_vm_for_running_docker/). Going forward you should create a seperate VM for all your docker containers and mount the data directory we created in the storage guide with the share. 

Within the VM install `cifs-utils` 
```
sudo apt install cifs-utils
```
Now, edit the fstab file and add the following lines editing them to match your information.
```
sudo nano /etc/fstab
//10.0.0.90/data /data cifs uid=1000,gid=1000,username=user,password=password,iocharset=utf8 0 0
//10.0.0.90/docker /docker cifs uid=1000,gid=1000,username=user,password=password,iocharset=utf8 0 0
```
Storing the user creditentials within this file it's the best idea. Check out [this question](https://unix.stackexchange.com/questions/178187/how-to-edit-etc-fstab-properly-for-network-drive) on Stack Exchange to learn more.
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

## Gluetun VPN
### Testing Gluetun Connectivity 
Once your containers are up and running, you can test your connection is correct and secured. This assumes you keeo the gluetun container name. Learn more at the [gluetun wiki](https://github.com/qdm12/gluetun-wiki/blob/main/setup/test-your-setup.md).
```
docker run --rm --network=container:gluetun alpine:3.18 sh -c "apk add wget && wget -qO- https://ipinfo.io"
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
Add ```--network=container:gluetun``` when launching the container, provided Gluetun is already running on the same machine.

### Container in another docker-compose.yml
Add network_mode: "container:gluetun" to your docker-compose.yml, provided Gluetun is already running. Ensure you open the ports through the the gluetun container.

### Arr apps wont connect to prowlarr

I like to run Prowlarr though a VPN as it is the service that will fetch magnet links and skim torrent sites. If you have issues with connections between arr apps and prowlarr you may need to [allow gluetun to access your lan](https://github.com/qdm12/gluetun-wiki/blob/main/setup/connect-a-lan-device-to-gluetun.md#access-your-lan-through-gluetun). You can do this by adding the following to your docker compose file under the glutun enviromental varibles. 
``` yaml
- FIREWALL_OUTBOUND_SUBNETS=192.168.1.0/24 #change to your specific subnet
```

see issue [here](https://github.com/TechHutTV/homelab/issues/9)

### Gluetun Proxmox Fix

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
As mentioned in this [issue](https://github.com/TechHutTV/homelab/issues/12) there is a [feature request](https://github.com/qdm12/gluetun/issues/765#issuecomment-1019367595) on the Glutun Github page to help reduce ram usage. Gluetun bundles a recursive caching DNS resolver called unbound for handling domain name requests securely. Over time the cache size, which rests in RAM, can balloon to gigabytes.

You can do this by adding the following to your docker compose file under the glutun enviromental varibles.
```
BLOCK_MALICIOUS=off
```
This may not be an issue as [DNS over HTTPS in Go to replace Unbound](https://github.com/qdm12/gluetun/issues/137) is impletmented, but it's worth the mention.

### Testing Other Containers
Jump into the Exec console and run the wget command below. Tested with nzbget, deluge, and prowlarr. Ensure you open the ports through the the gluetun container.
```
docker exec -it conatiner_name bash
wget -qO- https://ipinfo.io
```
## Download Clients

### NZBGet

#### Fix directory does not appear to exist inside the container error
This error may appear within Sonarr and Radarr. Once NZBGet is setup go to settings and under **INCOMING NZBS** change the **AppendCategoryDir** to **No**. This will prevent some potential mapping issues and save on unnessesary directories.

### qBittorrent

#### qBittorrent Stalls with VPN Timeout
I experianced where qBittorrent stalls out if there is a timeout or any type of interuption on the VPN. This is good becuase it drops connection, but I need it to fire back up when the connection is restored without manually restarting the container.

__Solution #1:__ Within the WebUI of qbittorrent head over to advanced options and select ```tun0``` as the networking interface. See image below for example.

![Set Network Interface to tun0](https://github.com/TechHutTV/homelab/blob/main/media/images/qbittorrent_tun0.jpeg)

Next, I added ```HEALTH_VPN_DURATION_INITIAL=120s``` to my glutun enviroment varibles as [per this issue](https://github.com/qdm12/gluetun/issues/1832). I updated my arr-compose.yaml above with this varible so you may already have this enabled. You can learn more about this on their [wiki](https://github.com/qdm12/gluetun-wiki/blob/main/faq/healthcheck.md). If you continue to have issues continue to next solution.

__Solution #2:__ Another solution, that can be used in conjection with __Solution #1__ is using the [deunhealth](https://github.com/qdm12/deunhealth/tree/main) container to automatically restart qbittorrent when it give an unheathly status. 

First, add the deunhealth service to your stack.
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
      deunhealth.restart.on.unhealthy= "true" # Label added for deunhealth monitoring
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=America/Los_Angeles
      - WEBUI_PORT=8080
      - TORRENTING_PORT=6881
    volumes:
      - /docker/qbittorrent:/config
      - /data:/data
    network_mode: service:gluetun
    healthcheck:
        test: ping -c 1 www.google.com || exit 1
        interval: 60s
        retries: 3
        start_period: 20s
        timeout: 10s
```
Relevent Resources: [DBTech video on deunhealth](https://www.youtube.com/watch?v=Oeo-mrtwRgE), [gluetun/issues/2442](https://github.com/qdm12/gluetun/issues/2442) and [gluetun/issues/1277](https://github.com/qdm12/gluetun/issues/1277#issuecomment-1352009151)

## arr Apps

Please refer to the [video tutorial](https://www.youtube.com/watch?v=Uzqf0qlcQlo) for this. We will soon update this section with text documentation.

## No VPN arr-compose (NOT RECOMMEDED)

```
services:
  qbittorrent:
    image: lscr.io/linuxserver/qbittorrent:latest
    container_name: qbittorrent
    restart: unless-stopped
    ports:
      - 8080:8080 # web interface
      - 6881:6881 # torrent port
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=America/Los_Angeles
      - WEBUI_PORT=8080
      - TORRENTING_PORT=6881
    volumes:
      - /docker/qbittorrent:/config
      - /data:/data
  nzbget:
    image: lscr.io/linuxserver/nzbget:latest
    container_name: nzbget
    ports:
      - 6789:6789
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=America/Los_Angeles
      - NZBGET_USER=user
      - NZBGET_PASS=password
    volumes:
      - /etc/localtime:/etc/localtime:ro
      - /docker/nzbget:/config
      - /data:/data
    restart: unless-stopped
  prowlarr:
    image: lscr.io/linuxserver/prowlarr:latest
    container_name: prowlarr
    ports:
      - 9696:9696
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=America/Los_Angeles
    volumes:
      - /etc/localtime:/etc/localtime:ro
      - /docker/prowlarr:/config
    restart: unless-stopped

```
