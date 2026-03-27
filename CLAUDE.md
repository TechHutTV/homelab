# CLAUDE.md — M3TA OS Project Configuration

## Project Overview

**Organization:** Eagle Eye Vision Labz, LLC (EEVL)
**System:** M3TA OS — File-Tree Agent Architecture
**Framework:** Jake Van Clief's "Agents as Folders"
**Repository:** homelab (EagleEyeVisionLabz/homelab)

## Tech Stack

### Infrastructure
- **Virtualization:** Proxmox (ZimaCube Pro i5-1235U/32GB, UGREEN DXP6800 Pro i5-1235U/16GB)
- **Business Server:** Intel NUC i7-1360P/32GB (Ubuntu 24.04 + Docker)
- **Networking:** TP-Link Omada (Hardware Controller, Multi-Gig VPN Router, PoE Switches, EAP610/EAP653 APs)
- **Remote Access:** Twingate (primary), Netbird (secondary/self-hosted)
- **DNS:** Pi-Hole (ad blocking + DNS sinkhole)
- **Proxy:** NGINX Proxy Manager + Cloudflare DDNS
- **Domain:** hopki.net (subdomains per service)
- **VPN:** AirVPN (for torrent traffic)
- **Backup:** Proxmox Backup Server

### Self-Hosted Services
- **Dashboard:** Glance (start page + service monitor)
- **Media Servers:** Plex (primary), Jellyfin (backup)
- **Media Management:** Radarr, Sonarr, Lidarr, Bazarr, Prowlarr, Overseerr
- **Download Clients:** qBittorrent, NZBGet
- **Cloud Storage:** Nextcloud (Google Drive replacement)
- **Photo Backup:** Immich (Google Photos replacement)
- **Wiki/Notes:** Docmost (Notion replacement)
- **Passwords:** Vaultwarden (Bitwarden-compatible)
- **Bookmarks:** Hoarder
- **Smart Home:** Home Assistant + Zigbee2MQTT
- **Surveillance:** Frigate NVR (Coral TPU)
- **3D Printing:** OctoPrint (Raspberry Pi 4)
- **Monitoring:** Grafana + Prometheus + InfluxDB2 + Telegraf
- **Vehicle:** TeslaMate
- **Docker Management:** Portainer, Cockpit
- **File Management:** File Browser

### EEVL Business Tools (External SaaS)
- **CRM:** GoHighLevel (GHL), Vtiger CRM (Club Trapeze)
- **Project Management:** ClickUp
- **Team Communication:** Lark (channels: #edited-by-sparrow, #gold-star, client channels)
- **Knowledge Base:** Notion, Kortex.ai, NotebookLM, MyMind AI (second brain)
- **Second Brain:** MyMind AI (personal knowledge capture, AI auto-categorization, document synthesis)
- **Database:** Airtable (rosters, inventories, CRM supplements)
- **Automation:** n8n (primary), Make.com (secondary)
- **Payments:** Stripe + QuickBooks
- **Contracts:** PandaDoc
- **Design:** Canva API (brand kit integration)
- **Email:** Gmail (automation + delivery)
- **AI Models:** Claude (Opus/Sonnet via Anthropic), GPT-4o (via OpenRouter), Gemini, NVIDIA Nematron 3 Super (120B MoE, 12B active, 1M context, agentic workloads), Nematron 3 Nano (lightweight/edge)
- **Agent Platforms:** NemoClaw (NVIDIA enterprise agent platform, hardware-agnostic, security-first), OpenClaw ecosystem awareness (NanoClaw, ZeroClaw, PicoClaw, IronClaw, MultiClaw variants)
- **Web Scraping:** Apify (TikTok, Reddit)
- **Post-Production:** Edited By Sparrow (external editor, Lark handoff)

### M3TA OS Architecture
- **Location:** `meta-os/` directory
- **Orchestrator:** `meta-os/orchestrator.json`
- **Workflows:** 5 pillars in `meta-os/workflows/`
  - Media Production, Content Pipeline, Gold Star Flag Football, Club Trapeze, Client Onboarding
- **Sub-Agents:** 8 agents in `meta-os/sub-agents/`
  - MeetingScribe, DocuMind, CodeReview AI, TaskMaster AI, EEVL Creative, SocialBot, Sparrow Handoff, Onboarding Agent
- **MCP Servers:** 11 configs in `meta-os/mcp-servers/` (includes Claude Computer Use, Cowork/Dispatch)
- **Engines:** 8 configs in `meta-os/engines/` (includes Agent Escalation, Apple Native)
- **Knowledge Base:** 8 configs in `meta-os/knowledge-base/` (includes MyMind AI second brain)
- **Skills:** 5 MyMind AI skills in `meta-os/skills/mymind-ai/` (vault-setup, daily-brief, tlddr, canvas-gen, pdf-synthesize)
- **Auto Research:** Self-improving skill system in `meta-os/auto-research/`

## Code Conventions

- **Config format:** JSON for machine-readable configs, YAML for Docker Compose and Glance
- **Agent prompts:** Markdown (`prompt.md`) — natural language instructions
- **Tool specs:** JSON (`tools.json`) — API/service connection definitions
- **Data schemas:** JSON (`schema.json`) — input/output contracts
- **Workflow manifests:** JSON (`workflow.json`) — execution order, triggers, dependencies
- **Naming:** kebab-case for directories and files
- **File naming (media):** `YYYY-MM-DD_ClientName_ProjectType_###`

## Key Business Rules

- **60-second SLA:** All GHL lead captures must trigger SMS response within 60 seconds
- **Sparrow Handoff:** Raw media → #edited-by-sparrow Lark channel (or #gold-star for Gold Star projects)
- **Brand Voice:** Club Trapeze = upscale Tamarac FL nightlife tone; EEVL = professional media production
- **Model Preference:** Claude Opus for deep analysis, Claude Sonnet for general tasks, NVIDIA Nematron 3 Super for persistent agentic workloads (MoE: 12B active / 120B total, 1M context, 5x speed), GPT-4o for vision/creative, Gemini Flash for speed/bulk, Nematron 3 Nano for edge/routing
- **Agent Platform:** NemoClaw for enterprise agent deployment (hardware-agnostic, open-source, security controls). Do NOT use raw OpenClaw in production (security vulnerabilities). Claw variants (NanoClaw, ZeroClaw, IronClaw) for specialized deployments
- **Agent Escalation:** 3-tier model — Tier 1 (local agents, Shortcuts, APIs, scripts = 90% of work) → Tier 2 (Cowork + Dispatch for knowledge work) → Tier 3 (Computer Use for GUI-only emergencies). Never skip tiers.
- **Computer Use:** Tier 3 only — expensive, 10 min hard limit, user approval required. Valid: no-API portals, broken automation rescue, creator app deep UI. Forbidden: financial apps, Vaultwarden, email sending, admin panels.
- **Apple Native:** Shortcuts + Siri for voice triggers, AppleScript/JXA for Omni suite and creator apps, launchd for scheduled jobs. Computer Use is the fallback for the 10-20% of "weird UI" jobs.
- **Future-Proof:** When new AI capabilities launch, update the leaf node (prompt.md or tools.json) — never rebuild the structure

## Development

- **Primary branch:** main
- **Feature branches:** `claude/<feature>-<session-id>`
- **Docker:** All services run as Docker containers with compose files
- **Network:** Services accessible via `*.hopki.net` subdomains through NGINX Proxy Manager

## MyMind AI — Second Brain Integration

MyMind AI is the personal cognitive layer of M3TA OS. Skills in `meta-os/skills/mymind-ai/`:

| Command | Purpose |
|---------|---------|
| `/vault-setup` | Interactive vault builder with multiple-choice guided setup |
| `/daily-brief` | Morning priority digest across all vault sections |
| `/tlddr` | End-of-conversation summary → auto-categorized into vault |
| `/canvas-gen` | Visual knowledge maps, process diagrams, architecture views |
| `/pdf-synthesize` | Large PDF → clean markdown cheat sheet → vault |

### Context Injection
Always reference MyMind vault when:
- Brainstorming content ideas → check `content-ideas/`
- Preparing for client work → check `client-notes/<client>/`
- Starting a new project → check `projects/` for related past work
- Looking for creative references → check `media-library/`

### Document Synthesis Pipeline
Raw PDF → Claude/Gemini extraction → salient points only → markdown cheat sheet → auto-categorized in vault → cross-referenced with existing notes

## Claude Code Guide Integration

Use the built-in `claude-code-guide` sub-agent for real-time diagnostics and configuration:

- **Tag it** to diagnose broken hooks, misconfigured CLAUDE.md rules, or skill issues
- **Point it at files** — drag/drop or tag any config file for expert analysis
- **Build with it** — create custom slash commands, hooks, and automation workflows
- **5 Levels of Use:**
  1. Q&A — Ask about any Claude Code feature (slash commands, model switching, hooks)
  2. Diagnostics — Tag your CLAUDE.md and have it identify what's missing or broken
  3. Deep Mechanics — Understand all 17 hook event types and their JSON configuration
  4. Build — Create custom `/standup`, `/deploy`, or any slash command with associated hooks
  5. Repair — Point it at broken hooks/configs and let it read docs, diagnose, and fix in real-time

### Auto Research + Claude Code Guide

Combine Auto Research with Claude Code Guide for maximum skill optimization:
1. Use Claude Code Guide to understand how skills/hooks work
2. Create binary eval suites in `meta-os/auto-research/evals/`
3. Run `/loop 5m` to execute auto-research optimization cycles
4. Claude Code Guide can diagnose why evals are failing and suggest prompt mutations

## CLAUDECLAW Integration

See `CLAUDECLAW.md` for the bridge layer connecting this homelab infrastructure to the M3TA OS agent architecture — mapping self-hosted services to MCP server connections and automation triggers.
