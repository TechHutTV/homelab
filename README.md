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

## Old Setup

![counter top lab](https://github.com/TechHutTV/homelab/blob/main/storage/homelab.png)

### Servers and NAS
#### ZimaCube Pro (Proxmox)
This machine is running our Proxmox Server. Handling our media storage, arr services, proxy, home assistant, monitoring, and many others.
* 12th Gen Intel Core i5 1235U
* [32GB SODIMM DDR5](https://amzn.to/3Um77VB)
* 256GB NVMe SSD (Boot Drive)
* x3 [T-Create 1TB 2280 NVMe](https://amzn.to/4dGvwMx) (Flash Pool raidz1)
* x6 [Western Digital 4TB WD Red Plus](https://amzn.to/4gCpv6d) (Vault Pool raidz1)

#### UGREEN DXP6800 Pro (Proxmox)
This machine is running Proxmox. Due to a free PCIe slot this has an additional Coral TPU and I'm currently testing it with Blue Iris. Running Immich for personal photo backups and MeTube to pull YouTube videos for use in media projects. This machines handles the network shares that connect to our Nextcloud instance.
* 12th Gen Intel Core i5 1235U
* [16GB SODIMM DDR5](https://amzn.to/3YqeZ9A)
* [DataTraveler 3.0 32GB USB](https://amzn.to/4fovZUR) (Boot Drive)
* x1 [Seagate IronWolf 16TB NAS HDD](https://amzn.to/4eNPXYv) (Parity Drive)
* x5 [Seagate IronWolf 4TB NAS HDD](https://amzn.to/3Y6eyT7) (Raid5 Pool)
* x2 [T-Create 1TB 2280 NVMe](https://amzn.to/4dGvwMx) (Cache)

#### Intel NUC (Ubuntu)
This NUC has a base install of Ubuntu 24.04 with docker. This machine is dedicated to business services including Odoo and Nextcloud.
* 13th Gen Intel i7-1360P
* [32GB SODIMM DDR5](https://amzn.to/3Um77VB)
* [Intel 670p Series M.2 2280 512GB](https://amzn.to/40fOiHn) (Boot Drive)

### Networking
#### Omada Short Stack
* [TP-Link Omada Hardware Controller](https://amzn.to/48vW6H1)
* [TP-Link Omada Multi-Gigabit VPN Router](https://amzn.to/4hmvgoD)
* [TP-Link Jetstream 8 Port Gigabit Smart Managed PoE Switch](https://amzn.to/4hxKqI2)

#### Other
* [TP-Link EAP610 Ultra-Slim Wireless Access Point](https://amzn.to/4e2clNc)
* [TP-Link EAP653 Wireless Access Point](https://amzn.to/48pY8IT)
* [TP-Link TL-SG2016P 16 Port Gigabit Smart Managed PoE Switch](https://amzn.to/3BYq5uV)
