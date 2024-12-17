# Storage and Backup
In this reposity I will layout my storage and backup solutions for all of the services and platforms running on my homelab. Currently, I have two main platforms that manage my storage; Proxmox and Unraid. Below I will explain their uses and how I created and manage them.

[please see this video](https://youtu.be/zLFB6ulC0Fg)

## Proxmox
Proxmox is responible for managaging my storage for video, images, logs, databases, and docker configurations. This is done on a network attached storage device with six 4TB HHDs and three 1TB NVME SSDs. These are combined into seperate ZFS pools for the HDDs (vault) and the SSDs (flash). Vault is used as a large data storage pool and Flash is used for containers and virtual machines disks.

__This is a work in progress__

### 1. Create ZFS Pools

First, we are going to setup two ZFS Pools. A "Vault" pool which is used for backups and shared data such as media and downloads. We also will make a "Flash" pool which is used for virtual machines and container storage. In my case I have an NVMe drive for my flash storage that will contain my containers and VM disks, and four hard drives for shared storage. To do this, on the Proxmox sidebar for your datacenter, go to Disks -> ZFS -> Create: ZFS. This will pop up the screen to create a ZFS pool.

![create zfs](https://github.com/TechHutTV/homelab/blob/main/storage/createzfs.png)

From this screen, it should show all of your drives, so select the ones you want in your pool, and select your RAID level (in my case RAIDZ for my vault pool and mirror for my flash pool) and compression, (in my case lz4). 

![create zfs dialog](https://github.com/TechHutTV/homelab/blob/main/storage/createzfsdialog.png)

2. Assigning Pools to Primary Vault Container

Now we want to assign our pools to certain tasks in Proxmox. So in the storage tab of our data center, we can add directories to our pools used for certain tasks. To do this go to Datacenter -> Storage -> Add -> Directory, and from here you can set the ID, where on the proxnox server, and the content.

![assigning pool](https://github.com/TechHutTV/homelab/blob/main/storage/assigningpool.png)

For me, I first made a directory for backups inside the vault. For the directory I made sure it was inside my /vault pool by setting the directory to /vault/backups, and than set the content to VZDump backup file for my backups.

![backup pool](https://github.com/TechHutTV/homelab/blob/main/storage/backuppool.png)

I also edited my local storage to not contain my VZDump backup files.

Next up I create a directory for my data which will contain my disk images and containers. So I created a new directory called "data" inside /vault/data, and here I set the content to disk image, and containers.

![data pool](https://github.com/TechHutTV/homelab/blob/main/storage/datapool.png)

Finally, I want a disks directory for my virtual machines on my NVMe drives. So, I made another "disks" pool inside /flash/disks, and just like data, I set the content to disk images and containers.

![disks pool](https://github.com/TechHutTV/homelab/blob/main/storage/diskspool.png)

3. Adding Virtual Drives (zfs mount points) in Pool to Other Containers

Now, I

4. Setup Cockpit for Shares

Steps above are based on [this video](https://www.youtube.com/watch?v=zLFB6ulC0Fg), when creating the data directory in that video I [changed how it's setup](https://www.youtube.com/watch?v=ObgzcKlozWQ).

* Proxmox: [https://www.proxmox.com/en/downloads](https://www.proxmox.com/en/downloads)
* Cockpit: [https://github.com/cockpit-project/cockpit](https://github.com/cockpit-project/cockpit)
* Cockpit Sharing: [https://github.com/45Drives/cockpit-file-sharing](https://github.com/45Drives/cockpit-file-sharing)
* Cockpit Identities: [https://github.com/45Drives/cockpit-identities](https://github.com/45Drives/cockpit-identities)
* Cockpit Navigator: [https://github.com/45Drives/cockpit-navigator](https://github.com/45Drives/cockpit-navigator)
* ZFS on Linux: [https://pve.proxmox.com/wiki/ZFS_on_Linux](https://pve.proxmox.com/wiki/ZFS_on_Linux)
* ZFS in Proxmox: [https://www.reddit.com/r/Proxmox/comments/jppohv/a_very_short_guide_into_how_proxmox_uses_zfs/](https://www.reddit.com/r/Proxmox/comments/jppohv/a_very_short_guide_into_how_proxmox_uses_zfs/)

5. Adding network shares to unprivalized containers
6. [https://www.youtube.com/watch?v=DMPetY4mX-c](https://www.youtube.com/watch?v=DMPetY4mX-c)
