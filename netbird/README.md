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
    networks:
      - proxy
    volumes:
      - ./data:/data
      - ./letsencrypt:/etc/letsencrypt

networks:
  proxy:
    name: proxy
    driver: bridge
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

For this guide, select option `[3]` to setup Nginx Proxy Manager. If you're using another service select it or select `[0]` Built-in Caddy if you want the script to handle all of that for you including automatice TSL certifications. 

```bash
Should container ports be bound to localhost only (127.0.0.1)?
Choose 'yes' if your reverse proxy runs on the same host (more secure).
Bind to localhost only? [Y/n]: 

Is Nginx Proxy Manager running in Docker?
If yes, enter the Docker network Nginx Proxy Manager is on (NetBird will join it).
Docker network (leave empty if not in Docker):
```

In the next options set ports bound to localhost and input the docker network for NGINX. Once the options are selected it will set everything up and spin up our containers. Since we have more to do we can spin down the stack for now.

```bash
docker compose down
```
### NetBird NPM Setup

Back in Nginx Proxy Manager, we need to create a proxy host entry for NetBird. The installation script generates an npm-advanced-config.txt file in your netbird directory with the configuration you'll need.

Back in Nginx Proxy Manager, we need to create a proxy host entry for NetBird. The installation script generates an `npm-advanced-config.txt` file in your netbird directory with the configuration you'll need.

1. Go to Hosts > Proxy Hosts > Add Proxy Host
2. Enter your NetBird domain (e.g., `netbird.example.com`)
3. Set Forward Hostname/IP to `127.0.0.1` and Forward Port to `80`
4. Under the SSL tab:
   - Select your wildcard certificate
   - Enable "Force SSL"
   - Enable "HTTP/2 Support" (required for gRPC)
5. Under the Advanced tab, paste the following configuration:

```nginx
# Required for long-lived connections (gRPC and WebSocket)
client_header_timeout 1d;
client_body_timeout 1d;

# Relay WebSocket
location /relay {
    proxy_pass http://netbird-relay:80;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "upgrade";
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
    proxy_read_timeout 1d;
}

# Signal WebSocket
location /ws-proxy/signal {
    proxy_pass http://netbird-signal:80;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "upgrade";
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
    proxy_read_timeout 1d;
}

# Management WebSocket
location /ws-proxy/management {
    proxy_pass http://netbird-management:80;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "upgrade";
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
    proxy_read_timeout 1d;
}

# API routes
location /api/ {
    proxy_pass http://netbird-management:80;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
}

# OAuth2/IdP routes
location /oauth2/ {
    proxy_pass http://netbird-management:80;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
}

# gRPC for Signal service
location /signalexchange.SignalExchange/ {
    grpc_pass grpc://netbird-signal:10000;
    grpc_read_timeout 1d;
    grpc_send_timeout 1d;
    grpc_socket_keepalive on;
}

# gRPC for Management service
location /management.ManagementService/ {
    grpc_pass grpc://netbird-management:80;
    grpc_read_timeout 1d;
    grpc_send_timeout 1d;
    grpc_socket_keepalive on;
}
```

> **Important:** For this configuration to work, NPM needs to be on the same Docker network as the NetBird containers. You can either run NPM in host mode or add it to the NetBird network.

Once configured, spin the NetBird stack back up:

### Onboarding/Local User

When you first access the NetBird dashboard at `https://netbird.example.com`, you'll be prompted to create an initial admin account. This uses NetBird's built-in identity provider which is perfect for getting started quickly.

1. Click "Create Account" and enter your email and a strong password
2. Verify your email if prompted
3. You'll be logged into the NetBird dashboard

This local user method works great for personal setups or small teams. Later in this guide, we'll configure PocketID as an external identity provider for more robust authentication with passkey support.

### Adding a Peer

To connect a device to your NetBird network:

1. In the dashboard, navigate to Peers > Add Peer
2. Copy the setup key or use the one-liner install command provided
3. On your device, install the NetBird client:

```bash
curl -fsSL https://pkgs.netbird.io/install.sh | sh
```

4. Connect using the setup key:

```bash
sudo netbird up --setup-key <YOUR_SETUP_KEY>
```

Alternatively, you can use the interactive login flow:

```bash
sudo netbird up --management-url https://netbird.example.com
```

Your device should now appear in the Peers list with a NetBird IP assigned.

### Creating a Network

Networks in NetBird allow you to expose resources to your peers without installing the NetBird client on every machine.

1. Navigate to Networks > Add Network
2. Give it a descriptive name (e.g., "Home Lab")
3. Add a network resource by specifying the IP range or specific hosts you want to expose (e.g., `192.168.1.0/24` for your home network)
4. Assign a routing peer - this is a device that has access to the target network and will route traffic for other peers
5. Create a policy to control which peers can access this network

This is incredibly useful for accessing your entire home network through a single peer running NetBird.

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

Once PocketID is configured and working, you may want to disable the local user authentication to enforce all logins through your IdP.

1. Log into NetBird using your PocketID credentials to verify it works
2. In the NetBird dashboard, go to Users and change ownership to the Pocket ID user
3. Remove any local user accounts that were created during initial setup

Note: Make sure you can successfully authenticate via PocketID before disabling local users, or you may lock yourself out of the dashboard.




