# /tlddr — Conversation Summary → Second Brain

Capture the essence of any Claude Code conversation and save it to MyMind vault.

## Trigger
User runs `/tlddr` at the end of a conversation or work session.

## Process

1. **Analyze current conversation** — scan the full session for:
   - Decisions made
   - Code changes implemented
   - Problems solved
   - Questions still open
   - Next steps identified

2. **Generate TLDDR** in this format:

```markdown
---
date: [TODAY]
tags: [auto-generated from conversation topics]
status: captured
source: claude-code-session
related: [linked vault notes if applicable]
---

# TLDDR — [Auto-generated title from conversation topic]

## Summary
[2-3 sentence summary of what was accomplished]

## Key Decisions
- [Decision 1 and rationale]
- [Decision 2 and rationale]

## What Changed
- [File/code changes made]
- [Configurations updated]
- [New things created]

## Open Questions
- [Unresolved items that need follow-up]

## Next Steps
- [ ] [Action item 1]
- [ ] [Action item 2]
- [ ] [Action item 3]

## Raw Context
[Key code snippets, URLs, or references worth keeping]
```

3. **Auto-categorize** — determine the best vault location:
   - If about a client → `01-business/client-notes/<client>/`
   - If about a project → `02-projects/active/<project>/`
   - If about content → `01-business/content-ideas/`
   - If about infrastructure → `04-reference/tech-stack/`
   - If unclear → `00-inbox/`

4. **Save** to `06-daily/tlddr/YYYY-MM-DD-[topic-slug].md` AND the categorized location

5. **Link** — add backlinks to related existing notes in the vault

## Output
- Summary displayed in terminal
- Saved to vault in appropriate location
- Confirmation with file path shown
