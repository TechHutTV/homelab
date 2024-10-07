# External Proxy and Internal Top-Level Domains

This is going to be an overview of my setup for connecting to specific services through a proxy and DDNS combo, local top-level domain names, and how I connect to the internal home network remotely with Twingate.

This is done on Proxmox with an LXC running Ubuntu 22.04 and Docker. However, these steps will work with any Docker installation. If you want details on installing Docker and a brief overview of all the basics you need to know to get started checkout our [7 Docker Basics for Beginners](https://techhut.tv/7-docker-basics-for-beginner).

## Installing NGINX Proxy Manager
This is done with the [Docker Compose file](https://github.com/TechHutTV/homelab/blob/main/proxy/compose.yaml) within this repository. Do note, I made some customizations for how I specifically like to set it up. I've changed some of the external ports to access 80, 443, and the GUI for NGINX Proxy Manager as well as placing the storage within [volumes](https://docs.docker.com/engine/storage/volumes/). Please change these as needed or use the [official compose file](https://github.com/NginxProxyManager/nginx-proxy-manager) as seen below. Additionally, I've added the container [cloudflare-dynamic-dns](https://github.com/favonia/cloudflare-ddns) as my IP address changes randomly. If you don't have a dynamic IP address or don't have intention on exposing a service to the internet you can remove this container from the compose file.

### NGINX Proxy Manager Compose (customized)

```
services:
  proxy:
    image: 'jc21/nginx-proxy-manager:latest'
    container_name: nginx-proxy-manager
    restart: unless-stopped
    network_mode: host
    volumes:
      - proxy-data:/data
      - proxy-letsencrypt:/etc/letsencrypt
    healthcheck:
      test: ["CMD", "/usr/bin/check-health"]
      interval: 10s
      timeout: 3s
volumes:
  proxy-data:
  proxy-letsencrypt:
```

This is setup as a host network to allow localhost and local networking connections without needing to add ports for all the services to the container.

_Below is a basic compose template from NGINX if you don't want to use [mine](https://github.com/TechHutTV/homelab/blob/main/proxy/compose.yaml)._

#### Official Compose from NginxProxyManager/nginx-proxy-manager

Checkout the [quick setup](https://github.com/NginxProxyManager/nginx-proxy-manager?tab=readme-ov-file#quick-setup) section in their official repo.

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

Due note, as seen in my docker compose you'll need to either need to set the network mode to [host](https://stackoverflow.com/questions/42438381/docker-nginx-proxy-to-host#:~:text=Use%20network_mode%3A%20host%2C%20this%20will%20bind%20your%20nginx,every%20exposed%20port%20is%20binded%20to%20host%27s%20interface.) or [expose the specific ports](https://www.reddit.com/r/homelab/comments/1c38ize/nginx_proxy_manager_cant_route_to_different_port/#:~:text=Nginx%20Proxy%20Manager%20is%20in%20a%20docker%20container.) if running on bridge mode for servers that are running on your home network from a different machine. Also, be sure to check out their [Advanced Configuration](https://nginxproxymanager.com/advanced-config/) documents.

#### If using bridge mode see the example below
```
  proxy:
    ...
    network_mode: bridge
    ports:
      - 5080:80
      - 5443:443
      - 5000:81
      - 8096:8096 # add ports you want to expose that are not on your local server
    ...
```


## Setup DDNS for and Cloudflare for Public Access

### Cloudflare Setup
1. Sign up for a Cloudflare account and use it to manage your domain using [this guide](https://developers.cloudflare.com/fundamentals/setup/manage-domains/add-site/).
2. Within Cloudflare [obtain your API token](https://developers.cloudflare.com/fundamentals/api/get-started/create-token/). _My Profile > API Tokens > Create Token > Edit Zone DNS > Include All Zones > Create Token > Save your Token_. We will be using this token in the cloudflare-ddns container configuration and when we generate SSL certificates.

### Port Forwarding
This is different for every router so you may need to do additional research to do this on your specific hardware. I currently use the Omada stack for networking needs. Basically, it's like Ubiquiti but cheaper (you get what you pay for).

Open the ports on your router for the 80 and 443 ports we set up in NGINX Proxy Manager. In my docker compose file I'm using the host networking mode so I'd open the ports 80 and 443 with the local IP of the machine that NGINX Proxy Manager is installed on. In my setup I needed to set the source port and destination port. See my example below.

<details>
<summary>Source Port vs. Destination Port</summary>
<br>
<b>Source Port:</b>
This is the port on the device that is initiating the communication. For example, when your computer sends a request to a server, it uses a source port to identify itself.
<br><br>
<b>Destination Port:</b>
This is the port on the device that will receive the communication. For example, when you're connecting to a web server. The destination port is fixed for the service you're trying to reach and tells the receiving device what service or application should handle the incoming data.
</details>

![Omada Port Forwarding](https://github.com/TechHutTV/homelab/blob/main/proxy/images/odama-port-forwarding-443.jpeg)

If using bridge mode with custom ports, for example 5080 and 5443 as shown in the example. I'd set the destination port to 5443 and the source port to 443 for https.

### Dynamic DNS
1. Within Cloudflare use an A record to create the root domain and/or sub-domains you wish to point to specific services within your home network. For the IPv4 address we will have our DDNS container handle that. I recommend adding a random IP now (ie. 8.8.8.8) so in the next step we can verify that it will update automatically to our public IP. Be sure to keep the 'Proxy status' option enabled.
2. If you need to use DDNS, edit your Docker Compose file, add your API, and domain names including subdomains you want to set up for external access. When the container runs ensure there are no errors and the public IP in Cloudflare is updated to your actual IP.

Below is the compose template for the cloudflare-dynamic-dns container. You can use it as I have it within my compose file or set it up separately.

```
services:
  ddns:
    image: favonia/cloudflare-ddns:latest
    container_name: cloudflare-ddns
    # network_mode: host # This bypasses network isolation and makes IPv6 easier (optional; see below)
    restart: always
    user: "1000:1000" # Run the updater with specific user and group IDs (in that order).
    read_only: true # Make the container filesystem read-only (optional but recommended)
    cap_drop: [all] # Drop all Linux capabilities (optional but recommended)
    security_opt: [no-new-privileges:true] # Another protection to restrict superuser privileges (optional but recommended)
    environment:
      - CLOUDFLARE_API_TOKEN=KEY
      - DOMAINS=example.com,jellyfin.example.com
      - PROXIED=true
      - IP6_PROVIDER=none
```

### Generate SSL Certificates and Add Hosts
1. Now head over to NGINX Proxy Manager and create your SSL certificates. You navigate to _SSL Certificates > Add SSL Certifcate_. Type in your domain name and then enable 'Use a DNS Challenge'. Select Cloudflare and paste in the API we saved from earlier. 
2. Now in NGINX Proxy Manager navigate to _Hosts > Add Proxy Host_. Add the domain name for the service (ie. nextcloud.example.com) and select http (this may vary on if the service is running on https locally) then add the local IP and port for the service you want forwarded to the domain.
* Depending on the service you may need to enable _Websockets Support_, but I always select _Block Common Exploits_.
* Navigate the the SSL tab and select your SSL Certificate and enable _Force SSL_. **See known issues below.**
* Depending on the service you may need to make changes to the settings in the specific service, such as allowing proxies and add some advanced configuration, for example Jellyfin requires some additional configuration.
  * Jellyfin requires you to add the approved proxy ip for the local NGINX Proxy Manager Machine. [source](https://jellyfin.org/docs/general/networking/#:~:text=SERVER_IP_ADDRESS)
  * Jellyfin has additional configurations for the advanced tab in proxy host settings. [source](https://jellyfin.org/docs/general/networking/nginx/#nginx-proxy-manager)

#### Known Issues and Tips
* **Too Many Redirects:** Force SSL may not work with CloudFlare proxying. [issue](https://github.com/NginxProxyManager/nginx-proxy-manager/issues/852)
* **Disable Cloudflare Proxy on Streaming:** Jellyfin, Plex and other streaming services are not allowed to use Proxy on the free plan. Doing this technically [breaks their TOS](https://www.cloudflare.com/service-specific-terms-application-services/#content-delivery-network-terms) and may result in your account getting banned. Just to be safe I used a subdomain for my Jellyfin instance as a separate A-Record and disabled the Cloudflare Proxy.

![Disable Cloudflare Proxy for Media Streaming](https://github.com/TechHutTV/homelab/blob/main/proxy/images/disable-proxy-media-streaming.png)

---

# Local Top-Level Domains and Twingate

Within this section we will use our NGINX Proxy Manager setup and our domain registrar directly to create a proxy host scheme for local access only. This will also allow us to use letsencrypt to generate SSL certificates for our local network. This will eliminate that horrible _this site is not secure_ message on our services! Also, we will be setting up Twingate (a channel sponsor) to enable a zero trust network for remote access to those services we don't want to expose publically.

## Setup a Top-Level Domain for Local Use

### Local IP on Registar
Assign a local IP scheme in the domain registration website. The local IP you will use is the same as the machine running NGINX Proxy Manager. (ie. 10.0.0.60). You'll want to assign this to the A-Record for the main domain and create a CNAME Record as a wildcard (*) pointing to the main domain name. Due note, this may take some time, it took about 15 minutes for the record to update for me. If you're using Cloudflare make sure you disable their proxy service.

![Record for Local Top-Level Domain](https://github.com/TechHutTV/homelab/blob/main/proxy/images/local-ip-wildcard.png)

While you're on Cloudflare or the registar find your API key. You'll need this for generating SSL certificates in the DNS challenges option. Many providers are supported and you can see a [full list here](https://community.letsencrypt.org/t/dns-providers-who-easily-integrate-with-lets-encrypt-dns-validation/86438).

### Adding Proxy Hosts
This will mirror the steps above, with some slight differences. In NGINX Proxy Manager navigate to _Hosts > Add Proxy Host_. Add the domain name for the service (ie. example.com) and select http (this may vary depending on if the service is running on https locally) then add the local IP and port for the service you want forwarded to that domain. If you want to test everything check below.

#### Testing
There is a simple container we can use to test our domain with the local IP. In the terminal run the docker command below on the same machine that is running your Proxy Manager. This is also available as docker compose in the compose.yaml file in this repository.

```
docker run -p 8888:80/tcp "karthequian/helloworld:latest"
```

Add a subdomain (hello.example.com) in proxy hosts with the IP running this helloworld container and the port _8888_. Set it to http only with no SSL since we have not set that up yet.
1. Navigate to example.com:8888 to test if the A-Record and CNAME is working properly.
2. Navigate to hello.example.com to test if the reverse proxy is working.

### Generate Let's Encrypt Certificates
Navigate to _SSL Certificates > Add SSL Certifcate_. Type in your root domain name (example.com) click add then input the wildcare domain (*.example.com) and then enable 'Use a DNS Challenge'. Select your registar and paste in the API we saved from eariler. If you run into error make sure that your API key is correct, whitelist your public IP with you registar if needed, or try increasing the _Propagation Seconds_ to 120 seconds.

#### Testing
With the helloworld container still running, head over to _Proxy Hosts_ and edit the hello.example.com host. In the SSL tab add _*.example.com_ under the SSL Certificate and enable _Force SSL_. Navigate to hello.example.com to ensure that the connection is automatically redirected to https.

#### Known Issues and Tips
* **Namecheap API Whitelist:** Namecheap isn't really the best for this if you have a Dynamic IP. Whenever I want to update my certificates I need to whitelist my public IP so it can use their API. I will be switching to using Cloudflare for this going forward.

## Setup Twingate for remote connections
**Notice: Twingate is a channel sponsor, thus this is a bias disclosure.** Twingate uses cloud based software for managing networks, resources, and users. Some users prefer to self-host every aspect of this, if that's you look into something like [Netbird](https://github.com/netbirdio/netbird). For myself, Twingate has been awesome; easy to use and [others rate it well](https://www.reddit.com/r/twingate/comments/1awg76l/how_safe_is_twingate/).

After [creating an account on Twingate](https://bit.ly/feb24-twingate) and setting up [your first network](https://www.twingate.com/docs/quick-start) we need to set up a connector. Within my docker compose file I have the twingate-connector service ready to deploy with the entire stack. Here is what this service looks like.

```
services:
  twingate_connector:
    container_name: twingate_connector
    restart: always
    image: "twingate/connector:latest"
    environment:
      - TWINGATE_NETWORK=<TENANT NAME>.twingate.com
      - TWINGATE_ACCESS_TOKEN=<ACCESS TOKEN>
      - TWINGATE_REFRESH_TOKEN=<REFRESH TOKEN>
```
When you create your connector in the Twingate dashboard you'll generate some tokens. Enter them in the environmental variables and launch the stack. Verify a connection under networks in Twingate.

Next, create a new resource with the IP of your proxy manager and add the local root domain as an alias. Once created you should be able to have access to the local domain we created earlier including sub-domains. See the image before for an example.

![Adding an Alias in Twingate](https://github.com/TechHutTV/homelab/blob/main/proxy/images/twingate-alias.jpeg)

# Additional Resources

| Additional Security Steps | Twingate Guide |
| ------------- | ------------- |
| [![Twingate Guide](https://github.com/TechHutTV/homelab/blob/main/proxy/images/technotim-security-guide.jpg)](https://www.youtube.com/watch?v=Cs8yOmTJNYQ "DITCH your VPN! - How I Access my Home Server from ANYWHERE @TechHut")  | [![Additional Security Steps](https://github.com/TechHutTV/homelab/blob/main/proxy/images/twingate-techhut.jpg)](https://youtu.be/yaw2A3DG664 "Self-Hosting Security Guide for your HomeLab @TechnoTim")  |



