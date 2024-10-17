# Storage and Backup
In this reposity I will layout my storage and backup solutions for all of the services and platforms running on my homelab. Currently, I have two main platforms that manage my storage; Proxmox and Unraid. Below I will explain their uses and how I created and manage them.

[please see this video](https://youtu.be/zLFB6ulC0Fg)

## Proxmox
Proxmox is responible for managaging my storage for video, images, logs, databases, and docker configurations. This is done on a network attached storage device with six 4TB HHDs and three 1TB NVME SSDs. These are combined into seperate ZFS pools for the HDDs (vault) and the SSDs (flash). Vault is used as a large data storage pool and Flash is used for containers and vurtual machines disks.

__This is a work in progress__

1. Create ZFS Pools
2. Assigning Pools to Primary Vault Container
3. Adding Virutal Drives in Pool to Other Containers
4. Setup Cockpit for Shares

* Proxmox: [https://www.proxmox.com/en/downloads](https://www.proxmox.com/en/downloads)
* Cockpit: [https://github.com/cockpit-project/cockpit](https://github.com/cockpit-project/cockpit)
* Cockpit Sharing: [https://github.com/45Drives/cockpit-file-sharing](https://github.com/45Drives/cockpit-file-sharing)
* Cockpit Identities: [https://github.com/45Drives/cockpit-identities](https://github.com/45Drives/cockpit-identities)
* Cockpit Navigator: [https://github.com/45Drives/cockpit-navigator](https://github.com/45Drives/cockpit-navigator)
* ZFS on Linux: [https://pve.proxmox.com/wiki/ZFS_on_Linux](https://pve.proxmox.com/wiki/ZFS_on_Linux)
* ZFS in Proxmox: [https://www.reddit.com/r/Proxmox/comments/jppohv/a_very_short_guide_into_how_proxmox_uses_zfs/](https://www.reddit.com/r/Proxmox/comments/jppohv/a_very_short_guide_into_how_proxmox_uses_zfs/)
