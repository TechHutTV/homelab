# My Homelab Setup
Homelab stacks, templates, and more fun resources! This is a *work in progress*. I still have a ton to update and add.

## Navigation
* [Apps](https://github.com/TechHutTV/homelab/tree/main/apps) - List of all the apps and services.
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
