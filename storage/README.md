# Storage and Backup
In this repo I will layout my storage and backup solutions for all of the services and platforms running on my homelab. Currently, I manage everything with Proxmox and Proxmox Backup Server. While soultions like Unraid and TrueNAS are awesome, I have found over the years the Proxmox is actually an amazing solution for managing storage, network shares, and backups.

## Proxmox as a NAS
My current setup involves a single server with x3 NVME drives and a bunch of harddrives in a ZFS configuration. These are combined into seperate ZFS pools for the HDDs (vault) and the SSDs (flash). Vault is used as a large data storage pool and Flash is used for containers and virtual machines disks. No mattery your configuratiuon you can follow this guide. However, I would recommend at least one NVME SSD for your boot drive, and at least 512gb if you don't have any other NVME SSDs and at least x2 HHDs for file storage.

### 1. Post Install Steps (optional)

Removing Proxmox Notice 
```
cd /usr/share/javascript/proxmox-widget-toolkit/
cp proxmoxlib.js proxmoxlib.js.bak # make a backup
nano proxmoxlib.js
```
Edit the following line by adding `void({ //`
```
# Before
Ext.Msg.show({
  title: gettext('No valid subscription'),
# After
void({ //Ext.Msg.show({
  title: gettext('No valid subscription'),
```
Now restart the Proxmox web server
```
systemctl restart pveproxy.service
```
### 2. Create ZFS Pools

First, we are going to setup two ZFS Pools. A "Vault" pool which is used for larger stored data sets such as media, images and archives. We also will make a "Flash" pool which is used for virtual machine and container root file systems. To do this, on the Proxmox sidebar for your datacenter, go to Disks -> ZFS -> Create: ZFS. This will pop up the screen to create a ZFS pool.

From this screen, it should show all of your drives, so select the ones you want in your pool, and select your RAID level (in my case RAIDZ for my vault pool and mirror for my flash pool) and compression, (in my case lz4). Make sure you check the box that says **Add to Storage**. This will make the pools immiatily avalible and will prevent using .raw files as obsosed to my previous setup when I added directorties. 

## 3. Creating Containers using ZFS Pools
Work in progress

## 4. Creating SMB Shares

Great video resource by KeepItTechie: [https://www.youtube.com/watch?v=2gW4rWhurUs](https://www.youtube.com/watch?v=2gW4rWhurUs)
[source](https://gist.github.com/pjobson/3811b73740a3a09597511c18be845a6c)

Create your share directory and set permissions 
```
sudo mkdir /data
sudo chmod -R 0777 /data
sudo chown -R brandon:brandon /data
```
Install Samba
```
sudo apt install samba
```
Edit the samba config
```
sudo nano /etc/samba/smb.conf
```
sudo nano /etc/samba/shares.conf
Add this to the configuration
```
[data]
   path = /data
   browseable = yes
   read only = no
   guest ok = no
[docker]
   path = /docker
   browseable = yes
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
Allow samba on firewall
```
sudo ufw allow Samba
sudo ufw status
```

