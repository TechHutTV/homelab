Work in progress.

# Self-Host the Cloud!
Ditching services like Google Workspace, Dropbox, iCloud, Microsoft 365, Amazon Photos, and other cloud service provides is critical...

## Nextcloud
Work in progress.

### NGINX Proxy Manager
Under details set the scheme to http, your local IP for the server, and the port 11000 then enable Block Common Exploits and Websocket Support under details. Under SSL enable Force SSL and HTTP/2 support. Under advanced add the following lines.
```
client_body_buffer_size 512k;
proxy_read_timeout 86400s;
client_max_body_size 0;
```

### Use a Network Share for Data Directory
[source](https://github.com/nextcloud/all-in-one?tab=readme-ov-file#can-i-use-a-cifssmb-share-as-nextclouds-datadir)
```
sudo nano /etc/fstab
//10.0.0.100/nextcloud /nextcloud cifs rw,mfsymlinks,seal,username=user,password=password,uid=33,gid=0,file_mode=0770,dir_mode=0770 0 0
```
### Add Files to Data Directory Manually
Nextcloud steps to add files manually to data directory. 
#### Fix permissions 
```
chown -R www-data:www-data ./directory
```
#### Scan for new files
```
sudo docker exec --user www-data -it nextcloud-aio-nextcloud php occ files:scan --all
```
### Find lost AIO Passphrase
```
docker exec nextcloud-aio-mastercontainer grep password /mnt/docker-aio-config/data/configuration.json
```

## Immich
Work in progress.
