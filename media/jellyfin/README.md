# Jellyfin Setup Guide
I run Jellyfin in Docker directly on Unraid alongside Plex. The current setup uses the shared Intel Arc Pro B60 for hardware acceleration and has writable media storage for DVR recordings. Below are my Unraid settings, followed by portable Docker and Proxmox examples.

## Current Unraid Setup

| Setting | My Unraid configuration |
|---|---|
| Host | `HopkiMedia`, `10.173.10.10` |
| Container | `Jellyfin` |
| Web interface | `http://10.173.10.10:8096` |
| Reverse proxy | Your domain, for example `https://jellyfin.example.com` |
| Application user / group | UID `1000`, GID `100` (`brandon:users`) |
| Supplemental GPU group | GID `18` |
| Media / DVR mapping | Read/write |
| GPU | Intel Arc Pro B60, `/dev/dri/renderD128` |

In the Unraid Docker template, I saved the following under **Extra Parameters** so recreating the container keeps the user and GPU group settings:

```text
--user=1000:100 --group-add=18
```

Keep the existing appdata and media path mappings, set the media/DVR mapping to **Read/Write**, and retain the `/dev/dri` device mapping. The recording directory you select in Jellyfin must be inside that writable media mapping. Check the configured user, group, and device access from the Unraid terminal:

```bash
docker inspect Jellyfin --format '{{.Config.User}} {{json .HostConfig.GroupAdd}}'
docker exec Jellyfin id
docker exec Jellyfin ls -l /dev/dri
```

My IDs are specific to this host. If you're adapting the setup, match your media ownership and GPU device's group. The [LinuxServer.io Compose example](compose.yaml) below uses that image's `PUID` and `PGID` settings, as described in its [image documentation](https://docs.linuxserver.io/images/docker-jellyfin/#user-group-identifiers).

> [!NOTE]
> If you're using Proxmox for Jellyfin, the VM and LXC examples below cover that alternative. My current container runs directly on Unraid.

## Data Directory
### Folder Mapping
It's good practice to give all media containers access to the same root directory or share. The portable Compose example uses `/data:/data:rw`, keeping the same container paths as the Servarr stack and allowing DVR writes. Here's an example folder layout for a separate Docker host:
```
data
├── movies
├── music
└── shows
docker
└── jellyfin
    ├── config
    └── jellystat
```

### Network Share (VM, Optional)
If Jellyfin runs in a separate VM, mount your media share into that VM before starting the container. On my Unraid host, the container uses the local media share directly, so these SMB steps aren't needed.

> [!IMPORTANT]
> This assumes you already have a working Samba share on your server. If you haven't set that up yet, follow the [SMB share guide](https://github.com/TechHutTV/homelab/tree/main/storage#creating-smb-shares) first. If you get `mount error(113): No route to host`, verify the Samba server is running and reachable from your VM.

Within the VM install `cifs-utils`
```bash
sudo apt install cifs-utils
```

Create a credentials file so your password isn't stored in plain text in fstab:
```bash
sudo nano /etc/samba/credentials
```
```
username=youruser
password=yourpassword
```
```bash
sudo chmod 600 /etc/samba/credentials
```

Create the mount point and edit fstab:
```bash
sudo mkdir -p /data
sudo nano /etc/fstab
```
```
//10.0.0.100/data /data cifs x-systemd.automount,uid=1000,gid=1000,credentials=/etc/samba/credentials,iocharset=utf8 0 0
```

> [!TIP]
> The `x-systemd.automount` option mounts the share on first access instead of at boot. This prevents `mount error(111): Connection refused` errors caused by fstab running before the network is ready.

Now reload the configuration and mount the shares with the following commands.
```bash
sudo systemctl daemon-reload
sudo mount -a
```

## User Permissions
For the LinuxServer.io image in the Compose example, set `PUID` and `PGID` to the user and group that can read and write your bind-mounted directories. The example defaults to `1000:1000`; my current Unraid template settings are listed [above](#current-unraid-setup).

On a separate Linux host, check your IDs and the mounted directory's ownership:

```bash
id your_user
ls -ld /data
```

If you're using the SMB mount above, set its `uid` and `gid` options to the intended application user and group too. Reading movies successfully doesn't establish write access, so check that Jellyfin can create a file in the chosen recording directory before relying on DVR.

## Installation
I use the Unraid Docker template for my current deployment. The examples below cover LinuxServer.io Docker on another host and a package installation for an existing Ubuntu or Proxmox guest.

### Docker Setup (Recommended)
The [Compose file](compose.yaml) uses the LinuxServer.io image with a separate `jellystat` database and application. Match the media and configuration paths to your Docker host.

```yaml
services:
  jellyfin:
    image: lscr.io/linuxserver/jellyfin:latest
    container_name: jellyfin
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=America/Los_Angeles
      # Uncomment and set the address clients use to reach this host.
      # - JELLYFIN_PublishedServerUrl=http://YOUR_SERVER_IP:8096
    volumes:
      - ./config:/config
      - /data:/data:rw
    devices:
      - /dev/dri:/dev/dri #Use for Intel QuickSync
    ports:
      - 8096:8096
      - 7359:7359/udp #Service Discovery
      - 1900:1900/udp #Client Discovery
    restart: unless-stopped
```

> [!TIP]
> **Autodiscovery not working?** Docker's port forwarding doesn't handle UDP broadcast well, so clients may not automatically find your Jellyfin server. You can always connect manually via `http://SERVER-IP:8096`. If you want autodiscovery to work, switch to host networking by replacing the `ports` section with `network_mode: host`.

### System Installation (NOT Recommended)
Run the following command on your Ubuntu system, VM, or Proxmox LXC. You can learn about how to verify the script integrity [here](https://jellyfin.org/docs/general/installation/linux/).
```bash
curl https://repo.jellyfin.org/install-debuntu.sh | sudo bash
```

#### Permissions
If you're running this with docker, you can skip these steps!

Stop the Jellyfin service.
```bash
sudo systemctl jellyfin stop
```
Edit the Jellyfin service configuration to reflect your user.
```bash
sudo nano /lib/systemd/system/jellyfin.service
```
Now change the `User` and `Group` settings to your main user. You can run the `id` command if you're unsure.
```ini
[Unit]
Description = Jellyfin Media Server
After = network-online.target

[Service]
Type = simple
EnvironmentFile = /etc/default/jellyfin
User = brandon
Group = brandon
WorkingDirectory = /var/lib/jellyfin
ExecStart = /usr/bin/jellyfin $JELLYFIN_WEB_OPT $JELLYFIN_FFMPEG_OPT $JELLYFIN_SERVICE_OPT
Restart = on-failure
TimeoutSec = 15
SuccessExitStatus=0 143

[Install]
WantedBy = multi-user.target
```
Now change the permissions of the Jellyfin files and folders
```bash
sudo chown -R brandon:brandon /etc/default/jellyfin
sudo chown -R brandon:brandon /usr/bin/jellyfin
sudo chown -R brandon:brandon /var/lib/jellyfin/
sudo chown -R brandon:brandon /etc/jellyfin/
sudo chown -R brandon:brandon /var/log/jellyfin/
sudo chown -R brandon:brandon /var/cache/jellyfin/
sudo chown -R brandon:brandon /usr/share/jellyfin
sudo chown -R brandon:brandon /usr/share/jellyfin-ffmpeg
sudo chown -R brandon:brandon /usr/lib/jellyfin/
sudo chown -R brandon:brandon /usr/lib/jellyfin-ffmpeg/
```
Reload the daemon and restart jellyfin
```bash
sudo systemctl daemon-reload
sudo systemctl restart jellyfin
```
Check the user that is running Jellyfin
```bash
ps -aux | grep jellyfin
```
Source: [https://github.com/tteck/Proxmox/discussions/286](https://github.com/tteck/Proxmox/discussions/286)

#### Hardware Transcoding
For the System Installation only, install the `jellyfin-ffmpeg7`. Remove the deprecated jellyfin meta package if it breaks the dependencies.
```bash
sudo apt update && sudo apt install -y jellyfin-ffmpeg7
```
Continue with the steps below...

## Hardware Transcoding
This focuses on Intel Quick Sync and VA-API. For the supported configuration steps, see Jellyfin's [Intel GPU guide](https://jellyfin.org/docs/general/post-install/transcoding/hardware-acceleration/intel/).

### Unraid (Current Setup)

My Arc Pro B60 is `/dev/dri/renderD128`; the host's UHD770 is `/dev/dri/renderD129`. In Jellyfin's transcoding settings, select the Arc device for Intel QSV or VA-API. The container keeps its `/dev/dri` access and supplemental group `18` through the Unraid template settings above. Verify device identities on your own host before choosing a render node.

Plex, Jellyfin, and the Immich ML worker share the Arc. It stays available to all three containers without PCI passthrough to a VM. Their workloads can still compete for GPU time. A VA-API query succeeded as Jellyfin's application user during maintenance; a full live-transcoding or simultaneous-load test wasn't performed.

### Proxmox Passthrough

> [!NOTE]
> Running Jellyfin with Docker on a VM is highly recommended. This eliminates permission issues with running Jellyfin on the system and running Docker on a VM is what is recommended by the Proxmox team.
>

#### Running on a VM (Recommended)
In the Proxmox UI, under your virtual machine, click the **Hardware** option in the sidebar. From there select _Add > PCI Device_. Then select **Raw** and pick the device that we will use for Quicksync or another GPU if you're not using Quicksync. For Quicksync it's often the very first Intel device that will say something like "Alderlake" in the name.

#### Running on an Unprivileged LXC
If you're running Jellyfin directly on the LXC that houses all your media, you will need to manually add the following to you LXC configuration. Add the lines below to the configuration of your container below. Don't forget to change the ID to match the container you've installed Jellyfin on.
```bash
nano /etc/pve/lxc/100.conf
```
```
#Add these for Intel QuickSync
dev0: /dev/dri/card0,gid=44
dev1: /dev/dri/renderD128,gid=104
```
### Ubuntu Setup
The following steps are necessary when running Jellyfin on an Ubuntu server, virtual machine or Proxmox LXC. Add user `jellyfin` and the user you're running jellyfin as to the `render` group.
```bash
sudo usermod -aG render jellyfin
sudo usermod -aG render brandon # since I'm running jellyfin as my user
sudo systemctl restart jellyfin
```
Now we can confirm hardware transcoding is ready by installing the `intel-gpu-tools` package and running the command `intel_gpu_top`.
```bash
sudo apt install intel-gpu-tools
intel_gpu_top
```

## Configuring Jellyfin
Open your web browser and navigate to your installed instance of Jellyfin using `http://IP:8096` and once there you can power through the initial setup by selecting your preferred language, then create an admin account with a secure username and password. Next, set up your media libraries by adding folders for movies, TV shows, or music. I tend to keep everything in my `/data` directory as shown in the media page on this repo.

## Plugins
Below are the plugins I'm currently testing. I'd recommend checking out [Awesome Jellyfin](https://github.com/awesome-jellyfin/awesome-jellyfin) for much more.

1. [Intro Skipper](https://github.com/intro-skipper/intro-skipper)
2. [Fan Art](https://github.com/jellyfin/jellyfin-plugin-fanart)
3. [Jellyfin Newsletter](https://github.com/Cloud9Developer/Jellyfin-Newsletter-Plugin)
4. [TMDb Box Sets](https://github.com/jellyfin/jellyfin-plugin-tmdbboxsets)
5. [TheTVDB](https://github.com/jellyfin/jellyfin-plugin-tvdb)
6. [SkinManager](https://github.com/danieladov/jellyfin-plugin-skin-manager)
7. [Media Bar](https://github.com/IAmParadox27/jellyfin-plugin-media-bar)

## Other Tools
Now to expand the functionality of Jellyfin I recommend these 3rd party tools that integrate well with Jellyfin.

### Jellystat
Find it [here](https://github.com/CyferShepard/Jellystat)

#### PostgreSQL 18 persistent storage

For a fresh installation, both Compose examples mount `./jellystat/postgres-data` at `/var/lib/postgresql`. PostgreSQL 18 stores its default cluster in `/var/lib/postgresql/18/docker`, so the cluster lives under `./jellystat/postgres-data/18/docker` on the Docker host. See the [official PostgreSQL image documentation](https://hub.docker.com/_/postgres).

> [!CAUTION]
> **Existing installations:** Inspect the current container before recreating or removing it. Changing the mount destination does not move existing data or upgrade an older PostgreSQL cluster. Do not delete the old container's volumes or data directories until you have identified the active cluster and verified a backup by restoring it separately.

Run these commands on the Docker host while the existing database is running (adjust the database username if you changed it):

```bash
docker inspect jellystat-db --format '{{json .Mounts}}'
docker exec jellystat-db psql -U postgres -d postgres -c 'SHOW data_directory;'
```

Match the reported data directory to the container's mounts and record the actual host directory or Docker volume holding the cluster. Back up that database with a supported database-aware procedure; copying a running PostgreSQL directory is not a consistent backup. If the database cannot start, preserve its storage and inspect its logs and previous image/mount configuration before attempting recovery. For PostgreSQL 15, retain the original `/var/lib/postgresql/data` mount while preparing the major-version migration below.

After deploying the corrected PostgreSQL 18 layout, confirm `SHOW data_directory` reports `/var/lib/postgresql/18/docker` and the host directory contains `18/docker/PG_VERSION`. Verify that representative Jellystat data survives container recreation without deleting persistent storage, and that a backup from the corrected deployment restores to a separate instance.

#### Upgrading the Jellystat Database (Postgres 15 → 18)

The Compose example uses PostgreSQL 18. Existing PostgreSQL 15 databases need a major-version migration; changing the image tag or mount path does not upgrade the data. Follow the [official PostgreSQL upgrade guide](https://www.postgresql.org/docs/18/upgrading.html) and the [Docker image storage instructions](https://hub.docker.com/_/postgres). Fresh installations can skip this migration.

Before the final database dump, stop Jellystat and any other database writers. Keep an off-host copy of the dump, the original data directory, and the original image/configuration until you have verified the restore and checked Jellystat's users and playback history. Restore into a separate PostgreSQL 18 directory so the original cluster remains available for recovery.

If you need to revert, use the original PostgreSQL 15 image and data directory with the original Jellystat version. Any changes collected after switching to PostgreSQL 18 will not be present in that older database.

## DVR and Live TV

### Live TV
This will require some extra hardware and a paid service for the guide data. HDHomeRun is great and it's what I use. Go to _Dashboard → Live TV → TV Sources → Add Tuner Device_. Select your tuner type _HDHomeRun_. Enter the tuner’s IP address and click **Save**.

Next you want to setup guide data. _Guide Providers → Add Guide Provider_. Schedules Direct is a paid service, but they're awesome. It's a non-profit and they commit to the Jellyfin code directly. Create an account and you can use it for free for a week. Input your username and password and give it some time to update the data. I didn't need to but you can map the channels by going to _Channels → Map Channels_.

### DVR

Choose a recording directory inside the container's writable media mapping. For example, if your media is mounted at `/data`, you could use `/data/recordings` after creating that directory with ownership that allows the application user to write to it. Use the container path in Jellyfin's DVR settings.

On my Unraid host, the media/DVR mapping is writable and an actual write check passed as UID `1000`, GID `100`. This confirms the storage permission fix; a complete scheduled recording still needs to be checked separately.

## Remote Connections

> [!NOTE]
> This is a work in progress.

In the meantime checkout [this](https://youtu.be/79e6KBYcVmQ?si=0yTkdD5UtDVs-jNX) video.
