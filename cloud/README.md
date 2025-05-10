Work in progress.

# Self-Host the Cloud!
Ditching services like Google Workspace, Dropbox, iCloud, Microsoft 365, Amazon Photos, and other cloud service provides is critical...

## Nextcloud
Work in progress.

### Use a Network Share for Data Directory
[source](https://github.com/nextcloud/all-in-one?tab=readme-ov-file#can-i-use-a-cifssmb-share-as-nextclouds-datadir)
```
sudo nano /etc/fstab
//10.0.0.100/nextcloud /nextcloud cifs rw,mfsymlinks,seal,username=user,password=password,uid=33,gid=0,file_mode=0770,dir_mode=0770 0 0
```
### Notes
Nextcloud steps to add files manually to data directory.
#### Fix permissions 
```
chown -r www-data:www-data 
```
#### Scan for new files
```
sudo docker exec --user www-data -it nextcloud-aio-nextcloud php occ files:scan --all
```

## Immich
Work in progress.
