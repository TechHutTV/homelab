# Jellyfin Setup Guide
Welcome to the ultimate Jellyfin setup guide. 

> [!NOTE]
> For simplicity and compatibility it is recommended to install Jellyfin with Docker in a __virtual machine__ if you're running Proxmox. See more information [here](https://pve.proxmox.com/pve-docs/pve-admin-guide.html#chapter_pct).
>

## Data Directory
### Folder Mapping
It's good practice to give all containers the same access to the same root directory or share. This is why all containers in the compose file have the bind volume mount `/data:/data`. It makes everything easier, plus passing in two volumes such as the commonly suggested `/tv`, `/movies`, and `/downloads` makes them look like two different file systems, even if they are a single file system outside the container. See my current setup below.
```
data
├── movies
├── music
└── shows
docker
└── jellyfin
    ├── config
    ├── jellyseerr
    └── jellystat
```

### Network Share (VM)
I generally install Docker on the same LXC that I have my media server on as well as all my data. This, however, is [not recommended by Proxmox](https://www.reddit.com/r/Proxmox/comments/1afslhs/should_i_use_lxc_or_vm_for_running_docker/). Going forward you should create a separate VM for all your docker containers and mount the data directory we created in the storage guide with the share. 

Within the VM install `cifs-utils`
```bash
sudo apt install cifs-utils
```
Now, edit the `fstab` file and add the following lines editing them to match your information.
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
uid=1000(brandon),gid=1003(brandon),groups=1000(data-share),988(docker)
```
In the example output above, if using a network share I would need to edit the `compose.yaml` with `PGID=1003`. If you are using a network share mounted though `/etc/fstab` match the permissions there. I use Cockpit with a custom group for shares so my permissions are `uid=1000(brandon),gid=1000(data-share)`.
If you run into errors after creating all the folders you can assign the permissions using `chown`. For example:
```bash
sudo chown -R 1000:1000 /data
sudo chown -R 1000:1000 /docker
```

## Installation
There are two options for installing Jellyfin. Both work great and it's all a matter of preference. I generally install Jellyfin directly on the LXC within Proxmox that contains all my data.

### Docker Setup (Recommended)
Docker is another option to install and run Jellyfin. Check out the `compose.yaml` file or the full stack.

```yaml
services:
  jellyfin:
    image: lscr.io/linuxserver/jellyfin:latest
    container_name: jellyfin
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=America/Los_Angeles
      - JELLYFIN_PublishedServerUrl=http://10.0.0.101 #optional
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
This focuses on transcoding with Intel QuickSync. In my experience it is simply the best option. If you're running a AMD CPU you can pickup a Intel Arc GPU fairly cheap. If you have any issues or don't have access to a Intel CPU or an Arc GPU be sure to checkout the official docs [here](https://jellyfin.org/docs/general/administration/hardware-acceleration/). If you're not doing this on Proxmox you can skip to the Ubuntu setup.

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

### Jellyseerr
Find it [here](https://github.com/fallenbagel/jellyseerr)

## DVR and Live TV

### Live TV
This will require some extra hardware and a paid service for the guide data. HDHomeRun is great and it's what I use. Go to _Dashboard → Live TV → TV Sources → Add Tuner Device_. Select your tuner type _HDHomeRun_. Enter the tuner’s IP address and click **Save**.

Next you want to setup guide data. _Guide Providers → Add Guide Provider_. Schedules Direct is a paid service, but they're awesome. It's a non-profit and they commit to the Jellyfin code directly. Create an account and you can use it for free for a week. Input your username and password and give it some time to update the data. I didn't need to but you can map the channels by going to _Channels → Map Channels_.

### DVR

> [!NOTE]
> This is a work in progress. Don't quite have it working yet.
>

## Remote Connections

> [!NOTE]
> This is a work in progress.

In the meantime checkout [this](https://youtu.be/79e6KBYcVmQ?si=0yTkdD5UtDVs-jNX) video.
