# External Proxy and Internal Top-Level Domains

This is going to be an overview of my setup for connecting to specific services through a proxy and DDNS combo, local top-level domain names, and how I connect to the internal home network remotley with Twingate.

This is done on Proxmox with an LXC running Ubuntu 22.04 and Docker. However, these steps will work with any Docker installation. If you want details on installing Docker and a breif overview of all the basics you need to know to get started checkout our [7 Docker Basics for Beginners](https://techhut.tv/7-docker-basics-for-beginner).

## Installing NGINX Proxy Manager
This is done with the [Docker Compose file](https://github.com/TechHutTV/homelab/blob/main/proxy/compose.yaml) within this reposity. Do note I made some customizations for how I specifically like to set it up. I've changed some of the external ports to access 80, 443, and the GUI for NGINX Proxy Manager as well as placing the storage within [volumes](https://docs.docker.com/engine/storage/volumes/). Please change these as needed or use the [offical compose file](https://github.com/NginxProxyManager/nginx-proxy-manager) as seen below. Additionally, I've added the container [cloudflare-dynamic-dns](https://github.com/favonia/cloudflare-ddns) as my IP address changes randomly. If you don't have a dynamic IP address or don't have intention on exposing a service to the internet you can remove this container from the compose file.

_Below is a basic compose template from NGINX if you don't want to use [mine](https://github.com/TechHutTV/homelab/blob/main/proxy/compose.yaml)._

### Offical Compose from NginxProxyManager/nginx-proxy-manager

Checkout the [quick setup](https://github.com/NginxProxyManager/nginx-proxy-manager?tab=readme-ov-file#quick-setup) section in their offical repo.

You can add an image or a code block, too.

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
Due note, as seen in my docker compose you'll need to either net the network mode to [host](https://stackoverflow.com/questions/42438381/docker-nginx-proxy-to-host#:~:text=Use%20network_mode%3A%20host%2C%20this%20will%20bind%20your%20nginx,every%20exposed%20port%20is%20binded%20to%20host%27s%20interface.) or [expose the specific ports](https://www.reddit.com/r/homelab/comments/1c38ize/nginx_proxy_manager_cant_route_to_different_port/#:~:text=Nginx%20Proxy%20Manager%20is%20in%20a%20docker%20container.) for servers that are running on your home network from a different machine. Also, be sure to checkout their[ Advanced Configuration](https://nginxproxymanager.com/advanced-config/) documents.


## Setup DDNS for and Cloudflare for Public Access

### Cloudflare Setup
1. Sign up for a Cloudflare account and use it to manage your domain using [this guide](https://developers.cloudflare.com/fundamentals/setup/manage-domains/add-site/).
2. Within Cloudflare [optain your API token](https://developers.cloudflare.com/fundamentals/api/get-started/create-token/). _My Profile > API Tokens > Create Token > Edit Zone DNS > Include All Zones > Create Token > Save your Token_. We will be using this token in the cloudflare-ddns container configuration and when we generate SSL certifations.

### Portforwarding
This is different for every router so you may need to do additional research to do this on your specific hardware. I currently use the Omada stack for networking needs. Basically, it's like Ubiquiti but cheaper (you get what you pay for).

Open the ports on your router for the 80 and 443 ports we setup in NGINX Proxy Manager. In my docker compose file I used 9080 and 9443 so these are the ports I would open with the local IP of the machine that NGINX Proxy Manager is installed on. In my setup I needed to set the source port and destination port. See my example below.
  
#### Source Port:
This is the port on the device that is initiating the communication. For example, when your computer sends a request to a server, it uses a source port to identify itself.

#### Destination Port:
This is the port on the device that will receive the communication. For example, when you're connecting to a web server. The destination port is fixed for the service you're trying to reach and tells the receiving device what service or application should handle the incoming data.

![Omada Port Forwarding](https://github.com/TechHutTV/homelab/blob/main/proxy/odama-port-forwarding-443.jpeg)

### Dynamic DNS
1. Within Cloudflare use an A record create the root domain and/or sub-domains you wish to point to speficic services within your home network. For the IPv4 address we will have our DDNS container handle that. I recommened adding a random IP now (ie. 8.8.8.8) so in the next step we can varify that it will update automatically to our public IP. Be sure to keep the 'Proxy status' option enabled.
2. If needing to use DDNS edit your Docker Compose file add your API, Zone, and domain names including sub-domains or a wildcard (*.exmaple.com) you want to setup for external access. When the container runs ensure there are no errors and the public IP in Cloudflare is updated to your actual IP.
Below is the compose tempplate for the cloudflare-dynamic-dns container. You can use it as I have it within my compose file or set it up seperatly.
```
services:
  ddns:
    image: favonia/cloudflare-ddns:latest
    container_name: cloudflare-ddns
    # network_mode: host
    # This bypasses network isolation and makes IPv6 easier
    restart: always
    user: "1000:1000"
    read_only: true
    # Make the container filesystem read-only (optional but recommended)
    cap_drop: [all]
    # Drop all Linux capabilities (optional but recommended)
    security_opt: [no-new-privileges:true]
    # Another protection to restrict superuser privileges (optional but recommended)
    environment:
      - CLOUDFLARE_API_TOKEN=token
      - DOMAINS=example.com,jellyfin.example.com
      - PROXIED=true
        # Tell Cloudflare to cache webpages and hide your IP (optional)
      - IP6_PROVIDER=none
```

### Generate SSL Certificates and Add Hosts
1. Now head over to NGINX Proxy Manager and create your SSL certificates. You navigate to _SSL Certificates > Add SSL Certifcate_. Type in your domain name and then enable 'Use a DNS Challenge'. Select Cloudflare and paste in the API we saved from eariler. 
2. Now in NGINX Proxy Manager navigate to _Hosts > Add Proxy Host_. Add the domain name for the service (ie. nextcloud.example.com) and select http (this may vary on if the service is running on https locally) then add the local IP and port for the service you want forwarded to the domain.
* Depending on the service you may need to enable _Websockets Support_, but I always select _Block Common Exploits_.
* Nagivate the the SSL tab and select your SSL Certificate and enable _Force SSL_. **See known issues below.**
* Depending on the service you may need to make changes to the settings in the specifc service, such as allowing proxys and add some advanced configuration, for example Jellyfin requires some additional configuration.
  * Jellyfin requires you to add the approved proxy ip for the local NGINX Proxy Manager Machine. [source](https://jellyfin.org/docs/general/networking/#:~:text=SERVER_IP_ADDRESS)
  * Jellyfin has additional configurations for the advanced tab in proxy host settings. [source](https://jellyfin.org/docs/general/networking/nginx/#nginx-proxy-manager)

#### Known Issues and Tips
* **Too Many Redirects:** Force SSL may not work with CloudFlare proxying. [issue](https://github.com/NginxProxyManager/nginx-proxy-manager/issues/852)
* **Disable Cloudflare Proxy on Streaming:** Jellyfin, Plex and other streaming services are not allowed to use Proxy on the free plan. Doing this technically [breaks their TOS](https://www.cloudflare.com/service-specific-terms-application-services/#content-delivery-network-terms) and may result in your account getting banned. Just to be safe I used a subdomain for my jellyfin instance as a seperate A-Record and disabled the Cloudflair Proxy.
![Disable Cloudflare Proxy for Media Streaming](https://github.com/TechHutTV/homelab/blob/main/proxy/disable-proxy-media-streaming.png)


---
# Work in Progress

## Setup a top-level domain for local use
General Steps
1. Assign a local IP scheme in the domain registration webaite. (ie. 10.0.0.60, container IP with proxy)
2. Add the domain in Nginx Proxy using the steps from a registar
3. Generate Let's Encrypt Certificate useing a DNS challenge and with API of your registar
4. Assign sub-domains to carious services with SSL certificates.
5. 
## Setup Twingate for remote connections
Goal: have local top-level domain working when connected remote with Twingate


