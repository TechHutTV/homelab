# /pdf-synthesize — Large Document → Cheat Sheet → Second Brain

Transform large PDFs, annual reports, contracts, and research papers into clean markdown cheat sheets stored in MyMind vault.

## Trigger
User runs `/pdf-synthesize [file-path]` or asks to process/summarize a document.

## Process

### Step 1: Organize
- Detect file type (PDF, DOCX, XLSX, CSV, TXT)
- Create organized subfolder if processing multiple files:
  ```
  _processing/
  ├── pdfs/
  ├── docs/
  ├── spreadsheets/
  └── other/
  ```

### Step 2: Extract
- For PDFs: Extract text content (use Claude's native PDF reading)
- For large documents (>50 pages): chunk into sections by headers/chapters
- Preserve tables, lists, and structured data

### Step 3: Synthesize
- Process through LLM with EEVL-specific synthesis prompt:

```
You are synthesizing this document for Eagle Eye Vision Labz.
Extract ONLY the signal — skip boilerplate, legalese filler, and redundant sections.

Focus on:
- Key terms, dates, and dollar amounts
- Action items and obligations
- Deadlines and renewal dates
- Risk flags (liability, exclusivity, auto-renewal)
- Strategic insights relevant to media production, content, or client services
- Competitive intelligence
- Anything Coach Metatron would need to reference later

Output as a clean markdown cheat sheet with:
- YAML frontmatter (source, date_processed, document_type, tags)
- Executive summary (3-5 sentences max)
- Key facts (bullet list)
- Action items (checkbox list)
- Important quotes/clauses (blockquotes)
- Related context (links to other vault notes if applicable)
```

### Step 4: Categorize and Save
- Auto-detect category from content:
  - Contract → `04-reference/contracts/`
  - Financial report → `01-business/revenue-notes/`
  - Research/article → `04-reference/cheat-sheets/`
  - Client document → `01-business/client-notes/<client>/`
  - SOP/how-to → `04-reference/sops/`
  - Unknown → `00-inbox/`

### Step 5: Cross-Reference
- Scan existing vault notes for related content
- Add backlinks to the new cheat sheet
- Update any existing notes that reference the source document

## Output Format

```markdown
---
source: "[Original filename]"
date_processed: [TODAY]
document_type: [contract|report|research|sop|other]
pages: [X]
tags: [auto-generated]
status: synthesized
---

# [Document Title] — Cheat Sheet

## Executive Summary
[3-5 sentence distillation]

## Key Facts
- [Fact 1]
- [Fact 2]
- [Fact 3]

## Action Items
- [ ] [Action 1 — due date if applicable]
- [ ] [Action 2]

## Important Clauses / Quotes
> [Notable quote or clause]

## Risk Flags
- [Any risks, auto-renewals, liability concerns]

## Related Notes
- [[linked-note-1]]
- [[linked-note-2]]
```

## Batch Processing
When given a folder of files:
1. Organize by type
2. Process each sequentially
3. Generate index file listing all cheat sheets created
4. Report summary: X files processed, Y cheat sheets created, Z categorized

## Model Selection
- Documents <20 pages: Claude Sonnet (fast, cheap)
- Documents 20-100 pages: Claude Opus (deep analysis)
- Documents >100 pages: Gemini Flash 1M context (handles full document)
