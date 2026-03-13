# My Homelab Setup

This repo contains Docker Compose stacks, configuration templates, and documentation for everything running in my homelab. It spans two servers - an Unraid NAS for media and a Proxmox node for everything else - tied together with Ubiquiti networking, self-hosted cloud services, and a full monitoring stack.

Whether you're looking for a complete Servarr media setup behind a VPN, a Frigate NVR config with Coral TPU, or just want to see how someone else organizes their self-hosted infrastructure, feel free to dig around and use whatever is helpful. This is a *work in progress* - I still have a ton to update and add.

## Table of Contents

* [Apps](apps/) - List of all the apps and services.
* [Automations](automations/) - n8n workflows and automation.
* [Cloud](cloud/) - NextCloud, Immich, and self-hosted cloud services.
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
| **Role** | Media server, NextCloud VM | Grafana, NPM, n8n, and other services |
| **CPU** | [Intel Core i5-13500T](https://amzn.to/4qblt9y) | AMD Ryzen AI 9 HX 370 |
| **RAM** | [Crucial Pro 96GB DDR5](https://amzn.to/3Jbh2uD) | 96GB SODIMM DDR5 |
| **Storage** | [5x Seagate Exos 20TB](https://amzn.to/4nPhn5d), [7x Seagate IronWolf 4TB](https://amzn.to/3W1IFt5) | [2x WD SN580 1TB NVMe](https://amzn.to/4hdfHzY) (mirrored boot) |
| **Link** | [learn more](https://opnnas.com/#hls) | [get it here](https://amzn.to/4n0xTOR) |

The OpnNAS is an early release press model - check [their site](https://opnnas.com/#hls) for availability. The Miniforum handles all non-media workloads as a dedicated Proxmox node.

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

## Stars

[![Star History Chart](https://api.star-history.com/svg?repos=techhuttv/homelab&type=date&legend=top-left)](https://www.star-history.com/#techhuttv/homelab&type=date&legend=top-left)
