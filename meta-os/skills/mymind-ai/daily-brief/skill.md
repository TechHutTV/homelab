# /daily-brief — Morning Priority Digest

Generate a morning briefing across all MyMind vault sections, surfacing what matters today.

## Trigger
User runs `/daily-brief` or asks for their morning brief.

## Process

1. **Scan vault sections** for items tagged with today's date, "urgent", "due-today", or "follow-up"

2. **Generate brief** in this format:

```markdown
# Daily Brief — [DATE]

## Priority Actions
- [ ] [Highest priority items from 02-projects/active/]
- [ ] [Client follow-ups from 01-business/client-notes/]
- [ ] [Overdue items flagged]

## Today's Schedule Context
- [Pull from any calendar-linked notes]
- [Meeting prep: surface relevant client notes]

## Content Pipeline
- [Any content due for publish today from 01-business/content-ideas/]
- [Social posts scheduled via SocialBot]

## Ideas to Revisit
- [Surface 1-2 items from 00-inbox/ that have been sitting >3 days]
- [Random useful note from 04-reference/ for serendipity]

## Yesterday's Unfinished
- [Items from yesterday's brief still marked incomplete]
```

3. **Save brief** to `06-daily/briefs/YYYY-MM-DD-brief.md`

4. **Cross-reference** with ClickUp tasks if available (via MCP server)

## Output
- Displayed directly in terminal
- Saved to vault for reference
- Optionally posted to Lark #daily channel
