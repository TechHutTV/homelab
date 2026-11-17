# My Homelab Setup

This repo contains Docker Compose stacks, configuration templates, and documentation for my homelab. It spans two servers: an Unraid NAS for storage, media, and the Immich and OpenCloud VMs, plus a Proxmox node for DNS, proxying, Vaultwarden, TeslaMate, and other guests. Ubiquiti networking ties it all together.

Whether you're looking for a complete Servarr media setup behind a VPN, a Frigate NVR config with Coral TPU, or just want to see how someone else organizes their self-hosted infrastructure, feel free to dig around and use whatever is helpful. This is a *work in progress* - I still have a ton to update and add.

## Table of Contents

* [Apps](apps/) - List of all the apps and services.
* [Cloud](cloud/) - Immich setup and self-hosted cloud service guides.
* [Home Assistant](homeassistant/) - Smart home services and automation.
* [Media Server](media/) - Plex, Jellyfin, *arr stack, and more.
* [Monitoring](monitoring/) - Graphs and visualizations for Unraid, Proxmox, and more.
* [NetBird](netbird/) - Self-hosted zero-trust networking with PocketID.
* [Proxy Management](proxy/) - NGINX Proxy Manager, DDNS with Cloudflare, local domains, and more.
* [Storage](storage/) - Current storage and backup solution.
* [Surveillance](surveillance/) - Frigate NVR solution with Coral TPU.

## Hardware

### Compute

| | OpnNAS SL-R12 Pro | Miniforum AI X1 Pro |
|---|---|---|
| **OS** | Unraid | Proxmox |
| **Hostname / IP** | `HopkiMedia` / `10.173.10.10` | `blackbox` / `10.173.10.12` |
| **Role** | Storage, media/download services, Immich and OpenCloud VMs, shared GPU ML worker | Technitium DNS, NPM, Vaultwarden, TeslaMate, and general-purpose VMs |
| **CPU** | [Intel Core i5-13500T](https://amzn.to/4qblt9y) | AMD Ryzen AI 9 HX 370 |
| **RAM** | [Crucial Pro 96GB DDR5](https://amzn.to/3Jbh2uD) | 96GB SODIMM DDR5 |
| **Storage** | [5x Seagate Exos 20TB](https://amzn.to/4nPhn5d), [7x Seagate IronWolf 4TB](https://amzn.to/3W1IFt5) | [2x WD SN580 1TB NVMe](https://amzn.to/4hdfHzY) (mirrored boot) |
| **Link** | [learn more](https://opnnas.com/#hls) | [get it here](https://amzn.to/4n0xTOR) |

The OpnNAS is an early release press model - check [their site](https://opnnas.com/#hls) for availability. I keep Immich and OpenCloud on Unraid so their photo and file shares stay local to the host. The Miniforum handles the lighter infrastructure services and other Proxmox guests.

### Networking

Ubiquiti sent all of this over to me. There is no way I'd be able to afford all this stuff on my own dime, so note that. This setup is crazy overpowered for what I need.

| Device | Description |
|---|---|
| **UDM Pro Max** | Primary router, gateway, and UniFi Protect NVR with 2x 8TB HDDs for camera storage. |
| **USW Pro Max 24 PoE** | Main 24-port PoE switch powering cameras and access points. |
| **USW Pro XG 10 PoE** | 10G aggregation switch for high-bandwidth connections between devices. |
| **2x U7 Pro XG** | Wi-Fi 7 access points providing wireless coverage. |
| **USW Lite 16 PoE** | Secondary 16-port PoE switch for additional wired devices. |

<details>
<summary>Accessories</summary>

* 1x U-Rack-6U-TL
* 1x UACC-Cable-Patch-EL-0.15M-W-24
* 5x UACC-Cable-Patch-EL-3M-W
* 2x UACC-Cable-Patch-EL-15M-W

</details>

### Security

Currently running two camera systems - originally fully Reolink, but added UniFi Protect after Ubiquiti sent equipment to try. See the [video overview](https://www.youtube.com/watch?v=2RiiKi4vjJc).

| Camera | Details |
|---|---|
| **3x UVC-G6-Turret-W** | Connected to the UDM Pro Max via UniFi Protect. Planning to slowly transition fully to Ubiquiti as budget allows. |
| **Reolink PoE** | [RLC-1224A 12MP](https://amzn.to/4d6BuGJ) and [4K Floodlight](https://amzn.to/4ceRKnM). Great for the Reolink ecosystem and 3rd party services like Frigate. |
| **Reolink NVR Kit** | [16CH 12MP](https://amzn.to/4d9YaWz) (my kit) - simple PoE plug-and-play solution. Kit cameras require the NVR to function. Budget option: [8CH 5MP](https://amzn.to/4dulGgG). |

### Other Devices

* **Raspberry Pi 5** (PoE HAT) - Twingate connector and Cloudflare DDNS updater.
* **Raspberry Pi 4** - OctoPrint for 3D printer management.
* **VPS** ([Hostinger KVM](https://www.hostinger.com/)) - Self-hosted NetBird management server.

> Previous hardware? See [old_hardware.md](old_hardware.md).

## Service Inventory

This inventory reflects the November 16, 2026 maintenance handoff. Running states are from that check.

### Unraid

`HopkiMedia` hosts the media and download containers, including Jellyfin, Plex, Tautulli, Seerr, the Servarr stack, Jellystat, Youtarr, and BookOrbit. The `immich-ml-gpu` container shares the Intel Arc Pro B60 with the media servers. All 20 host containers and both VMs were running at the last check.

| VM | Address | vCPU / RAM | File storage |
|---|---|---|---|
| `ubuntu-immich` | `10.173.10.31:2283` | 4 / 8 GiB | `/mnt/user/images` shared into the VM through VirtioFS |
| `ubuntu-opencloud` | `10.173.10.32:9200` | 2 / 8 GiB | `/mnt/user/opencloud` shared into the VM through VirtioFS |

Both VM system disks are on the mirrored NVMe pool. Nextcloud and Seafile have been retired.

### Proxmox

Workloads moved from the old host at `10.173.10.11` to `blackbox` at `10.173.10.12`. These are the final guest IDs after the migration. The old proxy LXC 102 has been removed.

| Guest | Name | Address / endpoint | CPU / RAM | Observed state / autostart |
|---|---|---|---|---|
| LXC 100 | `technitiumdns` | `10.173.10.100`, admin `:5380` | 2 cores / 1 GiB | Running / enabled |
| LXC 101 | `nginxproxymanager` | `10.173.10.101:81` | 2 cores / 2 GiB | Running / enabled |
| LXC 103 | `vaultwarden` | `10.173.10.103:8000` | 4 cores / 6 GiB | Running / enabled |
| LXC 104 | `teslamate` | `10.173.10.104`, app `:4000`, Grafana `:3000` | 2 cores / 4 GiB | Running / enabled |
| VM 200 | `truenas` | Not reverified | 4 vCPU / 4 GiB | Stopped / enabled |
| VM 201 | `ubuntu-gpu` | Not reverified | 8 vCPU / 32 GiB | Stopped / enabled |
| VM 202 | `ubuntu-dev` | Not reverified | 8 vCPU / 8 GiB | Stopped / disabled |
| VM 203 | `win11` | Not reverified | 6 vCPU / 12,336 MiB | Running / no explicit onboot setting |
| VM 204 | `haos` | Proxy target `10.173.10.204:8123` | 2 vCPU / 4 GiB | Running / enabled |

All four LXCs are unprivileged, with their root disks on `local-lvm`. Guest IDs don't establish an IP address, so I've left unverified addresses out. TrueNAS and `ubuntu-gpu` are stopped but configured to start on boot; the GPU VM needs 32 GiB RAM when running.

## Stars

[![Star History Chart](https://star-history.dera.page/svg?repos=techhuttv/homelab&type=date&legend=top-left)](https://star-history.dera.page/#techhuttv/homelab&type=date&legend=top-left)
