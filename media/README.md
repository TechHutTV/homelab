# Self-Hosted Media Server and Aggregation

This is currently a work in progress. Please refer to the [Servarr Docker Setup](https://wiki.servarr.com/docker-guide) for more details on installing the stack.

## Data Directory
### Folder Mapping
It's good practise to give all containers the same access to the same root directory or share. This is why all containers in the compose file have the bind volume mount ```/media/data:/data```. It makes everything easier, plus passing in two volumes such as the commonly suggested /tv, /movies, and /downloads makes them look like two different file systems, even if they are a single file system outside the container. See my current setup below. 
```
data
├── books
├── downloads
│   ├── qbittorrent
│   │   ├── completed
│   │   ├── incomplete
│   │   └── torrents
│   └── nzbget
│       ├── complete
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
mkdir -p downloads/qbittorrent/{completed,incomplete,torrents} && mkdir -p downloads/nzbget/{complete,intermediate,nzb,queue,tmp}
```
### Network Share
I use to use a network share to store downloads and all my media. To do this I created the share with Unraid and added it to the fstab file within my Docker server.
```
//10.0.0.90/data /media/data cifs uid=1000,gid=100,username=user,password=password,iocharset=utf8 0 0
```
Storing the user creditentials within this file it's the best idea. Check out [this question](https://unix.stackexchange.com/questions/178187/how-to-edit-etc-fstab-properly-for-network-drive) on Stack Exchange to learn more.
## User Permissions
Using bind mounts (path/to/config:/config) may lead to permissions conflicts between the host operating system and the container. To avoid this problem, you we specify the user ID (PUID) and group ID (PGID) to use within the container. This will have your user permission to read and write configuration files, etc.

In this compose file I use PUID=1000 and PGID=1000, as that is generally the default ID's in most Linux systems, but depending on your setup you may need to chage this.

```
id your_user
uid=1000(techhut),gid=1003(techhut),groups=1000(techhut),988(docker)
```
In the example output above, I would need to edit the compose.yaml with gid=1003. If you are using a network share mounted though ```/etc/fstab``` match the permissions there. I use Unraid so my permissions are ```uid=1000(brandon),gid=100(brandon)```.

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

