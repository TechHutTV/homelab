# MyMind AI — Second Brain Integration

**The cognitive layer of M3TA OS**

MyMind AI serves as Coach Metatron's personal second brain — the place where raw thoughts, client notes, creative references, competitive intel, and synthesized documents all converge into a searchable, AI-categorized knowledge graph.

## Why MyMind + M3TA OS

| Layer | Tool | Purpose |
|-------|------|---------|
| Team Knowledge | Notion | Shared wikis, client workspaces, SOPs |
| Content Knowledge | Kortex.ai | AI content library, trend data, research |
| Personal Knowledge | **MyMind AI** | Private cognitive workspace, second brain |
| Structured Data | Airtable | Rosters, inventories, CRM supplements |
| File Storage | Google Drive | Raw media, deliverables, contracts |

MyMind fills the gap between team-facing knowledge (Notion) and personal cognitive capture. It's where ideas are born before they become ClickUp tasks or Notion pages.

## Data Flow: Capture → Categorize → Action

```
Raw Input (thought, screenshot, PDF, URL, image)
  → MyMind AI auto-categorizes with tags
  → Claude Code searches/retrieves via skills
  → Surfaces in M3TA OS workflows as context
  → Becomes ClickUp task, Notion page, or content piece
```

## Integration with M3TA OS Workflows

### Content Pipeline
- Idea Harvest scrapes TikTok/Reddit → best finds saved to MyMind under `content_ideas/`
- Content Creation pulls reference material from MyMind during drafting
- SocialBot logs engagement insights back to MyMind for pattern recognition

### Media Production
- Shot references and mood boards stored in `media_library/`
- Pre-production pulls client visual preferences from MyMind
- Post-production retrospectives archived for future reference

### Client Onboarding
- MeetingScribe summaries auto-saved to `client_notes/`
- Contract cheat sheets (from DocuMind analysis) stored in `reference/contracts/`
- Client preferences and communication style notes captured

### Club Trapeze
- Competitor event research stored in `competitive_intel/`
- Fred's strategy session notes captured and searchable
- Event promotion ideas brainstormed and categorized

### Gold Star Flag Football
- Season planning notes and parent feedback
- Registration pattern insights for future campaign optimization
- Community partnership ideas and outreach contacts

## Claude Code Skills

Five purpose-built skills in `meta-os/skills/mymind-ai/`:

| Skill | Slash Command | Purpose |
|-------|--------------|---------|
| Vault Setup | `/vault-setup` | Interactive vault builder with multiple-choice questions |
| Daily Brief | `/daily-brief` | Morning digest of priorities across all vault sections |
| TLDDR | `/tlddr` | End-of-conversation summary → saved to MyMind |
| Canvas Gen | `/canvas-gen` | Visual knowledge maps and process diagrams |
| PDF Synthesizer | `/pdf-synthesize` | Large PDF → clean markdown cheat sheet → MyMind |

## PDF/Document Synthesis Pipeline

For large documents (annual reports, contracts, research papers):

```
1. Raw PDF/DOCX dropped into MyMind or local folder
2. Claude Code organizes by file type into subfolders
3. Gemini Flash (1M context) processes full document
4. Extracts salient points based on EEVL-specific criteria
5. Generates clean markdown cheat sheet
6. Cheat sheet saved to MyMind under appropriate category
7. Cross-referenced with related notes in the knowledge graph
```

## Setup

1. Install MyMind AI app (mymind.com)
2. Install Claude Code skills from `meta-os/skills/mymind-ai/`
3. Configure vault structure using `/vault-setup` skill
4. Add MyMind export path to CLAUDE.md for context injection
5. Start capturing — AI handles the rest
