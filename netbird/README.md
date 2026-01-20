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

## Nginx Proxy Manager (NPM)

I like to use Nginx Proxy Manager as its has a wonderful GUI and it's easy to use. I will spin this up in a seperate stack to keep the services indepentant from NetBird. Back up to your home directory and make a new directory. 

```bash
cd ~
mkdir proxy
cd proxy
```

### Running NPM

Now we can create a new `compose.yaml and paste in the following.

```bash
nano compose.yaml
```

```yaml
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
Use `CTRL-O` to save the file and `CTRL-X` to back out of nano. Then spin up the service will the following command.

```bash
docker compose up -d
```

### NPM Setup and TLS

Now navigate to your admin dashboard from the browser `http://your.public.ip.address:81`and create your admin account with a strong password. The default credentials are:

Email: admin@example.com
Password: changeme

You'll be prompted to change these immediately after logging in.

#### Generate Let's Encrypt Certificates

Navigate to SSL Certificates > Add SSL Certifcate. Type in your root domain name (example.com) click add then input the wildcare domain (*.example.com) and then enable 'Use a DNS Challenge'. Select your registar and paste in the API we saved from eariler. If you run into error make sure that your API key is correct, whitelist your public IP with you registar if needed, or try increasing the Propagation Seconds to 120 seconds.

### Proxy NetBird

Back in Nginx Proxy Manager, we need to create proxy host entries for NetBird. You'll need to create three separate proxy hosts:

## NetBird Setup

### Onboading/Local User

### Adding a Peer

### Creating a Network

## Pocket ID

### Running Pocket ID

compose.yaml
```yaml
services:
  pocket-id:
    image: ghcr.io/pocket-id/pocket-id:v2
    restart: unless-stopped
    env_file: .env
    ports:
      - 1411:1411
    volumes:
      - "./data:/app/data"
      - "./key:/key"
    # Optional healthcheck
    healthcheck:
      test: [ "CMD", "/app/pocket-id", "healthcheck" ]
      interval: 1m30s
      timeout: 5s
      retries: 2
      start_period: 10s
```
 .env
```yaml
# See the documentation for more information: https://pocket-id.org/docs/configuration/environment-variables

# These variables must be configured for your deployment:
APP_URL=https://your-pocket-id-domain.com

# Method 2: File-based key
# Put the base64 key in a file and point to it here.
ENCRYPTION_KEY_FILE=/key/encryption_key

# These variables are optional but recommended to review:
TRUST_PROXY=true
# MAXMIND_LICENSE_KEY=
PUID=1000
PGID=1000
```

### Proxy Pocker ID with NPM

### Add Pocket ID to NetBird

## Remove NetBird Local User




