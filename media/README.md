# Work in Progress

## User Permissions
Using bind mounts (path/to/config:/config) may lead to permissions conflicts between the host operating system and the container. To avoid this problem, you we specify the user ID (PUID) and group ID (PGID) to use within the container. This will have your user permission to read and write configuration files, etc.

In this compose file I use PUID=1000 and PGID=1000, as that is generally the default ID's in most Linux systems, but depending on your setup you may need to chage this.

```
id your_user
uid=1000(techhut),gid=1003(techhut),groups=1000(techhut),988(docker)
```
In the example output above, I would need to edit the compose.yaml with gid=1003.

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

### Testing Other Containers
Jump into the Exec console and run the wget command below. Tested with nzbget, deluge, and prowlarr. Ensure you open the ports through the the gluetun container.
```
docker exec -it conatiner_name bash
wget -qO- https://ipinfo.io
```

