# Auto Research — Self-Improving Skill System for M3TA OS

## Inspiration

This system is directly inspired by [Andrej Karpathy's auto-research repo](https://github.com/karpathy/auto-research), which distills autonomous AI research into just three files:

| Karpathy's Repo | M3TA OS Equivalent | Purpose |
|---|---|---|
| `train.py` | `skills/<name>/prompt.md` | The thing being optimized — the skill prompt |
| `program.md` | `program.md` | Agent instructions that drive the optimization loop |
| `prepare.py` | `evals/<name>.json` | The evaluation criteria that define "good" |

The core insight: if you have (1) an objective metric, (2) a tool to measure it, and (3) something you can change, you can set up an autonomous loop that improves the thing until it hits a target score.

## The Three Ingredients

### 1. Objective Metric — Eval Pass Rate

Every skill is evaluated against a set of **binary criteria** (yes/no questions). The score is simply:

```
score = (total_passes / (N * criteria_count)) * 100
```

Where `N` is the number of outputs generated per run. Binary evals eliminate subjectivity — there is no Likert scale, no "rate from 1-5", no partial credit. Either the output includes the thing or it does not.

### 2. Measurement Tool — Automated Eval Suite

Each skill has a corresponding eval file (`evals/<skill-name>.json`) containing binary questions. An evaluator model reads each generated output and answers each question with a strict yes or no. No human in the loop during optimization.

### 3. Something to Change — The Skill Prompt

The prompt markdown file is the "weights" of our system. The optimization agent reads failure patterns, hypothesizes what instruction changes would fix them, mutates the prompt, and tests again.

## The Optimization Loop

```
┌─────────────────────────────────────────────────┐
│  1. Load current prompt from skills/<name>/     │
│  2. Generate N outputs (default N=10)           │
│  3. Evaluate each output against all criteria   │
│  4. Score: passes / (N * criteria_count) * 100  │
│  5. Score >= target? ──► YES ──► Save winner    │
│         │                                        │
│         NO                                       │
│         │                                        │
│  6. Analyze failure patterns                     │
│  7. Mutate prompt (one change at a time)        │
│  8. Log results + mutation to changelog         │
│  9. Go to step 2                                │
└─────────────────────────────────────────────────┘
```

Each iteration produces a scored result logged to `results/<skill-name>/run-<timestamp>.json`. The system keeps running until the target score is met or `max_iterations` is exhausted.

## Key Principles

### Binary Evals Only

**Never use Likert scales.** "Rate the quality from 1-5" is meaningless when an LLM is the evaluator. Instead, ask questions with unambiguous yes/no answers:

- BAD: "How good is the shot list? (1-5)"
- GOOD: "Does the shot list include all three pillars (video, drone, photo)? (yes/no)"

Binary evals are reproducible, composable, and honest.

### Don't Over-Constrain

If your eval has 30 ultra-specific criteria, the optimization agent will "teach to the test" — producing outputs that technically pass every check but read like a compliance checklist. Keep criteria focused on **outcomes that matter** for the business purpose of the skill.

### Run Many Times to Account for Variance

LLM outputs are stochastic. A single generation tells you almost nothing. Run N=10 (or more) and look at:

- **Mode**: the most common score (tells you what "usually" happens)
- **Median**: the middle score (robust to outliers)
- **Min/Max spread**: how volatile the prompt is

A prompt that scores 95% once and 40% the next time is worse than one that reliably scores 80%.

### Compound Probability Warning

If you have 6 binary criteria and each independently passes 90% of the time, the probability of ALL six passing on a single generation is:

```
0.9^6 = 0.53 (53%)
```

This means even a "90% good" prompt will produce a fully-passing output only about half the time. This is why batch evaluation matters — you need the aggregate score, not individual perfection.

## Why This Matters for EEVL

Every workflow prompt in the M3TA OS stack is a candidate for auto-research optimization:

- **Client onboarding flows** — ensure every step is covered every time
- **Media production briefs** — guarantee shot lists, equipment, and schedules are complete
- **Content pipelines** — maintain SEO, brand, and platform standards automatically
- **SocialBot prompts** — optimize for platform-specific engagement patterns
- **Sub-agent prompts** — any prompt that drives an EEVL agent can be scored and improved

The end state: a system where every prompt in the organization is continuously tested and improved, with full traceability of what changed, why, and what score it achieved.

## Directory Structure

```
auto-research/
├── README.md              # This file
├── program.md             # Master agent instructions
├── run-config.json        # Default configuration
├── evals/                 # Eval suites (one JSON per skill)
│   ├── README.md          # How to write eval suites
│   ├── media-production-brief.json
│   ├── client-onboarding.json
│   ├── content-pipeline.json
│   ├── social-bot.json
│   └── club-trapeze-social.json
├── results/               # Scored run logs (auto-generated)
│   └── .gitkeep
└── skills/                # Skill prompts to optimize
    └── .gitkeep
```

## Getting Started

1. Place a skill prompt in `skills/<skill-name>/prompt.md`
2. Create an eval suite in `evals/<skill-name>.json`
3. Run the optimization agent with `program.md` as instructions
4. Monitor results in `results/<skill-name>/`
5. The winning prompt is saved back to the skill location automatically
