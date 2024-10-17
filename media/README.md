# Self-Hosted Media Server and Aggregation

This is currently a work in progress. Please refer to the [Servarr Docker Setup](https://wiki.servarr.com/docker-guide) for more details on installing the stack.

## Companion Video
[![my NEW Proxmox Media Server - Full Walkthrough Guide Pt.2 (Jellyfin, Sonarr, Gluetun, and MORE)](https://i3.ytimg.com/vi/Uzqf0qlcQlo/mqdefault.jpg)](https://www.youtube.com/watch?v=Uzqf0qlcQlo)

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
### Fix NZBGet "directory does not appear to exist inside the container" error in Sonarr/Radarr
Once NZBGet is setup go to settings and under **INCOMING NZBS** change the **AppendCategoryDir** to **No**. This will prevent some potential mapping issues and save on unnessesary directories.

### Network Share
Before switching to zfs on Proxmox I used to use a network share from Unraid to store downloads and all my media. This was created by adding the share to the fstab file within my Docker server. Due note, the appliction ```cifs-utils``` is required for this method.
```
//10.0.0.90/data /media/data cifs uid=1000,gid=100,username=user,password=password,iocharset=utf8 0 0
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

## VPN Information
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

### Testing Other Containers
Jump into the Exec console and run the wget command below. Tested with nzbget, deluge, and prowlarr. Ensure you open the ports through the the gluetun container.
```
docker exec -it conatiner_name bash
wget -qO- https://ipinfo.io
```
### Qbittorrent Stalls and VPN Timeout (under investigation)
I experianced where qBittorrent stalls out if there is a timeout or any type of interuption on the VPN. This is good becuase it drops connection, but I need it to fire back up when the connection is restored without manually restarting the container.

#### Solution #1
Within the WebUI of qbittorrent head over to advanced options and select ```tun0``` as the networking interface. See image below for example.

![Set Network Interface to tun0](https://github.com/TechHutTV/homelab/blob/main/media/images/qbittorrent_tun0.jpeg)

Next, I added ```HEALTH_VPN_DURATION_INITIAL=120s``` to my glutun enviormental varibles as [per this issue](https://github.com/qdm12/gluetun/issues/1832). I updated my arr-compose.yaml above with this varible so you may already have this enabled. You can learn more about this on their [wiki](https://github.com/qdm12/gluetun-wiki/blob/main/faq/healthcheck.md). If you continue to have issues continue to next solution.

#### Solution #2
Another solution, that can be used in conjection with __Solution #1__ is using the [deunhealth](https://github.com/qdm12/deunhealth/tree/main) container to automatically restart qbittorrent when it give an unheathly status. 

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
      deunhealth.restart.on.unhealthy: true # Label added for deunhealth monitoring
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
