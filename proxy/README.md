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

