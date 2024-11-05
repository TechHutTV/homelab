# Nextcloud AIO
Nextcloud is my main platform for file sharing and synchronization for my video production projects and other business related documents. In addition to the files service we heavily utilize deck for project managment and collectives to manage our internal business documentation.

## Set up Nextcloud
Nextcloud AIO (all-in-one) is a docker container that manages and pull all other containers needed to run Nextcloud. The setup is fairly straight forward, but there are some important steps to get everything working correctly. The compose.yaml file within this directory contains all the varibles and everything you need to get started. Nextcloud provides comments within this file to help configure it to your needs. 

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

