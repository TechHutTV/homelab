# Self Hosting NetBird with NPM and PocketID

work in progress

see [this guide](https://docs.netbird.io/selfhosted/selfhosted-quickstart) for now.

---

NetBird is an Open Source Zero Trust Networking platform that allows you to create secure private networks to connect directly to your home servers using the WireGuard protocol. Recently NetBird has improved the ease of self-hosting making it do a single script handles a majority of the configuration effort. This includes automatically setting up Caddy as a revserve proxy and allowing you to use local users. For my setup I like to use my own proxy and a IdP for added security. While I could self-host this on my own hardware at home I perfer to use a VPS in the cloud. That way I don't need to open any ports at home and if my home internet goes down (happens quite a bit) I don't lose access to other resources that are not hosted in my home servers.

## Prerequisites 
Most VPS services will work great, I'm currently using [Hostinger with their KVM 2](https://www.hostinger.com/cart?product=vps%3Avps_kvm_2&period=12&referral_type=cart_link&REFERRALCODE=TQRTECHHU4RG&referral_id=019bdcad-8f2c-71c7-b117-b48c2f17b265) _referral link*_ as they have been a sponsor in the past and it's a good value. This needs to be a Linux VM with at least 1CPU and 2GB of memory that is publicly accessible on TCP ports 80 and 443, and UDP port 3478.

Next you'll need a public domain prointing to the VPS IP address you'll be installing NetBird on. For this instance I will be running 3 different services. I use Cloudflare to manage my domains. To learn more checkout [our guide](https://github.com/TechHutTV/homelab/tree/main/proxy) on setting up a proxy for external access and local top-level domains.
| A Record    | IP Address |
| -------- | ------- |
| netbird  | your.public.ip.address    |
| proxy | your.public.ip.address      |
| auth    | your.public.ip.address     |

On the VPS create a local user with sudo privlages `adduser brandon` then `adduser brandon sudo` replacing _brandon_ with you name and switch to that user with `su brandon`. Additionally, you will need to make sure you have a few packages and docker setup. Update your system then install `curl` and `jq`. Then use the commands below to setup docker on your system.

```bash
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
sudo usermod -aG docker $USER
newgrp docker
```

## NetBird Setup

First in our home directory we can make a new folder for the NetBird services:

```bash
cd ~
mkdir netbird
cd netbird
```


Download and run the installation script:

```bash
curl -fsSL https://github.com/netbirdio/netbird/releases/latest/download/getting-started.sh | bash
```



The script will prompt you to enter your domain name for NetBird. You'll type something like `https://netbird.example.com` then select a reverse proxy option:

```
Which reverse proxy will you use?
  [0] Built-in Caddy (recommended - automatic TLS)
  [1] Traefik (labels added to containers)
  [2] Nginx (generates config template)
  [3] Nginx Proxy Manager (generates config + instructions)
  [4] External Caddy (generates Caddyfile snippet)
  [5] Other/Manual (displays setup documentation)

Enter choice [0-5] (default: 0):
```

For this guide, select option `[3]` to setup Nginx Proxy Manager. If you're using another service select it or select `[0]` Built-in Caddy if you want the script to handle all of that for you including automatice TSL certifications. Once the option is selected it will set everything up and spin up our containers. Since we have more to do we can spin down the stack for now.

```bash
docker compose down
```

## Custom Proxy and IdP

I like to use my own proxy and identiy provider. I will spin these up in a seperate stack to keep the services indepentant from NetBird. Back up to your home directory and make a new directory. 


