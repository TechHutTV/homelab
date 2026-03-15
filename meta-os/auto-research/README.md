# Auto Research — M3TA OS Self-Improving Skill System

Inspired by [Andrej Karpathy's auto-research repo](https://github.com/karpathy/auto-research), adapted for Eagle Eye Vision Labz autonomous operations.

## Core Concept

Treat every skill prompt in M3TA OS like a `train.py` script:

- **`program.md`** = the agent instructions (the "training code")
- **`prompt.md`** (any skill/workflow prompt in the file tree) = the thing being optimized (the "model weights")
- **`evals/<skill>.json`** = the test suite (the "eval benchmark")

The system autonomously improves any prompt in the M3TA OS file tree by generating outputs, scoring them against binary criteria, mutating the prompt, and repeating until the target score is hit.

## Three Ingredients

Every auto-research loop requires exactly three things:

1. **Objective Metric** — Eval pass rate. A single number (0-100) that tells you how good the current prompt is. No vibes, no subjective judgment. Just: what percentage of binary criteria pass across N generations?

2. **Measurement Tool** — An automated test suite with binary (yes/no) criteria. An evaluator model reads each generated output and answers each criterion question with pass (1) or fail (0). No Likert scales. No "rate from 1-5." Binary only.

3. **Thing to Change** — The prompt/skill markdown file itself. The auto-research agent mutates the prompt instructions, adds constraints, rephrases sections, and tests whether the change improves the score.

## The Optimization Loop

```
┌─────────────────────────────────────────────────┐
│  1. Load current prompt.md                      │
│  2. Generate N outputs (default N=10)           │
│  3. Evaluate each output against binary criteria │
│  4. Score = passes / (N × criteria_count) × 100 │
│  5. Score >= target? → STOP, save winner         │
│  6. Score < target? → Analyze failures           │
│  7. Mutate prompt based on failure patterns      │
│  8. Go to step 2                                │
└─────────────────────────────────────────────────┘
```

Each iteration produces a scored run logged to `results/<skill-name>/run-<timestamp>.json`. The mutation history creates a complete changelog of what was tried and what worked.

## Key Principles

### BINARY Evals Only
No Likert scales. No "rate 1-5." When you use scales, you compound probability variance across criteria and runs. A criterion either passes or it doesn't. This gives you clean, composable signal.

### Don't Over-Constrain
If your eval criteria are too narrow or too specific, the optimization loop will "teach to the test." The prompt will game the criteria without producing genuinely useful output. Keep criteria focused on business-meaningful outcomes, not surface-level formatting.

### Run Many Times to Account for AI Variance
AI outputs are stochastic. A single generation tells you almost nothing. Run N=10+ times per iteration and look at:
- **Mode** — the most frequent score (what you'll usually get)
- **Median** — the middle score (robust to outliers)
- **Mean** — the average (useful but sensitive to outliers)

### Keep Criteria Independent
Each criterion should test one thing. Overlapping criteria double-count signal and create false confidence. If C1 and C2 both test "does it include a schedule," you're inflating your score without adding information.

## Why This Matters for EEVL

Every workflow prompt, every sub-agent prompt, every skill in M3TA OS can be autonomously optimized over time. Instead of manually tweaking prompts and hoping they improve:

- **Media production briefs** get better at including shot lists and equipment checklists
- **Client onboarding sequences** get faster and more complete
- **Content pipeline prompts** produce more SEO-optimized, platform-appropriate content
- **Social media bots** learn to respect character limits and optimize hashtags

The system improves itself while you sleep.

## Integration with M3TA OS

Auto-research operates on the M3TA OS file tree directly:

- **Target any prompt**: Point auto-research at any `prompt.md` anywhere in the file tree
- **Results stored centrally**: All run data lives in `auto-research/results/<skill-name>/`
- **Winning prompts written back**: When a prompt hits the target score, it's saved back to its original location
- **Scheduled rotation**: The orchestrator can rotate auto-research across all skills on a schedule (see `run-config.json`)

```
meta-os/
├── auto-research/
│   ├── program.md          ← Master agent instructions
│   ├── run-config.json     ← Configuration and skill rotation
│   ├── evals/              ← Binary eval suites per skill
│   ├── results/            ← Scored run logs per skill
│   └── skills/             ← Optimized skill snapshots
├── workflows/
│   └── */prompts/prompt.md ← Target prompts (optimized in-place)
└── sub-agents/
    └── */prompts/prompt.md ← Target prompts (optimized in-place)
```

## The Compound Value

Even if you can't reach 100% today, every auto-research run produces structured data:

- Which criteria fail most often
- Which mutations helped vs. hurt
- What score trajectories look like over time
- What failure patterns persist across model versions

This research data compounds. When future models (GPT-6, Opus 5, next-gen Sonnet) become available, you can re-run the same evals and immediately quantify improvement. The eval suite becomes a durable asset that outlasts any single model generation.

The prompts get better. The data accumulates. The system learns.
