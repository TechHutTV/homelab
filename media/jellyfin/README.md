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
Below are the plugins I'm currently using. I'd recommend checking out [Awesome Jellyfin](https://github.com/awesome-jellyfin/awesome-jellyfin) for much more.

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

### Live TV
This will require some extra hardware and a paid service for the guide data. HDHomeRun is great and it's what I use. Go to _Dashboard → Live TV → TV Sources → Add Tuner Device_. Select your tuner type _HDHomeRun_. Enter the tuner’s IP address and click Save.

Next you want to setup guide data. _Guide Providers → Add Guide Provider_. Schedules Direct is a paid service, but their awesome. It's a non-profit and they commit to the Jellyfin code directly. Create a account and you can use it free for a week. Input your username and password and give it some time to update the data. I didn't need to but you can map the channels by going to _Channels → Map Channels_.

### DVR
Under DVR we have a bunch of different settings to go over. 


https://forum.jellyfin.org/t-live-tv-recording-not-showing-in-recordings-tab

PULLED FROM REDDIT WILL BE EDITED DOWN
[https://www.reddit.com/r/jellyfin/comments/sz8tnq/a_much_more_detailed_guide_on_how_to_use_post/](https://www.reddit.com/r/jellyfin/comments/sz8tnq/a_much_more_detailed_guide_on_how_to_use_post/)

#### Recording Post Processing

Jellyfin supports Post Processing of recorded Live TV shows. This can be used to transcode the recording to a specific format that does not require transcoding on the fly when playing back, extract subtitles, remove commercials, and more.

> [!NOTE]
> There are several different ways to set up your post-processing script, and this largely will need to be changed to your individual use case.
>

>Described below is one way to do post processing, there may be other ways (other ways may be more efficient, too) to run your post-processor.

Mess around with this to change to your needs. Search around, post questions to the [Jellyfin Reddit Forum](https://www.reddit.com/r/jellyfin) or elsewhere, and others may be able to help. Logging is your friend! Make sure your script(s) logs adequately to a file or elsewhere in order to troubleshoot any issues you may encounter, as any output to stdout/stderror will not be seen in the Jellyfin logs.

#### Jellyfin Dashboard/DVR/Recording Post Processing Settings

Jellyfin runs the script you specify in the Admin Dashboard DVR settings with a command line parameter of the filepath automatically when recording finishes.

In Jellyfin Dashboard/DVR/Recording Post Processing settings:

Set "Post-processing application" to your shell script which calls your actual post processor (details of this 'actual' post processor script below). In this example, that would be `/path/to/run_post_process.sh`

Set "Post-processor command line arguments" to `"{path}"`.


With the settings above, the server executes this command when running the post processor:

````bash

"/path/to/run_post_process.sh" "\"/path/to/LiveTV/Shows/Series/Season/Episode.ts\""

````

#### Run Post Processor Shell Script (to be run directly by Jellyfin Server)

Quote interpretation is one of the hardest things to manage when using a post-processor script. Because of this, one easy way to run your post-processor is to have Jellyfin start a "runner" shell script, which then calls your actual post-processor script. This shell script then can be put into Jellyfin settings, and have a "clean" shell environment where it is easier to configure, look at logs, and more.

In the sample script below:

- Logging is enabled, and a logfile is created at some location accessible by your Jellyfin instance.

- The first command line argument, `$1` is written to the logfile (majorly for debugging purposes). This argument will be a path to the show to be post-processed. This argument is in the format of `/path/to/LiveTV/Shows/Series/Season/Episode.ts`

- The actual post processor Python script `record_post_process.py` is then called with s command line argument of the file name.

##### An example `run_post_processor.sh` script

[GitHub Gist link to `run_post_processor.sh`](https://gist.github.com/AndrewBreyen/0fc36c868486d48583a369b657e22c69)

````bash

#!/bin/sh

exec > "/path/to/logging/directory/logs/$(date +"%Y-%m-%d_%H-%M-%S")-run_post_process-sh.log" 2>&1

echo $1

/usr/local/bin/python3 /path/to/record_post_process.py "$1"

````

#### Post Processor Python Script (to be run by `run_post_processor.sh`)

In this example, a python script is where it all goes down. I chose to use Python primarilly because of how adaptable it is, and various third party extensions and packages that make it ideal for a post-processing script. This script can be customized to fit your individual requirements.

In the sample script:

- Logging is enabled, and a logfile is created at some location accessible by your Jellyfin instance.

- Command line arguments are checked, if no argument provided, script exits.

- Variables are determined for things such as the full non-transcoded file path, the basename, the file to be transcoded with extension, transcoded file name and path, and more.

- FFMPEG command is created and ran.

- In this example, the `h264_videotoolbox` video codec is used, and the audio is copied from source. Change the ffmpeg command to fit your requirements.

- Nontranscoded file is moved out of the Series/Season directory, into a folder not accessible by Jellyfin, called OLDFILES (This portion could also be configured to delete the non-transcoded file)

##### An example `record_post_process.py` script

This script is too much to post here, so a link to a GitHub Gist is provided.

Comments are listed that describes what each section does.

[GitHub Gist Link to `record_post_process.py`](https://gist.github.com/AndrewBreyen/1ac109bb485d8523e28fe98b3a222602)

#### Diving Deeper

Once you have post processing working in a basic format, there is loads more that can be done!

Some ideas:

- [Commercial Skipping with comskip](https://www.reddit.com/comments/jvzxnd/comment/hh6zwdn/)

- Post transcode progress to a Slack/Discord channel for notifications when transcoding starts/finishes 

## Remote Connections
wip
