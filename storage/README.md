# Storage and Backup
In this repo I will layout my storage and backup solutions for all of the services and platforms running on my homelab. Currently, I manage everything with Proxmox and Proxmox Backup Server. While solutions like Unraid and TrueNAS are awesome, I have found over the years that Proxmox is actually an amazing solution for managing storage, network shares, and backups.

> [!NOTE]
> I've recently switched to using Unraid on a separate machine for my shares and primary file storage; however, this guide still works great for an all-Proxmox solution. I will make changes to this page in the next few weeks to reflect additional storage options and the best ways to mount NFS shares in Proxmox.

## Video Guides
This readme is a companion to my official walkthrough guide!

[![](https://raw.githubusercontent.com/TechHutTV/homelab/refs/heads/main/storage/part1_thumbnail.webp)](https://youtu.be/qmSizZUbCOA)

## Navigation
* [Apps](https://github.com/TechHutTV/homelab/tree/main/apps) - List of all the apps and services.
* [Home Assistant](https://github.com/TechHutTV/homelab/tree/main/homeassistant) - Smart home services and automation.
* [Media Server](https://github.com/TechHutTV/homelab/tree/main/media) - Plex, Jellyfin, *arr stack, and more.
* [Server Monitoring](https://github.com/TechHutTV/homelab/tree/main/monitoring) - Graphs and Visualizations for Unraid, Proxmox, and more.
* [Surveillance System](https://github.com/TechHutTV/homelab/tree/main/surveillance) - Frigate NVR Solution with Coral TPU.
* **Storage** - Current Storage and Backup Solution.
* [Proxy Management](https://github.com/TechHutTV/homelab/tree/main/proxy) - NGINX Proxy Manager, DDNS with Cloudflare, Local Domains, and more.

## Proxmox as a NAS
My current setup involves a single server with x3 NVME drives and a bunch of hard drives in a ZFS configuration. These are combined into separate ZFS pools for the HDDs (vault) and the SSDs (flash). Vault is used as a large data storage pool and Flash is used for containers and virtual machine disks. No matter your configuration you can follow this guide. However, I would recommend at least one NVME SSD for your boot drive, and at least 512gb if you don't have any other NVME SSDs and at least x2 HDDs for file storage.

### Post Install Steps (optional)

#### Disable Enterprise Repositories
1. Navigate to _Node > Repositories_ Disable the enterprise repositories.
2. Now click Add and enable the no subscription repository. Finally, go _Updates > Refresh_.
3. Upgrade your system by clicking _Upgrade_ above the repository setting page.

![](https://raw.githubusercontent.com/TechHutTV/homelab/refs/heads/main/storage/1_proxmox-repos.jpeg)

#### Delete local-lvm and Resize local (fresh install)
> [!WARNING]
> This assumes a fresh installation without advanced storage settings during the installation. See this [issue](https://github.com/TechHutTV/homelab/issues/19).

My boot drive is small and I run all my containers and virtual machine disks on a separate storage pool. So the LVM partition is not necessary for me and goes unused. If you're running everything off the same boot drive for fast storage skips this. Also you should check out this [video](https://www.youtube.com/watch?v=czQuRgoBrmM) to learn more about LVM before doing anything.

1. Delete local-lvm manually from web interface under _Datacenter > Storage_.
2. Run the following commands within _Node > Shell_.
   ```bash
   lvremove /dev/pve/data
   lvresize -l +100%FREE /dev/pve/root
   resize2fs /dev/mapper/pve-root
   ```
3. Check to ensure your local storage partition is using all available space. Reassign storage for containers and VM if needed.

#### Ensure IOMMU is enabled
Enable IOMMU on in grub configuration within _Node > Shell_.
```bash
nano /etc/default/grub
```
You will see the line with `GRUB_CMDLINE_LINUX_DEFAULT="quiet"`, all you need to do is add `intel_iommu=on` or `amd_iommu=on` depending on your system.
```
# Should look like this
GRUB_CMDLINE_LINUX_DEFAULT="quiet intel_iommu=on"
```
![](https://raw.githubusercontent.com/TechHutTV/homelab/refs/heads/main/storage/2_proxmox-iommu.jpeg)

Next run the following commands and reboot your system.
```bash
update-grub
```
Now check to make sure everything is enabled.
```bash
dmesg | grep -e DMAR -e IOMMU
dmesg | grep 'remapping'
```
Learn about enabling PCI Passthrough [here](https://pve.proxmox.com/wiki/PCI_Passthrough)

### Create ZFS Pools
First, we are going to setup two ZFS Pools. A _tank_ pool which is used for larger stored data sets such as media, images and archives. We also will make a _flash_ pool which is used for virtual machine and container root file systems. This is what I name them for my setup. You can name these however you'd like.

First, check out your disks and make sure that they're all there. Find this under _Node > Disks_. Make sure you wipe all the disks you plan on using and do note this will wipe any data on the disks, so make sure there is no important data on them and back up if needed.

![](https://raw.githubusercontent.com/TechHutTV/homelab/refs/heads/main/storage/3_proxmox-wipe-disk.jpeg)

Now, on the Proxmox sidebar for your datacenter, go to _Disks > ZFS > Create: ZFS_. This will pop up the screen to create a ZFS pool.

From this screen, it should show all of your drives, so select the ones you want in your pool, and select your RAID level (in my case RAIDZ for my vault pool and mirror for my flash pool) and compression, (in my case I keep it at on). Make sure you check the box that says __Add to Storage__. This will make the pools immediately available and will prevent using .raw files as opposed to my previous setup when I added directories.

![](https://raw.githubusercontent.com/TechHutTV/homelab/refs/heads/main/storage/4_proxmox-mirror-nvme.jpeg)

### Creating Containers using ZFS Pools
Now time to put these new storage pools in use. For this, we are going to create our first LXC. In this example the LXC is going to be in charge of managing our media server. First we need a operating system image. Click on your local storage in the sidebar and click on CT Templates then the Templates button. From there search for Ubuntu and download the latest ubuntu template (22.04 or 24.04).

Now in the top right click on Create CT. The "Create: LXC Container" prompt should show up. On the general tab I set my CT ID to 100 (later I will match this to a local IP for organization) and I set the hostname to "servarr", you can name it anything like media, jellyfin, or whatever. Set your password, keep the container and unprivileged and click Next. Select your downloaded Ubuntu template and click next. Under disk you can select your storage location. If you created the flash pool like we did earlier select it, otherwise local is fine. For storage I picked 64gb as my media server is quite large. Click next as we will add the data and docker directory later. Give it as many CPU cores and ram as you need, for my setup I gave it 6 cores and 8gb of memory.

Under network we will leave most everything, but I like to give it a static IP here. If you want to manage this with your router select DHCP. Under IPv4 I set the IPv4/CIDR to `10.0.0.100/24` and the gateway to `10.0.0.1` your local IP may be different. Keep DNS as is and confirm the installation.

### Adding Mount Points
Now that our container is created I want to add some storage and mount the data and docker directories on my system. Click on your newly created LXC and then click on Resources. From there click the Add button and select mount point. The first one I'll add is going to be for the bulk file storage or I will change the option under storage to tank. For path I will set this to /data and uncheck backup. We will set up backups later. I want to dedicate a ton of room to this so I set it to 26078 GiB (28 TB). Set this to what works best for how much media you'd like to store there. I keep everything else as is and click create. For the docker mount I repeated all these steps, but set the storage to flash, mount point to /docker, and gave it about 128gb of space.

### Creating SMB Shares
In our new LXC we first need to run some general updates and user creation.

> [!IMPORTANT]
> Throughout this section, replace `youruser` with your own username.

1. Update your system
   ```bash
   apt update && apt upgrade -y
   ```

2. Create your user
   ```bash
   adduser youruser
   adduser youruser sudo
   ```
   Great [video resource by KeepItTechie](https://www.youtube.com/watch?v=2gW4rWhurUs), [source](https://gist.github.com/pjobson/3811b73740a3a09597511c18be845a6c)

3. Switch to your new user
   ```bash
   su - youruser
   ```

4. Set permissions of mount points created earlier.
   ```bash
   sudo chown -R youruser:youruser /data
   sudo chown -R youruser:youruser /docker
   ```

5. Install Samba
   ```bash
   sudo apt install samba
   ```

6. Create a backup of the default configuration
   ```bash
   cd /etc/samba
   sudo mv smb.conf smb.conf.old
   ```

7. Edit the samba config
   ```bash
   sudo nano smb.conf
   ```

   > [!IMPORTANT]
   > Replace `youruser` with your actual username. Change `hosts allow` to match your network subnet if needed. The default config below allows all common private networks (192.168.x.x, 10.x.x.x, 172.16-31.x.x). See [issue 51](https://github.com/TechHutTV/homelab/issues/51) for more info on subnets.

   ```
   [global]
      server string = Servarr
      workgroup = WORKGROUP
      security = user
      map to guest = Bad User
      name resolve order = bcast host
      hosts allow = 192.168.0.0/16 10.0.0.0/8 172.16.0.0/12
      hosts deny = 0.0.0.0/0

   [data]
      path = /data
      force user = youruser
      force group = youruser
      create mask = 0774
      force create mode = 0774
      directory mask = 0775
      force directory mode = 0775
      browseable = yes
      writable = yes
      read only = no
      guest ok = no

   [docker]
      path = /docker
      force user = youruser
      force group = youruser
      create mask = 0774
      force create mode = 0774
      directory mask = 0775
      force directory mode = 0775
      browseable = yes
      writable = yes
      read only = no
      guest ok = no
   ```

8. Add your samba user
   ```bash
   sudo smbpasswd -a youruser
   ```

9. Set services to auto start on reboot
   ```bash
   sudo systemctl enable smbd
   sudo systemctl enable nmbd
   sudo systemctl restart smbd
   sudo systemctl restart nmbd
   ```

10. Install wsdd for Windows network discovery
    ```bash
    sudo apt install wsdd
    sudo systemctl enable wsdd
    sudo systemctl start wsdd
    ```

11. Allow services on firewall if you run into any issues.
    ```bash
    sudo ufw allow OpenSSH
    sudo ufw allow Samba
    # following 3 are needed for wsdd
    sudo ufw allow 3702/udp
    sudo ufw allow 5357/tcp
    sudo ufw allow 5358/tcp
    # Check ufw status
    sudo ufw status
    ```
    Optionally, enable the firewall.
    ```bash
    sudo ufw enable
    ```

### macOS Compatibility (Optional)
If you're connecting from macOS or iOS devices, add the `vfs_fruit` module for better compatibility. This handles Apple-specific features like resource forks, Finder metadata, and fixes common issues with `.DS_Store` files and slow browsing.

Add these lines to the `[global]` section of your `smb.conf`:

```
[global]
   # ... existing config above ...
   
   # macOS/iOS compatibility
   vfs objects = catia fruit streams_xattr
   fruit:nfs_aces = no
   fruit:zero_file_id = yes
   fruit:metadata = stream
   fruit:model = MacSamba
```

**What each option does:**
- `vfs objects = catia fruit streams_xattr` - Enables Apple SMB extensions and character translation
- `fruit:nfs_aces = no` - Disables NFS ACLs which can cause permission issues
- `fruit:zero_file_id = yes` - Fixes issues with Time Machine and some apps
- `fruit:metadata = stream` - Stores Apple metadata in streams instead of separate files
- `fruit:model = MacSamba` - Sets the icon shown in Finder

After editing, restart Samba:
```bash
sudo systemctl restart smbd
```

For more options, see the [vfs_fruit documentation](https://www.samba.org/samba/docs/current/man-html/vfs_fruit.8.html).

## Connecting to Shares

### From Windows
Since `guest ok = no` is set in the config, Windows requires explicit credentials to connect.

**Map Network Drive (Recommended)**
1. Open File Explorer
2. Right-click "This PC" → "Map network drive"
3. Enter `\\SERVER-IP\data` (e.g., `\\192.168.1.100\data`)
4. Check ✅ "Connect using different credentials"
5. Enter your Samba username and password (created with `smbpasswd`)

**Quick Access**

Press `Win + R`, type `\\SERVER-IP\data`, and press Enter. Enter credentials when prompted.

### From macOS
1. Open Finder
2. Press `Cmd + K` or go to _Go > Connect to Server_
3. Enter `smb://SERVER-IP/data` (e.g., `smb://192.168.1.100/data`)
4. Click Connect
5. Select "Registered User" and enter your Samba username and password

To auto-mount on login, add the share to _System Preferences > Users & Groups > Login Items_.

### From Linux
You can mount SMB shares temporarily or permanently on Linux clients.

**Temporary Mount**
```bash
# Install cifs-utils if not already installed
sudo apt install cifs-utils

# Create mount point
sudo mkdir -p /mnt/data

# Mount the share (you'll be prompted for password)
sudo mount -t cifs //SERVER-IP/data /mnt/data -o username=youruser
```

**Permanent Mount (fstab)**
1. Create a credentials file:
   ```bash
   sudo nano /etc/samba/credentials
   ```
   Add:
   ```
   username=youruser
   password=yourpassword
   ```
   Secure it:
   ```bash
   sudo chmod 600 /etc/samba/credentials
   ```

2. Add to `/etc/fstab`:
   ```
   //SERVER-IP/data /mnt/data cifs credentials=/etc/samba/credentials,uid=1000,gid=1000 0 0
   ```

3. Mount it:
   ```bash
   sudo mount -a
   ```

### From iOS
1. Open the Files app
2. Tap the three dots (...) in the top right → "Connect to Server"
3. Enter `smb://SERVER-IP` (e.g., `smb://192.168.1.100`)
4. Select "Registered User" and enter your Samba credentials
5. Your shares will appear under "Shared" in the Files app

> [!TIP]
> For best iOS compatibility, make sure you've added the [macOS compatibility options](#macos-compatibility-optional) to your Samba config.

## Troubleshooting Samba

### Server not showing in Windows Network
Windows 10/11 use WS-Discovery instead of NetBIOS. Make sure `wsdd` is installed and running:
```bash
sudo systemctl status wsdd
```

### "The specified network name is no longer available"
This usually means the connection is being blocked. Check these in order:

1. **Verify the server is reachable:**
   ```powershell
   # Run from Windows PowerShell
   Test-NetConnection -ComputerName SERVER-IP -Port 445
   ```

2. **Check `hosts allow` matches your network:**
   ```bash
   grep "hosts allow" /etc/samba/smb.conf
   ```
   If your PC is on `192.168.x.x` but `hosts allow` only has `10.0.0.0/24`, connections will be refused.

3. **Check firewall:**
   ```bash
   sudo ufw status
   ```
   Make sure Samba is allowed. If not: `sudo ufw allow Samba`

4. **Verify Samba is running:**
   ```bash
   sudo systemctl status smbd
   ```

5. **Check Samba config for errors:**
   ```bash
   testparm
   ```

### "Access Denied" or "Bad Password"
Verify the Samba user exists and password is set:
```bash
sudo pdbedit -L  # List Samba users
sudo smbpasswd -a youruser  # Add/reset password
```

### Can connect via IP but not hostname
Install and enable wsdd, then verify it's running:
```bash
sudo apt install wsdd
sudo systemctl enable --now wsdd
```

# Backups
Work in Progress
