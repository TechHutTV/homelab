# M3TA OS — File-Tree Agent Architecture

**Eagle Eye Vision Labz, LLC**
Built on Jake Van Clief's "Agents as Folders" Framework

> "People building agents are operating at the wrong abstraction layer. Map every agent to a file tree." — Jake Van Clief

## Core Principle

Instead of coding complex agent frameworks (LangChain, Semantic Kernel, etc.), every AI process is structured as a **file tree**. Each "folder" is a workflow, task, or sub-task. Each "file" inside is a prompt, tool connection, or data source. One single AI agent reads the folder tree and executes — and it will **never be made obsolete by a model update** because the structure is model-agnostic.

## Architecture Layers

| Layer | Directory | Purpose |
|-------|-----------|---------|
| Root | `meta-os/` | Master M3TA OS agent — governs all workflows |
| Workflows | `workflows/` | 5 business pillars with sequential task folders |
| Sub-Agents | `sub-agents/` | Specialized agents (MeetingScribe, DocuMind, etc.) |
| MCP Servers | `mcp-servers/` | External service connections (Google, Stripe, Lark, etc.) |
| Engines | `engines/` | Automation runtimes (n8n, Make.com, GHL, etc.) |
| Knowledge Base | `knowledge-base/` | Data layer (Kortex.ai, Notion, Airtable, etc.) |

## Workflow Pillars

| # | Pillar | Directory | Tasks |
|---|--------|-----------|-------|
| 1 | Media Production | `workflows/media-production/` | Pre-Production → Shoot Day → Post-Production → Client Delivery |
| 2 | Content Pipeline | `workflows/content-pipeline/` | Idea Harvest → Content Creation → Design Assets → Multi-Platform Distribution |
| 3 | Gold Star Flag Football | `workflows/gold-star-flag-football/` | Tryout Campaign → Registrations → Event Coverage → Season Report |
| 4 | Club Trapeze | `workflows/club-trapeze/` | Ops Strategy → Social Media Management → Revenue Reporting → Event Promotion |
| 5 | Client Onboarding | `workflows/client-onboarding/` | Lead Capture → Sales Call → Contract & Invoice → Project Kickoff |

## Sub-Agents

| Agent | Purpose | Directory |
|-------|---------|-----------|
| MeetingScribe | Lark meeting audio → Notion summary → ClickUp tasks | `sub-agents/meeting-scribe/` |
| DocuMind | Contract/doc analysis → key term extraction → risk flagging | `sub-agents/docu-mind/` |
| CodeReview AI | GitHub PR review → code feedback | `sub-agents/code-review/` |
| TaskMaster AI | ClickUp board analysis → auto-assignment by skill/workload | `sub-agents/task-master/` |
| EEVL Creative | Brand-kit design generation and asset management | `sub-agents/eevl-creative/` |
| SocialBot | Multi-platform content distribution automation | `sub-agents/social-bot/` |
| Sparrow Handoff | Raw file packaging → Edited By Sparrow delivery | `sub-agents/sparrow-handoff/` |
| Onboarding Agent | Full client kickoff sequence automation | `sub-agents/onboarding-agent/` |

## MCP Server Connections

| Server | Config | Services |
|--------|--------|----------|
| Google Drive | `mcp-servers/google-drive.json` | File storage, shared drives, asset management |
| Gmail | `mcp-servers/gmail.json` | Email automation, invoice delivery, notifications |
| Stripe | `mcp-servers/stripe.json` | Payments, invoicing, subscription management |
| ClickUp | `mcp-servers/clickup.json` | Project management, task tracking, automations |
| Notion | `mcp-servers/notion.json` | Knowledge base, wikis, client workspaces |
| Claude / GPT | `mcp-servers/llm-router.json` | AI model routing (Claude, GPT-4o, Gemini) |
| Airtable / GHL | `mcp-servers/airtable-ghl.json` | CRM, lead management, roster tracking |
| Lark | `mcp-servers/lark.json` | Team messaging, channels, meeting integration |
| Web Scrape | `mcp-servers/web-scrape.json` | Apify, content harvesting, trend monitoring |

## Why This Is Future-Proof

| Scenario | Old Way (Framework) | M3TA OS Way (File Tree) |
|----------|---------------------|-------------------------|
| OpenAI releases auto-invoicing | Rebuild LangChain agent | Update `contract-invoice/prompts/prompt.md` |
| Anthropic releases shot-list tool | Rebuild workflow | Add tool to `pre-production/tools/` |
| GHL adds AI follow-up | Rebuild automation | Condense `lead-capture/` into 1 tool call |
| New social platform emerges | Re-architect posting agent | Add new MCP to `multi-platform-distribution/tools/` |
| Add new business pillar | New codebase | New `workflows/<pillar-name>/` folder |

## Quick Start

1. Each workflow folder contains a `workflow.json` manifest describing its tasks, dependencies, and execution order
2. Each task folder contains `prompts/`, `tools/`, and `data/` subdirectories
3. Sub-agents are registered in `sub-agents/` with their own prompt/tool/data structure
4. MCP servers are configured in `mcp-servers/` as JSON connection specs
5. The root `orchestrator.json` governs the entire system

## File Conventions

- `prompt.md` — Natural language instructions for the AI agent executing this task
- `tools.json` — Tool/API connections required for this task
- `schema.json` — Input/output data schema for this task
- `workflow.json` — Execution manifest (task order, dependencies, triggers)
- `config.json` — Agent or server configuration
