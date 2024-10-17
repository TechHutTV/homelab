# Nextcloud AIO
wip
## Set up Nextcloud
wip
## NGINX Proxy Manager
Please see our proxy folder in this repo to get NGINX Proxy Manager set up. When adding the proxy make sure you use port ```11000``` as that is the apache port that Nextcloud uses. You can see the port layout and notes in the compose.yaml file.
```
...
    ports:
      # - 80:80 # Can be removed when running behind a web server or reverse proxy.
      - 8080:8080
      # - 8443:8443 # Can be removed when running behind a web server or reverse proxy.
    environment:
...
      APACHE_PORT: 11000 # Is needed when running behind a web server or reverse proxy.
      APACHE_IP_BINDING: 0.0.0.0 # Should be set when running behind a web server or reverse proxy.
...
      
```

Add the following to the advanced configuration. Some of these option are added for better performance behind the proxy.
```
fastcgi_request_buffering off;
client_body_buffer_size 512k;
proxy_read_timeout 86400s;
client_max_body_size 0;
```
When your proxy is added with SSL enabled head to the AIO dashboard and add your domain name there and ensure a successful connection.
## Additional Setups
wip
### Using Proton Mail (untested)
[Official Guide](https://proton.me/support/bridge-cli-guide)

On the same machine you're running Nextcloud on SSH into the server and run the following command.
```
docker run --name=protonmail-bridge --network=nextcloud-aio -p 1025:25/tcp -p 1143:143/tcp --rm -it -v protonmail:/root shenxn/protonmail-bridge init
```
Now login to your Proton account with the ```login``` command to begin syncing your data. Once that is done, you can enter info to verify that the login credentials are correct. Copy the password displayed, you will need it for the configuration on Nextcloud.

Then exit the container with CTRL+C. Run the following command to keep this container running.
```
docker run -d --name=protonmail-bridge -v protonmail:/root -p 127.0.0.1:1025:25/tcp -p 127.0.0.1:1143:143/tcp --restart=unless-stopped shenxn/protonmail-bridge
```
Configure Nextcloud
[source](https://github.com/steffenmalisi/nextcloud-all-in-one?tab=readme-ov-file#how-to-edit-nextclouds-configphp-file-with-a-texteditor)
```
sudo docker run -it --rm --volume nextcloud_aio_nextcloud:/var/www/html:rw alpine sh -c "apk add --no-cache nano && nano /var/www/html/config/config.php"
```
Add the following section to yout config.php in your Nextcloud config. This is because the certificate for the bridge is self-signed, and Nextcloud will reject it by default.
```
  'mail_smtpstreamoptions' => 
    array (
      'ssl' => 
      array (
        'allow_self_signed' => true,
        'verify_peer' => false,
        'verify_peer_name' => false,
      ),
    ),
```


Next, we will setup our account on Nextcloud. Enable the Mail app by clicking on your user profile and selecting Apps. Under featured apps enable the Mail application. After installation is complete click on Mail in the top nagivgation. 

```
IMAP Host: 127.0.0.1
IMAP Security: None
IMAP Port: 1143 (or whatever your ProtonMail Bridge config says)
SMTP Host: 127.0.0.1
SMTP Security: None
SMTP Port: 1025 (or whatever your ProtonMail Bridge config says)
```
[Source](https://help.nextcloud.com/t/nextcloud-mail-and-proton-mail/104205/2)
