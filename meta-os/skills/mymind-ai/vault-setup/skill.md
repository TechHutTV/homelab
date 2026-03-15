# /vault-setup — MyMind AI Second Brain Builder

Build a personalized MyMind AI vault structure through guided multiple-choice questions.

## Trigger
User runs `/vault-setup` or asks to set up their second brain.

## Process

### Step 1: Discovery (Use AskUserQuestion tool with multiple-choice)

Ask these 4 questions using the AskUserQuestion tool with multiple-choice options:

**Q1: "What do you do for work?"**
- Business Owner / Entrepreneur
- Creative Professional (Photo/Video/Design)
- Agency / Client Services
- Tech / Engineering
- Other (free text)

**Q2: "What falls through the cracks most? What do you wish you tracked better?"**
- Projects and deadlines
- Ideas and creative inspiration
- Client communication and follow-ups
- Financial decisions and invoices
- Meeting notes and action items

**Q3: "Should this be work-only or full life OS?"**
- Work only
- Work + Personal life
- Full Life OS (work, personal, health, goals, learning)

**Q4: "Do you have existing files to import?"**
- Yes — PDFs, docs, spreadsheets to synthesize
- Yes — existing notes from Notion/Evernote/Apple Notes
- No — starting fresh
- Both — existing files AND existing notes

### Step 2: Vault Architecture

Based on answers, generate a vault structure. Default EEVL structure:

```
MyMind Vault/
├── 00-inbox/                    # AI auto-sorts from here
├── 01-business/
│   ├── eevl-operations/
│   ├── client-notes/
│   │   ├── club-trapeze/
│   │   ├── gold-star/
│   │   └── _template.md
│   ├── content-ideas/
│   ├── competitive-intel/
│   └── revenue-notes/
├── 02-projects/
│   ├── active/
│   ├── pipeline/
│   └── archive/
├── 03-media/
│   ├── shot-references/
│   ├── mood-boards/
│   ├── inspiration/
│   └── edit-notes/
├── 04-reference/
│   ├── sops/
│   ├── contracts/
│   ├── brand-guidelines/
│   ├── tech-stack/
│   └── cheat-sheets/
├── 05-personal/
│   ├── goals/
│   ├── learning/
│   ├── ideas/
│   └── health/
└── 06-daily/
    ├── briefs/
    ├── standups/
    └── tlddr/
```

### Step 3: Skill Installation

Offer to install companion skills:
- `/daily-brief` — Morning priority digest
- `/tlddr` — End-of-conversation summary capture
- `/canvas-gen` — Visual knowledge maps
- `/pdf-synthesize` — Large document → cheat sheet pipeline

### Step 4: Context Injection Setup

Add to the user's CLAUDE.md:
```markdown
## Second Brain (MyMind AI)
Always reference ~/MyMind/ (or configured path) when:
- Brainstorming content ideas → check 01-business/content-ideas/
- Preparing for client work → check 01-business/client-notes/<client>/
- Starting a new project → check 02-projects/ for related past work
- Looking for creative references → check 03-media/
```

### Step 5: Build

Create the folder structure, template files, and initial markdown files for each section. Each template includes:
- YAML frontmatter (date, tags, status, related)
- Section headers appropriate to the folder type
- Placeholder prompts to guide first entries
