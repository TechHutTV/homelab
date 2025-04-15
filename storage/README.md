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

**NOTE: CHECK THE ADD TO STORAGE BOX**

![create zfs dialog](https://github.com/TechHutTV/homelab/blob/main/storage/createzfsdialog.png)

2. Assigning Pools to Primary Vault Container

Process chaged (WIP)

4. Adding Virtual Drives (zfs mount points) in Pool to Other Containers

Process chaged (WIP)

4. Creating SMB Shares

Create your share director and set permissions 
```
sudo mkdir /data
sudo chmod 777 /data
```
Install Samba
```
sudo apt install samba
```
Edit the samba config
```
sudo nano /etc/samba/smb.conf
```
Add this to the bottom on the configuration
```
[share]
path = /data
browseable = ues
read only = no
guest ok = no
```
Add your samba user
```
sudo smbpasswd -a [username]
```
Set services to auto start on startup
```
sudo systemctl enable smbd
sudo systemctl enable nmbd
Restart samba services
sudo systemctl restart smbd
sudo systemctl restart nmbd
```
