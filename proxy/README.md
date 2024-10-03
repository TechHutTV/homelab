# Work in progress

# Proxy with Interal and External Access

Goal: Use a proxy and ddns to manage two sperate domains. One for local only access to give all local ports a subdoamin and another to give external access via a domain for select services.

## Create LXC with Docker
wip
## Installing NGINX Reverse Proxy Manager
wip
## Setup a top-level domain for local use
General Steps
1. Assign a local IP scheme in the domain registration webaite. (ie. 10.0.0.60, container IP with proxy)
2. Add the domain in Nginx Proxy using the steps from a registar
3. Generate Let's Encrypt Certificate useing a DNS challenge and with API of your registar
4. Assign sub-domains to carious services with SSL certificates.
## Setup Twingate for remote connections
Goal: have local top-level domain working when connected remote with Twingate
## Setup DDNS for public access domain
Goal: use cloudflair and a dynamic DNS script for public IP updating

2. Open the ports on your router for the 80 and 443 ports we setup in NGINX Proxy Manager. In my docker compose file I used 9080 and 9443 so these are the ports I would open with the local IP of the machine that NGINX Proxy Manager is installed on.
2. Within Cloudflair optain your essentials tokens. My Profile > API Tokens > Create Token > Edit Zone DNS > Include All Zones > Create Token > Save your Token. Next, go to the domain you'll be using and find the Zone ID under API in the right sidebar, save that as well. 
3. Within Cloudflair use an A record create the sub-domains (or a wildcard) you wish to point to speficic services within your home network. For the IPv4 address we will have our DDNS container handle that. I recommened adding a random IP now (ie. 10.10.10.10) so in the next step we can varify that it will update aoutmatically to our public IP. Be sure to keep the 'Proxy status' option enabled.
4. If needing to use DDNS edit your Docker Compose file add your API, Zone, and domain names including sub-domains or a wildcard (*.exmaple.com) you want to setup for external access. When the container runs ensure there are no errors and the public IP in Cloudflair is updated to your actual IP.
5. 
