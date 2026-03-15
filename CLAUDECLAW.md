# CLAUDECLAW — M3TA OS Integration Bridge

**Eagle Eye Vision Labz, LLC**
*Connecting homelab infrastructure to the M3TA OS agent architecture*

CLAUDECLAW is the integration layer that maps your self-hosted services, external SaaS tools, and automation engines into a unified system that M3TA OS can read and execute. Think of it as the nervous system connecting the brain (M3TA OS orchestrator) to the body (your actual tools and services).

## Architecture Overview

```
CLAUDECLAW Bridge
├── Homelab Infrastructure (self-hosted)
│   ├── Proxmox Cluster (compute)
│   ├── Docker Services (runtime)
│   ├── NGINX Proxy Manager (routing)
│   ├── Glance Dashboard (monitoring)
│   └── Network Stack (Omada)
│
├── Business SaaS Layer (external)
│   ├── GoHighLevel (CRM + SMS + Funnels)
│   ├── ClickUp (Project Management)
│   ├── Lark (Team Communication)
│   ├── Stripe + QuickBooks (Payments)
│   ├── PandaDoc (Contracts)
│   ├── Canva (Design)
│   ├── Notion + Kortex.ai (Knowledge)
│   └── Airtable (Structured Data)
│
├── AI Model Layer (inference)
│   ├── Anthropic Claude (Opus 4.6 / Sonnet 4.6)
│   ├── OpenAI GPT-4o (via OpenRouter)
│   ├── Google Gemini Flash
│   └── OpenRouter (routing hub)
│
└── M3TA OS Agent Layer (orchestration)
    ├── Orchestrator (meta-os/orchestrator.json)
    ├── 5 Workflow Pillars
    ├── 8 Sub-Agents
    ├── 9 MCP Servers
    └── Auto Research (self-improvement)
```

## Service-to-MCP Mapping

This table maps every active service in the homelab and SaaS stack to its corresponding M3TA OS MCP server configuration.

### Self-Hosted Services → M3TA OS

| Service | Homelab Location | M3TA OS Connection | Purpose |
|---------|-----------------|-------------------|---------|
| Proxmox | `10.0.0.80:8006` / proxmox.hopki.net | Infrastructure (not MCP-managed) | Compute orchestration |
| NGINX Proxy Manager | `10.0.0.102:81` / proxy.hopki.net | Infrastructure (routing layer) | Service routing + SSL |
| Pi-Hole | `10.0.0.103:80` / pihole.hopki.net | Infrastructure (DNS) | Ad blocking + DNS |
| Nextcloud | `10.0.0.104:11000` / cloud.hopki.net | `mcp-servers/google-drive.json` | File sync (supplements Google Drive) |
| Immich | `10.0.0.104:2283` / immich.hopki.net | `mcp-servers/google-drive.json` | Photo/video asset backup |
| Grafana | `10.0.0.101:3001` / grafana.hopki.net | `engines/n8n.json` | Monitoring dashboards |
| Prometheus | Monitoring stack | `engines/n8n.json` | Metrics collection |
| InfluxDB2 | Monitoring stack | `engines/n8n.json` | Time-series data |
| Home Assistant | `10.0.0.250:8123` / hass.hopki.net | `engines/n8n.json` | Smart home triggers |
| Frigate NVR | `10.0.0.107:5000` / frigate.hopki.net | `engines/n8n.json` | Security event triggers |
| Plex | `10.0.0.100:32400` / plex.hopki.net | Content delivery (not MCP-managed) | Media streaming |
| Docmost | `10.0.0.104:3001` / doc.hopki.net | `mcp-servers/notion.json` | Internal wiki (supplements Notion) |
| Glance | Dashboard | Infrastructure (monitoring) | Service health overview |
| Vaultwarden | `10.0.0.109:8000` / warden.hopki.net | Infrastructure (secrets) | Credential management |

### External SaaS → M3TA OS MCP Servers

| Service | MCP Config | Workflows Using It |
|---------|-----------|-------------------|
| Google Drive | `mcp-servers/google-drive.json` | All 5 workflows, all sub-agents |
| Gmail | `mcp-servers/gmail.json` | Client Delivery, Client Onboarding, Club Trapeze, Season Report |
| Stripe | `mcp-servers/stripe.json` | Client Delivery, Registrations, Contract-Invoice, Revenue Reporting |
| ClickUp | `mcp-servers/clickup.json` | All 5 workflows, TaskMaster AI, MeetingScribe |
| Notion | `mcp-servers/notion.json` | Client Onboarding, Content Pipeline, MeetingScribe, DocuMind |
| Claude / GPT / Gemini | `mcp-servers/llm-router.json` | All sub-agents, Content Creation, Ops Strategy |
| Airtable + GHL | `mcp-servers/airtable-ghl.json` | Gold Star, Client Onboarding, Club Trapeze |
| Lark | `mcp-servers/lark.json` | All workflows (channels), MeetingScribe, Sparrow Handoff |
| Apify / Web Scrape | `mcp-servers/web-scrape.json` | Content Pipeline (Idea Harvest) |

### Automation Engines → M3TA OS

| Engine | Config | Role in M3TA OS |
|--------|--------|----------------|
| n8n | `engines/n8n.json` | Primary workflow automation — webhook triggers, HTTP requests, conditional logic. Existing n8n workflows to be gradually migrated to read from M3TA OS folder structure |
| Make.com | `engines/make.json` | Secondary automation for complex multi-step scenarios |
| GoHighLevel | `engines/ghl.json` | CRM automation — SMS sequences, email drips, pipeline management, form processing |
| ClickUp AI | `engines/clickup-ai.json` | Task auto-assignment, status updates, dependency management |
| Lark Bot | `engines/lark-bot.json` | Channel management, message routing, meeting scheduling |
| Airtable Automations | `engines/airtable-automations.json` | Database-triggered workflows for rosters and reporting |

## Lark Channel Architecture

Critical integration point — Lark channels are the primary real-time communication layer for M3TA OS.

| Channel | Purpose | Triggered By |
|---------|---------|-------------|
| `#edited-by-sparrow` | Default post-production handoff to Edited By Sparrow | Sparrow Handoff Agent |
| `#gold-star` | Gold Star Flag Football media and operations | Event Coverage workflow |
| `#club-trapeze` | Club Trapeze operations and approvals | Club Trapeze workflows |
| `#client-<name>` | Per-client project channel (created at onboarding) | Onboarding Agent |
| `#content-pipeline` | Content creation approvals and distribution | SocialBot |
| `#sales` | Lead notifications and pipeline updates | Lead Capture workflow |

## Data Flow Patterns

### Pattern 1: Lead → Client (Client Onboarding)
```
GHL Form Submit
  → 60-sec SMS (GHL)
  → Lead Score (GHL auto-tag)
  → Lark #sales notification
  → Sales Call (MeetingScribe → Notion + ClickUp)
  → Contract (PandaDoc → Stripe → QuickBooks)
  → Kickoff (Onboarding Agent → ClickUp + Lark + Notion + Gmail)
```

### Pattern 2: Shoot → Delivery (Media Production)
```
ClickUp Task Created
  → Pre-Production (shot list, equipment, Lark channel)
  → Shoot Day (Lark updates, raw file upload to Google Drive)
  → Sparrow Handoff Agent → Lark #edited-by-sparrow
  → Post-Production (revision tracking)
  → Client Delivery (Google Drive share, Stripe invoice, Gmail)
```

### Pattern 3: Idea → Published (Content Pipeline)
```
Scheduled / Manual Trigger
  → Idea Harvest (Apify scrape → Kortex.ai KB → ClickUp calendar)
  → Content Creation (OpenRouter/GPT-4o → blog + captions)
  → Design Assets (Canva API → brand kit graphics)
  → SocialBot → FB, IG, TW, LI, GMB, newsletter
```

### Pattern 4: Season Lifecycle (Gold Star)
```
Seasonal Trigger
  → Tryout Campaign (Canva flyers, GHL blasts)
  → Registrations (GHL form → 60-sec SMS → Airtable roster → ClickUp)
  → Event Coverage (shoot → Sparrow → #gold-star)
  → Season Report (Airtable stats → PDF → Gmail to parents)
```

### Pattern 5: Weekly Operations (Club Trapeze)
```
Weekly Trigger
  → Ops Strategy (Vtiger KPI → recommendations for Fred)
  → Social Media (5 posts/week, upscale Tamarac tone)
  → Revenue Report (Stripe + QuickBooks → exec summary)
  → Event Promotion (brief → Canva flyer → email + social)
```

## Auto Research Integration

The Auto Research system (`meta-os/auto-research/`) can target any `prompt.md` in the M3TA OS file tree for autonomous optimization. Current eval suites:

| Eval | Target Prompt | Target Score | Criteria |
|------|--------------|-------------|----------|
| `media-production-brief` | Pre-production prompt | 90% | 6 binary criteria |
| `client-onboarding` | Lead capture prompt | 90% | 7 binary criteria |
| `content-pipeline` | Content creation prompt | 85% | 6 binary criteria |
| `social-bot` | SocialBot prompt | 90% | 5 binary criteria |
| `club-trapeze-social` | Club Trapeze social prompt | 90% | 5 binary criteria |

## Migration Path

The existing homelab infrastructure doesn't need to be rebuilt. CLAUDECLAW maps what exists into the M3TA OS folder model:

1. **n8n workflows** → Externalize prompts into `prompt.md` files, reference via `workflow.json`
2. **ClickUp projects** → Map to `workflow.json` task sequences
3. **Lark channels** → Already mapped to sub-agent triggers
4. **Notion pages** → Serve as knowledge base layer
5. **Google Drive folders** → Asset storage referenced by `tools.json`
6. **Docker services** → Infrastructure layer managed by Proxmox + Portainer

The model is replaceable. The structure is permanent. When a new AI capability launches, update the leaf node — never rebuild the tree.

## Quick Reference

```bash
# M3TA OS file tree
meta-os/
├── orchestrator.json          # Master config
├── workflows/                 # 5 business pillars
│   ├── media-production/      # Video + Drone + Photo
│   ├── content-pipeline/      # Idea → Published
│   ├── gold-star-flag-football/  # Season lifecycle
│   ├── club-trapeze/          # Weekly ops for Fred
│   └── client-onboarding/     # Lead → Client
├── sub-agents/                # 8 specialized agents
├── mcp-servers/               # 9 external service configs
├── engines/                   # 6 automation runtimes
├── knowledge-base/            # 7 data layer configs
└── auto-research/             # Self-improving skill system
    ├── program.md             # Optimization agent instructions
    ├── evals/                 # Binary eval suites
    ├── results/               # Run history
    └── run-config.json        # Defaults + rotation schedule
```
