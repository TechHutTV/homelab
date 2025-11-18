# My Homelab Setup
Homelab stacks, templates, and more fun resources! This is a *work in progress*. I still have a ton to update and add.

## Navigation
* [Apps](https://github.com/TechHutTV/homelab/tree/main/apps) - List of all the apps and services.
* [Automations](https://github.com/TechHutTV/homelab/tree/main/automations) - Making my life easier.
* [Home Assistant](https://github.com/TechHutTV/homelab/tree/main/homeassistant) - Smart home services and automation.
* [Media Server](https://github.com/TechHutTV/homelab/tree/main/media) - Plex, Jellyfin, *arr stack, and more.
* [Server Monitoring](https://github.com/TechHutTV/homelab/tree/main/monitoring) - Graphs and Visualizations for Unraid, Proxmox, and more.
* [Surveillance System](https://github.com/TechHutTV/homelab/tree/main/surveillance) - Frigate NVR Solution with Coral TPU.
* [Storage](https://github.com/TechHutTV/homelab/tree/main/storage) - Current Storage and Backup Solution.
* [Proxy Managment](https://github.com/TechHutTV/homelab/tree/main/proxy) - NGINX Proxy Manager, DDNS with Cloudflare, Local Domains, and more.

## Hardware
wip

### Compute
#### OpnNAS SL-R12 Pro (Unriad)
The model I have is a early release press model. I'm not quite sure when they will be ready for sale, but you can learn more [here](https://opnnas.com/#hls). This machine is running Unraid and acts as my main media server. Also includes a Ubuntu VM running NextCloud.
* [13th Gen Intel® Core™ i5-13500T](https://amzn.to/4qblt9y)
* [Crucial Pro 96GB DDR5 RAM](https://amzn.to/3Jbh2uD)
* [x5 Seagate Exos 20TB](https://amzn.to/4nPhn5d) 
* [x7 Seagate IronWolf 4TB](https://amzn.to/3W1IFt5)

#### Miniforum AI X1 Pro (Proxmox)
This acts as my Proxmox Node. Running a varity of services including; graphana, npm, n8n, and another other non-media related instances.
[get it here](https://amzn.to/4n0xTOR)
* AMD Ryzen AI 9 HX 370
* 96GB SODIMM DDR5
* [x2 Western Digital SN580 1TB](https://amzn.to/4hdfHzY) (Mirrored Boot Drives)


### Networking
Now, Ubiquiti sent all this over to me. There is no way I'd be able to afford all this stuff on my own dime, so note that. This setup is crazy overpowered for what I need.

#### UDM Pro Max
#### USW Pro Max 24 PoE
#### USW Pro XG 10 PoE
#### x2 U7 Pro XG
#### USW Lite 16 PoE

Other Unifi Devices
* 2 x 8TB-HDDs
* 1 x U-Rack-6U-TL
* 1 x UACC-Cable-Patch-EL-0.15M-W-24
* 5 x UACC-Cable-Patch-EL-3M-W
* 2 x UACC-Cable-Patch-EL-15M-W

### Security
Currently have two different systems as I was fully on Reolink for sometime, but since Ubiquiti sent over some equipment to try we currently have a mix. 

[https://www.youtube.com/watch?v=2RiiKi4vjJc](https://www.youtube.com/watch?v=2RiiKi4vjJc)

#### Reolink NVR Kit
This is great if you're trying to save money and have a simple PoE plug and play solution. This does add difficulties to manage as all the streams are under a single device and kit camera NEED the NVR to function. Some kit cameras may have more features so check the specific features of the camera model in the kit. You can probably find a great deal used. Cameras are the B400s.
* [REOLINK 8CH 5MP](https://amzn.to/4dulGgG)
* [REOLINK 16CH 12MP](https://amzn.to/4d9YaWz) (my kit)

#### Reolink PoE
I've had no issues with these. Great for the Reolink ecosystem and using 3rd party services such as Frigate.
* [Reolink RLC-1224A 12MP UHD PoE](https://amzn.to/4d6BuGJ)
* [Reolink 4K Floodlight](https://amzn.to/4ceRKnM)


#### x3 UVC-G6-Turret-W
These are running hooked up to the Dream Machine Pro Max. I like it enough I will be slowly switching to Unifi stuff as I can afford it.
