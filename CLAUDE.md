# CLAUDE.md

## Project Overview

TechHutTV's Homelab — a self-hosted infrastructure repository containing Docker Compose stacks, service configurations, and documentation for a complete home automation, media, monitoring, and surveillance environment. Runs primarily on Proxmox VE with ZFS storage across multiple servers.

This is a documentation-and-config repository, not a traditional application codebase. There are no builds, tests, or linters.

## Repository Structure

```
homelab/
├── apps/                        # App catalog with descriptions and images (no configs)
│   └── images/                  # Screenshots and logos for documented apps
├── cloud/                       # Nextcloud AIO + NGINX Proxy Manager compose stack
│   └── compose.yaml
├── homeassistant/               # Home Assistant configs and integration guides
│   ├── dashboard/               # Custom dashboard cards (bubble-card, mini-graph-card)
│   ├── localtuya/               # LocalTuya integration (xZetsubou fork via HACS)
│   └── zigbee/                  # Zigbee2MQTT + Mosquitto setup (Sonoff ZBDongle-E)
├── media/                       # Media server and *arr stack
│   ├── compose.yaml             # Main *arr stack + VPN + download clients
│   ├── .env                     # VPN credentials, timezone, UID/GID
│   ├── jellyfin/                # Jellyfin + Jellyseerr + Jellystat compose
│   │   └── compose.yaml
│   └── plex/                    # Plex + Overseerr + Tautulli compose
│       └── compose.yaml
├── monitoring/                  # Prometheus, Grafana, InfluxDB, Telegraf stack
│   ├── compose.yaml
│   ├── grafana/                 # Datasource provisioning (datasource.yml)
│   ├── prometheus/              # Scrape configs (prometheus.yml)
│   ├── telegraf/                # Agent config (telegraf.conf)
│   ├── proxmox/                 # Proxmox-specific monitoring guide
│   └── unraid/                  # Unraid telegraf variant config
├── proxy/                       # NGINX Proxy Manager, Cloudflare DDNS, Twingate
│   ├── compose.yaml
│   └── images/                  # Reference screenshots
├── storage/                     # ZFS pools, backups, SMB sharing docs (no compose)
├── surveillance/                # Frigate NVR with Coral TPU
│   ├── compose.yaml
│   └── frigate/                 # Frigate config.yml (cameras, detectors, recording)
├── glance.yml                   # Glance dashboard config (services, RSS, weather, markets)
└── timezones.properties         # Reference timezone list
```

## Key Technologies

| Category | Tools |
|----------|-------|
| Hypervisor | Proxmox VE, LXC containers |
| Container runtime | Docker, Docker Compose |
| Storage | ZFS (raidz1 pools: "flash" NVMe, "vault" HDD), SMB shares |
| Media | Plex, Jellyfin, Radarr, Sonarr, Lidarr, Prowlarr, Bazarr, Overseerr, Tautulli |
| Downloads | qBittorrent, NZBGet, Gluetun (VPN wrapper via AirVPN/WireGuard) |
| Smart home | Home Assistant, Zigbee2MQTT (Sonoff ZBDongle-E), Mosquitto MQTT, LocalTuya |
| Monitoring | Prometheus, Grafana, InfluxDB 2.x, Telegraf, Node Exporter |
| Networking | NGINX Proxy Manager, Cloudflare DDNS, Twingate, Pi-Hole, TP-Link Omada |
| Surveillance | Frigate NVR (dual Google Coral PCIe TPUs, 10 cameras, VAAPI hwaccel) |
| Cloud | Nextcloud AIO, Immich, Vaultwarden |
| Dashboard | Glance (dark theme, service monitoring, RSS, weather, markets) |
| Other | TeslaMate, OctoPrint, Docmost, Hoarder, File Browser |

## Docker Compose Stacks

Seven compose files define the full infrastructure:

| Stack | File | Key Services |
|-------|------|-------------|
| Media *arr | `media/compose.yaml` | Gluetun, qBittorrent, NZBGet, Prowlarr, Sonarr, Radarr, Lidarr, Bazarr, deunhealth |
| Jellyfin | `media/jellyfin/compose.yaml` | Jellyfin, Jellyseerr, Jellystat, PostgreSQL |
| Plex | `media/plex/compose.yaml` | Plex, Overseerr, Tautulli |
| Monitoring | `monitoring/compose.yaml` | Prometheus, Grafana, InfluxDB 2.7.6, Telegraf |
| Proxy | `proxy/compose.yaml` | NGINX Proxy Manager, Cloudflare DDNS, Twingate, helloworld (test) |
| Surveillance | `surveillance/compose.yaml` | Frigate (privileged, 64MB SHM) |
| Cloud | `cloud/compose.yaml` | Nextcloud AIO, NGINX Proxy Manager |

## File Conventions

### Docker Compose Files
- Named `compose.yaml` (not `docker-compose.yml`)
- Paired with `.env` files for secrets and environment-specific values
- Secrets and credentials go in `.env`, never in compose files
- Standard patterns:
  - `PUID=1000` / `PGID=1000` for container user mapping
  - `TZ=America/Los_Angeles` for timezone (or bind mount `/etc/localtime:/etc/localtime:ro`)
  - `restart: unless-stopped` on most services (`restart: always` on Telegraf, Nextcloud AIO, Twingate, deunhealth)
  - Named volumes for databases (e.g., `prom_data`, `grafana_data`, `influxdb2_data`)
  - Bind mounts for configs (`./config:/config`) and shared media (`/data:/data`)

### Network Patterns
- **Custom bridge with static IPs**: `servarrnetwork` (172.39.0.0/24) — Gluetun at .2, Sonarr at .3, Radarr at .4, Lidarr at .5, Bazarr at .6
- **VPN-wrapped services**: `network_mode: "service:gluetun"` for qBittorrent, NZBGet, Prowlarr (ports exposed on Gluetun container)
- **Host network mode**: NGINX Proxy Manager, Plex
- **Local network**: 10.0.0.x/24 range (e.g., MQTT broker at 10.0.0.50, Jellyfin at 10.0.0.101)
- **Extra hosts**: `dockerlab:host-gateway` for Prometheus and InfluxDB

### Health Checks
- **Gluetun**: `ping -c 1 www.google.com` (20s interval, 10s timeout, 5 retries)
- **qBittorrent**: `ping -c 1 www.google.com` (60s interval, 20s start period, 3 retries)
- **NGINX Proxy Manager**: `/usr/bin/check-health` (10s interval, 3s timeout)
- **deunhealth**: Monitors container health labels and auto-restarts unhealthy containers

### Hardware Passthrough
- **Intel QuickSync/VAAPI**: `/dev/dri:/dev/dri` or `/dev/dri/renderD128` (Jellyfin, Plex, Nextcloud AIO, Frigate)
- **Google Coral TPU**: USB via `/dev/bus/usb`, PCIe via `/dev/apex_0` and `/dev/apex_1` (Frigate uses dual PCIe: `pci:0`, `pci:1`)
- **LXC passthrough**: Requires `lxc.cgroup2.devices.allow` entries for USB (188:*, 189:*), DRI (226:0, 226:128, 29:0), serial devices, and `features: nesting=1`

### Documentation
- Each top-level directory has its own `README.md` (12 total)
- Markdown with tables, code blocks, and inline images
- "Work in Progress" sections clearly marked
- Paired with YouTube video guides (links in READMEs)
- Resource links to Proxmox Community Helper Scripts, GitHub repos, and external docs

## Important Patterns to Follow

1. **Never commit secrets** — credentials, API keys, WireGuard keys, and MQTT passwords belong in `.env` files which are not tracked
2. **Use compose.yaml naming** — not `docker-compose.yml` or other variants
3. **Maintain directory isolation** — each functional stack is self-contained with its own compose file, configs, and README
4. **Keep documentation paired** — config changes should have corresponding README updates
5. **Preserve existing formatting** — READMEs use a consistent style with tables, headers, resource links, and YouTube embeds
6. **Hardware passthrough** — GPU and TPU device mappings are documented in both compose files and LXC configs; changes need matching documentation
7. **Placeholder values in tracked files** — use obvious placeholders (`key`, `password`, `<ACCESS TOKEN>`, `example.com`) for credentials that must appear in tracked configs (Frigate MQTT, Zigbee2MQTT, Cloudflare DDNS)
8. **LinuxServer.io images preferred** — use `lscr.io/linuxserver/*` images when available (consistent PUID/PGID/TZ interface)

## Common Operations

```bash
# Start a stack
cd media && docker compose up -d

# Restart a single service
docker compose restart <service-name>

# View logs
docker compose logs -f <service-name>

# Update all containers in a stack
docker compose pull && docker compose up -d

# Check VPN connectivity (via gluetun)
docker exec gluetun ping -c 1 www.google.com
```

## Hardware Environment

### Servers
- **ZimaCube Pro** (Proxmox) — i5-1235U, 32GB DDR5, 3x1TB NVMe raidz1 + 6x4TB HDD raidz1
- **UGREEN DXP6800 Pro** (Proxmox) — i5-1235U, 16GB DDR5, 2x1TB NVMe cache + 5x4TB HDD RAID5 + 1x16TB parity, PCIe Coral TPU
- **Intel NUC** (Ubuntu 24.04) — i7-1360P, 32GB DDR5, 512GB NVMe (business services: Odoo, Nextcloud)

### Networking
- TP-Link Omada stack (hardware controller, multi-gig VPN router, PoE switches, access points)

## External Dependencies

- **Cloudflare** — DNS hosting, DDNS API tokens, SSL certificate generation
- **AirVPN** — VPN provider with WireGuard (requires port forwarding config)
- **Twingate** — Zero-trust remote access (network, access token, refresh token)
- **Tuya IoT Platform** — Cloud API credentials for LocalTuya (Client ID, Secret, User ID)
- **Google Coral TPU** — Hardware AI accelerator for Frigate object detection
- **Intel GPU/QuickSync** — Hardware video transcoding for Plex, Jellyfin, Nextcloud
- **Proxmox Community Helper Scripts** — Used for LXC setup (Zigbee2MQTT, InfluxDB, Grafana, Glance, Docker)
