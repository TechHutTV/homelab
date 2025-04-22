Work in progress. Dedicated Plex Stuff

## Permissions
```
sudo systemctl stop plexmediaserver
```
Stop the Plex service. `sudo systemctl plexmediaserver stop` Edit the Plex service configuration to reflext your user. `sudo systemctl edit plexmediaserver` Now add the user and group to your main user. You can run the `id` command if you're unsure.
```
[Service]
User=user
Group=group
```
```
sudo systemctl daemon-reload
```
```
sudo nano /etc/default/plexmediaserver 
```
```
PLEX_MEDIA_SERVER_USER
```
```
sudo chown -R user:group /var/lib/plexmediaserver
```
```
sudo systemctl start plexmediaserver
```
Source: [dausruddin.com](https://dausruddin.com/how-to-change-plex-user-running-under-in-ubuntu/)
