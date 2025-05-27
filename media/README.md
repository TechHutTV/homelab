# Self-Hosted Media Server and Aggregation

Make sure to review everything here and if you have any issues please submit it as an issue. Also, we are more than open to any suggests or edits. Also, checkout the [Servarr Docker Setup](https://wiki.servarr.com/docker-guide) for more details on installing the stack.

> [!CAUTION]
> Some MAJOR Updates! Moved the VPN configuration and some of the env variables to a `.env` file. If you're watching the current live video it's a huge change. Will be uploading a new one in the next few days.

## Navigation
* [Apps](https://github.com/TechHutTV/homelab/tree/main/apps)
* [Home Assistant](https://github.com/TechHutTV/homelab/tree/main/homeassistant)
* [__Media Server__](https://github.com/TechHutTV/homelab/tree/main/media)
  - [Companion Video](#companion-video)
    * [Updates Since Video Publish](#updates-since-video-publish)
  - [Media Server](#media-server)
    * [Jellyfin](https://github.com/TechHutTV/homelab/tree/main/media/jellyfin)
    * [Plex](https://github.com/TechHutTV/homelab/tree/main/media/plex)
  - [Data Directory](#data-directory)
    * [Folder Mapping](#folder-mapping)
    * [Network Share](#network-share)
  - [User Permissions](#user-permissions)
  - [Docker Compose and .env](#docker-compose-and-env)
  - [Gluetun VPN](#gluetun-vpn)
    * [Setup and Configuration](#setup-and-configuration)
    * [Testing Gluetun Connectivity](#testing-gluetun-connectivity)
    * [Passing Through Containers](#passing-through-containers)
    * [External Container to Gluetun](#external-container-to-gluetun)
    * [Gluetun Proxmox LXC Setup](#gluetun-proxmox-fix)
    * [Reduce Gluetun Ram Usage](#reduce-gluetun-ram-usage)
  - [Download Clients](#download-clients)
    * [NZBGet](#nzbget)
      + [NZBGet Login Credentials](#nzbget-login-credentials)
      + [Download Directories Mapping](#nzbget-download-directories)
      + [Fix "directory does not appear" error in Sonarr/Radarr](#fix-directory-does-not-appear-to-exist-inside-the-container-error)
    * [qBittorrent](#qbittorrent)
      + [qBittorrent Login Credentials](#qbittorrent-login-credentials)
      + [Download Directories Mapping](#qbittorrent-download-directories)
      + [qBittorrent Stalls with VPN Timeout](#qbittorrent-stalls-with-vpn-timeout)
  - [*arr Apps](#arr-apps)
* [Server Monitoring](https://github.com/TechHutTV/homelab/tree/main/monitoring)
* [Surveillance System](https://github.com/TechHutTV/homelab/tree/main/surveillance)
* [Storage](https://github.com/TechHutTV/homelab/tree/main/storage)
* [Proxy Management](https://github.com/TechHutTV/homelab/tree/main/proxy)

## Companion Video
```
# Updated video coming soon
[![alt text](image url)](video link)
```
### Updates Since Video Publish
* Added [ytdl-sub](https://ytdl-sub.readthedocs.io/en/latest/) to the `compose.yaml`. Remove if unwanted.

## Media Server
Media Servers have their own guides! Check the link below and it will take you to the folder for the guides.

- [Jellyfin](https://github.com/TechHutTV/homelab/tree/main/media/jellyfin)
- [Plex](https://github.com/TechHutTV/homelab/tree/main/media/plex)

## Data Directory
### Folder Mapping
It's good practice to give all containers the same access to the same root directory or share. This is why all containers in the compose file have the bind volume mount `/data:/data`. It makes everything easier, plus passing in two volumes such as the commonly suggested `/tv`, `/movies`, and `/downloads` makes them look like two different file systems, even if they are a single file system outside the container. See my current setup below.
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
Here is a easy command to create the download directory scheme. Run within the `/data` directory.
```bash
mkdir -p downloads/qbittorrent/{completed,incomplete,torrents} && mkdir -p downloads/nzbget/{completed,intermediate,nzb,queue,tmp}
```

### Network Share
I generally install Docker on the same LXC that I have my media server on as well as all my data. This, however, is [not recommended by Proxmox](https://pve.proxmox.com/pve-docs/pve-admin-guide.html#chapter_pct). Going forward you should create a separate VM for all your docker containers and mount the data directory we created in the [storage guide](https://github.com/TechHutTV/homelab/tree/main/storage) with the share. You can also use this method if you're using a separate share on another machine running something like Unraid or TrueNAS.

Within the VM install `cifs-utils`
```bash
sudo apt install cifs-utils
```
Now, edit the `fstab` file and add the following lines editing them to match your information:
```bash
sudo nano /etc/fstab
```
```
//10.0.0.100/data /data cifs uid=1000,gid=1000,username=user,password=password,iocharset=utf8 0 0
```
Storing the user credentials within this file isn't the best idea. Check out [this question](https://unix.stackexchange.com/questions/178187/how-to-edit-etc-fstab-properly-for-network-drive) on Stack Exchange to learn more.

Now reload the configuration and mount the shares with the following commands.
```bash
sudo systemctl daemon-reload
sudo mount -a
```

## User Permissions
Using bind mounts (`path/to/config:/config`) may lead to permission conflicts between the host operating system and the container. To avoid this problem, you can specify the user ID (`PUID`) and group ID (`PGID`) to use within some of the containers. This will give your user permissions to read and write configuration files, etc.

In the compose file I use `PUID=1000` and `PGID=1000`, as those are generally the default IDs in most Linux systems, but depending on your setup you may need to change this.

```bash
id your_user
```
This command will return something like the following:
```
uid=1000(your_user) gid=1000(your_user) groups=1000(your_user),27(sudo),24(cdrom),30(dip),46(plugdev),108(lxd)
```
If you are using a network share mounted though `/etc/fstab` match the permissions there. Learn more above.

If you run into errors after creating all the folders you can assign the permissions using `chown`. For example:
```bash
sudo chown -R 1000:1000 /data
```
Also, I like to store all my Docker configurations in a root `/docker` directory on my Linux system. These can go wherever you prefer whether that be your home directory or somewhere else. Do note, many Docker apps may have issues if you're trying to store you Docker configurations in a SMB network share.
```bash
mkdir /docker
sudo chown -R 1000:1000 /docker
```
## Docker Compose and .env
Navigate to the directory you want to spin up the servarr stack in. I run mine from `/docker/servarr` but you can run it from anywhere you'd like such as `/home/user/docker/servarr`. Then download the `compose.yaml` and `.env` files from this repo.
```bash
wget https://github.com/TechHutTV/homelab/raw/refs/heads/main/media/compose.yaml && wget https://github.com/TechHutTV/homelab/raw/refs/heads/main/media/.env
```
Most of our editing is going to be done in the `.env` file. Here you change your `UID` and `GID`, timezone, and add all your VPN keys and info. You can also make edits to the `compose.yaml` file such as the mount point locations, for example, if you are using something other than `/data:/data` or even changing the docker network IP addresses for your services.

## Gluetun VPN

### Setup and Configuration
I like to set this out with [AirVPN](https://airvpn.org/?referred_by=673908) (referral link). I'm not affiliated with them in any way other than the referral link. I've tried a few other providers and they're my preference. If you already have a VPN checkout the [providers](https://github.com/qdm12/gluetun-wiki/tree/main/setup/providers) page on their wiki.

On AirVPN navigate to the **Client Area** from here select the **Config Generator**. Now in the options select **Linux** then toggle the **WireGuard** option. Select **New device** and then scroll down to **By single server** and select a server that is best for you. For example, _Titawin (Vancouver)_ was my selection because, at the time, it had the fewest users with good speeds. Scroll all the way to the bottom and select **Generate**. This will download a conf file with all of your information.

Back in AirVPN navigate to the **Client Area** from here select **Manage** under **Ports**. If you already have a port open click on **Test open** otherwise click the plus button under **Add a new port** then click **Test open** for that port. Here you will find the specific servers that you can use your port on. If there is a `Connection refused` warning next the server you generated your configuration for change the port until the warning goes away. For example, in my case the _'Titawin (Vancouver)_ server that I selected with my port is good to use.

> [!CAUTION]
> Do NOT forward on your router the same ports you use on your listening services while connected to the VPN.

Now, in the same directory as your docker `compose.yaml` file create a `.env` file. Paste in the variables below and then add all the information from your downloaded `.conf` file.

```bash
nano .env
```
```bash
# General UID/GIU and Timezone
TZ=America/Los_Angeles
PUID=1000
PGID=1000

# Input your VPN provider and type here
VPN_SERVICE_PROVIDER=airvpn
VPN_TYPE=wireguard

# Mandatory, airvpn forwarded port
FIREWALL_VPN_INPUT_PORTS=port # mandatory, airvpn forwarded port

# Copy all these variables from your generated configuration file
WIREGUARD_PUBLIC_KEY=key
WIREGUARD_PRIVATE_KEY=key
WIREGUARD_PRESHARED_KEY=key
WIREGUARD_ADDRESSES=ipv4

# Optional location variables, comma separated list, no spaces after commas, make sure it matches the config you created
SERVER_COUNTRIES=country
SERVER_CITIES=city 

# Heath check duration
HEALTH_VPN_DURATION_INITIAL=120s
```

### Testing Gluetun Connectivity
Once your containers are up and running, you can test your connection is correct and secured. This assumes you keep the `gluetun` container name. Learn more at the [gluetun wiki](https://github.com/qdm12/gluetun-wiki/blob/main/setup/test-your-setup.md).

> [!Note]
> If you run into issues try restarting the stack with `docker compose restart`.
```bash
docker run --rm --network=container:gluetun alpine:3.18 sh -c "apk add wget && wget -qO- https://ipinfo.io"
```
If you'd like to test Gluetun connectivity from a container using the service jump into the `docker compose exec` console and run the `wget` command below. Tested with `nzbget`, `qbittorrent`, and `prowlarr` containers. Ensure you open the ports through the the `gluetun` container.
```bash
docker exec -it container_name bash
wget -qO- https://ipinfo.io
```
### Passing Through Containers
When containers are in the same docker compose they all you need to add is a `network_mode: service:container_name` and open the ports through the the gluetun container. See example with a different torrent client below.
```yaml
services:
  gluetun:
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
If the container is in another docker `compose.yaml`, assuming Gluetun is already running add the following network mode. Ensure you open the ports through the the gluetun container.
```yaml
network_mode: "container:gluetun"
```

### Gluetun Proxmox LXC Setup

Errors like `cannot Unix Open TUN device file: operation not permitted` and `cannot create TUN device file node: operation not permitted` may happen if you're running this on LXC containers.

Find your container number, for example mine is 101

Edit `/etc/pve/lxc/101.conf` and add:
```
lxc.cgroup2.devices.allow: c 10:200 rwm
lxc.mount.entry: /dev/net dev/net none bind,create=dir
lxc.mount.entry: /dev/net/tun dev/net/tun none bind,create=file
```
Make sure you pass through the tun device (`/dev/net/tun:/dev/net/tun`) as shown in my compose file.

### Reduce Gluetun Ram Usage
As mentioned in this [issue](https://github.com/TechHutTV/homelab/issues/12) there is a [feature request](https://github.com/qdm12/gluetun/issues/765#issuecomment-1019367595) on the Gluetun Github page to help reduce ram usage. Gluetun bundles a recursive caching DNS resolver called `unbound` for handling domain name requests securely. Over time the cache size, which rests in RAM, can balloon to gigabytes.

You can do this by adding the following to your docker `compose.yaml` file under the `gluetun` environment variables.
```yaml
  gluetun:
    ...
    environment:
      - BLOCK_MALICIOUS=off # Disable unbound DNS resolver
```
This may not be an issue as [DNS over HTTPS in Go to replace Unbound](https://github.com/qdm12/gluetun/issues/137) is implemented, but it's worth the mention.

## Download Clients

### NZBGet

#### NZBGet Login Credentials 
The default credentials for NZBGet are a username of `nzbget` and a password of `tegbzn6789`. It's strongly recommended to change these default credentials for security reasons. This can be done under _Settings > SECURITY_, then change the ControlUsername and ControlPassword.

#### NZBGet Download Directories
If following the `/data:/data` directory scheme and used the command to setup the download directories open the qBittorent Web UI and do under _Settings > PATHS_ and change the paths.

_MainDir:_ `/data/downloads/nzbget`

_DestDir:_ `${MainDir}/completed`

_InterDir:_ `${MainDir}/intermediate`

And keep everything else as is.

#### Fix directory does not appear to exist inside the container error
This error may appear within Sonarr and Radarr. Once NZBGet is setup go to settings and under **INCOMING NZBS** change the **AppendCategoryDir** to **No**. This will prevent some potential mapping issues and save on unnecessary directories.

### qBittorrent

#### qBittorrent Login Credentials
When you first launch qBittorrent it will generate a random password. To find this password you can view the logs to see what the password is.
```bash
docker container logs qbittorrent
```
Now, go to your settings and setup a new username and password under _WebUI > Authentication_.

#### Qbittorrent Download Directories
If following the `/data:/data` directory scheme and used the command to setup the download directories open the qBittorent Web UI and do under _Settings > Downloads_ and change the paths.

_Default Save Path:_ `/data/downloads/qbittorrent/completed`

_Keep incomplete torrents in:_ `/data/downloads/qbittorrent/incomplete`

_Copy .torrent files to:_ `/data/downloads/qbittorrent/torrents`

#### qBittorrent Stalls with VPN Timeout
qBittorrent stalls out if there is a timeout or any type of interruption on the VPN. This is good because it drops connection, but we need it to fire back up when the connection is restored without manually restarting the container.

__Solution #1:__ Within the WebUI of qBittorrent head over to advanced options and select `tun0` as the networking interface. See image below for example.

![Set Network Interface to tun0](https://raw.githubusercontent.com/TechHutTV/homelab/refs/heads/main/media/images/qbittorrent_tun0.jpeg)

Next, I added `HEALTH_VPN_DURATION_INITIAL=120s` to my gluetun environment variables as [per this issue](https://github.com/qdm12/gluetun/issues/1832). I updated my `compose.yaml` above with this variable so you may already have this enabled. You can learn more about this on their [wiki](https://github.com/qdm12/gluetun-wiki/blob/main/faq/healthcheck.md). If you continue to have issues continue to next solution.

__Solution #2:__ Another solution, that can be used in conjunction with __Solution #1__ is using the [deunhealth](https://github.com/qdm12/deunhealth/tree/main) container to automatically restart qBittorrent when it gives an unhealthy status. We've added this to our `compose.yaml` for this stack.
```yaml
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

Next we need to add a health check and label to our `qbittorrent` container. We add `deunhealth.restart.on.unhealthy=true` as a label and a simple ping health check as shown below.

```yaml
  qbittorrent:
    image: lscr.io/linuxserver/qbittorrent:latest
    container_name: qbittorrent
    restart: unless-stopped
    labels:
      deunhealth.restart.on.unhealthy=true # Label added for deunhealth monitoring
    ...
```
Relevant Resources: [DBTech video on deunhealth](https://www.youtube.com/watch?v=Oeo-mrtwRgE), [gluetun/issues/2442](https://github.com/qdm12/gluetun/issues/2442) and [gluetun/issues/1277](https://github.com/qdm12/gluetun/issues/1277#issuecomment-1352009151)

## *arr Apps

When connecting your *arr applications be sure to use the new configured IP addresses in the `servarrnetwork`. We will soon update this section with more text documentation.
