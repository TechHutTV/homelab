# Work in progress

# Setting up NGINX Proxy Manager for local and external domains

This is done on Proxmox with an LXC running Ubuntu 22.04 and Docker. However, these steps will work with any Docker installation. If you want details on installing Docker and a breif overview of all the basics you need to know to get started checkout our [7 Docker Basics for Beginners](https://techhut.tv/7-docker-basics-for-beginner).

## Installing NGINX Proxy Manager
This is done with the [Docker Compose file](https://github.com/TechHutTV/homelab/blob/main/proxy/compose.yaml) within this reposity. Do note I made some customizations for how I specifically like to set it up. I've changed some of the external ports to access 80, 443, and the GUI for NGINX Proxy Manager as well as placing the storage within a folder in my home directory. Please change these as needed or use the [offical compose file](https://github.com/NginxProxyManager/nginx-proxy-manager) from NGINX Proxy Manager. Additionally, I've added the container 'cloudflare-dynamic-dns' as my IP address changes randomly. If you don't have a dynamic IP address or don't have intention on exposing a service to the internet you can remove this container from the compose file.

Below is a basic compose template from NGINX if you don't want to use mine.

```
services:
  app:
    image: 'docker.io/jc21/nginx-proxy-manager:latest'
    restart: unless-stopped
    ports:
      - '80:80'
      - '81:81'
      - '443:443'
    volumes:
      - ./data:/data
      - ./letsencrypt:/etc/letsencrypt
```

## Setup a top-level domain for local use
General Steps
1. Assign a local IP scheme in the domain registration webaite. (ie. 10.0.0.60, container IP with proxy)
2. Add the domain in Nginx Proxy using the steps from a registar
3. Generate Let's Encrypt Certificate useing a DNS challenge and with API of your registar
4. Assign sub-domains to carious services with SSL certificates.
5. 
## Setup Twingate for remote connections
Goal: have local top-level domain working when connected remote with Twingate

## Setup DDNS for public access domain
1. Sign up for a Cloudflare account and use it to manage your domain using [this guide](https://developers.cloudflare.com/fundamentals/setup/manage-domains/add-site/).
2. Open the ports on your router for the 80 and 443 ports we setup in NGINX Proxy Manager. In my docker compose file I used 9080 and 9443 so these are the ports I would open with the local IP of the machine that NGINX Proxy Manager is installed on.
2. Within Cloudflare optain your essentials tokens. My Profile > API Tokens > Create Token > Edit Zone DNS > Include All Zones > Create Token > Save your Token. Next, go to the domain you'll be using and find the Zone ID under API in the right sidebar, save that as well. 
3. Within Cloudflare use an A record create the sub-domains (or a wildcard) you wish to point to speficic services within your home network. For the IPv4 address we will have our DDNS container handle that. I recommened adding a random IP now (ie. 10.10.10.10) so in the next step we can varify that it will update automatically to our public IP. Be sure to keep the 'Proxy status' option enabled.
4. If needing to use DDNS edit your Docker Compose file add your API, Zone, and domain names including sub-domains or a wildcard (*.exmaple.com) you want to setup for external access. When the container runs ensure there are no errors and the public IP in Cloudflare is updated to your actual IP.
Below is the compose tempplate for the cloudflare-dynamic-dns container. You can use it as I have it within my compose file or set it up seperatly.
```
services:
  ddns:
    image: mxmlndml/cloudflare-dynamic-dns:latest
    container_name: cloudflare-dynamic-dns
    restart: unless-stopped
    environment:
      - "API_KEY=${API_KEY}"
      - "ZONE_ID=${ZONE_ID}"
      - "DOMAIN_NAMES=example.com,dyndns.example.com"
      - "RECORD_TYPES=A"
      - "INTERVAL=5"
```
6. Now head over to NGINX Proxy Manager and create your SSL certificates. You navigate to SSL Certificates > Add SSL Certifcate. Type in your domain name, in my instance I used a wildcard (*.example.com) and then enable 'Use a DNS Challenge'. Select Cloudflare and paste in the API we saved from eariler.
7. Now in NGINX Proxy Manager navigate to Hosts > Add Proxy Host. Add the domain name for the service (ie. nextcloud.example.com) and select http (this may vary on if the service is running on https locally) then add the local IP and port for the service you want forwarded to the domain. Depending on the service you may need to enable 'Websockets Support', but I always select 'Block Common Exploits'. Nagivate the the SSL tab and select your SSL Certificate and enable 'Force SSL'
