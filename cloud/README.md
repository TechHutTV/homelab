Work in progress.

# Self-Host the Cloud!
Ditching services like Google Workspace, Dropbox, iCloud, Microsoft 365, Amazon Photos, and other cloud service provides is critical...

## Nextcloud
Work in progress.

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
