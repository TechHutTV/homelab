# CLAUDE.md

## Project Overview

TechHutTV's Homelab — a self-hosted infrastructure repository containing Docker Compose stacks, service configurations, and documentation for a complete home automation, media, and monitoring environment. Runs primarily on Proxmox VE with ZFS storage.

This is a documentation-and-config repository, not a traditional application codebase. There are no builds, tests, or linters.

## Repository Structure

```
homelab/
├── apps/              # App catalog with descriptions and images (no configs)
├── cloud/             # Nextcloud AIO compose stack
├── homeassistant/     # Home Assistant configs, dashboards, Zigbee, LocalTuya
├── media/             # Media server and *arr stack (Radarr, Sonarr, etc.)
│   ├── jellyfin/      # Jellyfin compose and setup docs
│   └── plex/          # Plex setup docs
├── monitoring/        # Prometheus, Grafana, InfluxDB, Telegraf stack
│   ├── grafana/       # Datasource provisioning
│   ├── prometheus/    # Scrape configs
│   └── telegraf/      # Agent config
├── proxy/             # NGINX Proxy Manager, Cloudflare DDNS, Twingate
├── storage/           # ZFS pools, backups, SMB sharing docs
├── surveillance/      # Frigate NVR with Coral TPU
│   └── frigate/       # Frigate config.yml
├── glance.yml         # Dashboard/start page configuration
└── timezones.properties
```

## Key Technologies

| Category | Tools |
|----------|-------|
| Hypervisor | Proxmox VE, LXC containers |
| Container runtime | Docker, Docker Compose |
| Storage | ZFS (raidz1 pools: "flash" for NVMe, "vault" for HDD) |
| Media | Plex, Jellyfin, Radarr, Sonarr, Lidarr, Prowlarr, Bazarr, Overseerr |
| Downloads | qBittorrent, NZBGet, Gluetun (VPN wrapper) |
| Smart home | Home Assistant, Zigbee2MQTT, Mosquitto MQTT, LocalTuya |
| Monitoring | Prometheus, Grafana, InfluxDB 2.x, Telegraf, Node Exporter |
| Networking | NGINX Proxy Manager, Cloudflare DDNS, Twingate, Pi-Hole |
| Surveillance | Frigate NVR (Google Coral TPU) |
| Cloud | Nextcloud AIO, Immich, Vaultwarden |

## File Conventions

### Docker Compose Files
- Named `compose.yaml` (not `docker-compose.yml`)
- Paired with `.env` files for secrets and environment-specific values
- Secrets and credentials go in `.env`, never in compose files
- Standard patterns:
  - `PUID=1000` / `PGID=1000` for container user mapping
  - `restart: unless-stopped` on all services
  - `/etc/localtime:/etc/localtime:ro` bind mount for timezone
  - Named volumes for databases, bind mounts for configs and media

### Network Patterns
- Custom Docker bridge networks with defined subnets (e.g., `servarrnetwork` at 172.39.0.0/24)
- VPN-wrapped services use `network_mode: "service:gluetun"`
- NGINX Proxy Manager runs in host network mode
- Local network uses 10.0.0.x/24 range

### Documentation
- Each top-level directory has its own `README.md`
- Markdown with tables, code blocks, and inline images
- "Work in Progress" sections clearly marked
- Paired with YouTube video guides (links in READMEs)

## Important Patterns to Follow

1. **Never commit secrets** — credentials, API keys, and WireGuard keys belong in `.env` files which are not tracked
2. **Use compose.yaml naming** — not `docker-compose.yml` or other variants
3. **Maintain directory isolation** — each functional stack (media, monitoring, proxy, etc.) is self-contained with its own compose file and configs
4. **Keep documentation paired** — config changes should have corresponding README updates
5. **Preserve existing formatting** — READMEs use a consistent style with tables, headers, and resource links
6. **Hardware passthrough** — GPU and TPU device mappings are documented in compose files and LXC configs; changes need matching documentation in README

## Common Operations

```bash
# Start a stack
cd media && docker compose up -d

# Restart a service
docker compose restart <service-name>

# View logs
docker compose logs -f <service-name>

# Update containers
docker compose pull && docker compose up -d
```

## External Dependencies

- **Cloudflare** — DNS hosting and DDNS API
- **AirVPN** — VPN provider (configurable via Gluetun)
- **Twingate** — Zero-trust remote access
- **Tuya IoT Platform** — Smart device cloud API (for LocalTuya)
- **Google Coral TPU** — Hardware AI accelerator for Frigate
- **Intel GPU/QuickSync** — Hardware video transcoding for Plex/Jellyfin
