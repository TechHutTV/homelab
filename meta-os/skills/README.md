# M3TA OS Skills

Claude Code skills that extend M3TA OS capabilities. Skills are slash-command-driven workflows that agents can execute.

## MyMind AI — Second Brain Skills

| Skill | Command | Purpose |
|-------|---------|---------|
| [Vault Setup](mymind-ai/vault-setup/) | `/vault-setup` | Interactive vault builder with guided multiple-choice questions |
| [Daily Brief](mymind-ai/daily-brief/) | `/daily-brief` | Morning priority digest across all vault sections |
| [TLDDR](mymind-ai/tlddr/) | `/tlddr` | End-of-conversation summary capture → saved to vault |
| [Canvas Gen](mymind-ai/canvas-gen/) | `/canvas-gen` | Visual knowledge maps, process diagrams, architecture views |
| [PDF Synthesizer](mymind-ai/pdf-synthesizer/) | `/pdf-synthesize` | Large document → clean markdown cheat sheet → vault |

## How Skills Connect to M3TA OS

Skills are the human-facing interface to M3TA OS workflows. While workflows run autonomously via triggers and sub-agents, skills let Coach Metatron manually invoke specific capabilities:

```
User runs /tlddr
  → Skill analyzes conversation
  → Generates structured summary
  → Auto-categorizes into MyMind vault
  → Cross-references with existing notes
  → Saves to 06-daily/tlddr/ AND categorized location
```

## Adding New Skills

1. Create a folder under `skills/<category>/<skill-name>/`
2. Add `skill.md` with trigger, process, and output format
3. Optionally add an eval suite in `auto-research/evals/` for Auto Research optimization
4. Reference in CLAUDE.md for context injection
