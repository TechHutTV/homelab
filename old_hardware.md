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
