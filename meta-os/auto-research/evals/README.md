# Writing Eval Suites for Auto Research

## Overview

Each skill in the M3TA OS auto-research system has a corresponding eval suite — a JSON file in this directory named after the skill (e.g., `media-production-brief.json`). The eval suite defines the binary criteria used to score generated outputs during optimization.

## File Naming

```
evals/<skill-name>.json
```

The skill name must match the directory name in `skills/`. For example:
- Skill prompt: `skills/media-production-brief/prompt.md`
- Eval suite: `evals/media-production-brief.json`

## Structure

Every eval JSON file follows this schema:

```json
{
  "skill_name": "example-skill",
  "description": "What this eval measures",
  "target_score": 90,
  "batch_size": 10,
  "max_iterations": 20,
  "criteria": [
    {
      "criteria_id": "unique-kebab-case-id",
      "question": "Does the output include X? (yes/no)",
      "weight": 1,
      "category": "completeness"
    }
  ]
}
```

### Field Reference

| Field | Type | Required | Description |
|---|---|---|---|
| `skill_name` | string | yes | Must match the skill directory name |
| `description` | string | yes | Human-readable summary of what this eval checks |
| `target_score` | number | yes | Score (0-100) at which optimization stops |
| `batch_size` | number | no | Outputs per iteration (overrides run-config default) |
| `max_iterations` | number | no | Max optimization loops (overrides run-config default) |
| `criteria` | array | yes | List of binary eval criteria |
| `criteria[].criteria_id` | string | yes | Unique identifier, kebab-case |
| `criteria[].question` | string | yes | Binary yes/no question about the output |
| `criteria[].weight` | number | yes | Multiplier for this criterion (default: 1) |
| `criteria[].category` | string | yes | Grouping label for analysis |

## Writing Good Criteria

### The Golden Rule: Binary Only

Every criterion must be answerable with a strict **yes** or **no**. There is no "partially" or "somewhat". If you find yourself wanting to say "it kind of does", the criterion is too vague — rewrite it.

**Good criteria:**
- "Does the output include a shot list? (yes/no)"
- "Is the client name referenced at least once? (yes/no)"
- "Are file naming conventions specified? (yes/no)"

**Bad criteria:**
- "Is the shot list comprehensive?" (subjective — what counts as comprehensive?)
- "How well does the output handle scheduling?" (not binary)
- "Rate the quality of the equipment checklist" (Likert scale, not binary)

### Keep Criteria Independent

Each criterion should test one thing. If criterion A passing depends on criterion B passing, they are coupled and should be restructured.

**Bad (coupled):**
1. "Does the output include a schedule?"
2. "Does the schedule include setup time?"

Criterion 2 implicitly requires criterion 1. If there is no schedule at all, criterion 2 is not really testable.

**Better:**
1. "Does the output include a schedule?"
2. "Does the output include time allocated for setup and teardown (either in a schedule or as a separate note)?"

### Don't Overlap

If two criteria can be satisfied by the exact same piece of content, one of them is redundant. Each criterion should target a distinct aspect of the output.

### Avoid Overly Narrow Constraints

Criteria that are too specific force the optimization agent to game them. The model will learn to insert the exact phrasing rather than genuinely improving.

**Too narrow:**
- "Does the output contain the exact phrase 'Brand Guidelines v3.2'?"

**Better:**
- "Does the output reference brand guidelines? (yes/no)"

### Recommended Category Labels

Use consistent categories across eval suites for cross-skill analysis:

| Category | What it Covers |
|---|---|
| `completeness` | Are all required sections/items present? |
| `accuracy` | Are facts, names, and references correct? |
| `format` | Does the output follow structural requirements? |
| `integration` | Does the output connect to other systems (Lark, ClickUp, etc.)? |
| `timing` | Are time-based requirements met? |
| `brand` | Does the output maintain brand standards? |
| `platform` | Are platform-specific requirements met? |

## Scoring Math

The score for a single run is:

```
score = (total_weighted_passes / (N * sum_of_weights)) * 100
```

Where:
- `total_weighted_passes` = sum of (pass * weight) across all outputs and all criteria
- `N` = batch size (number of outputs generated)
- `sum_of_weights` = sum of all criterion weights

With all weights set to 1, this simplifies to:

```
score = (total_passes / (N * criteria_count)) * 100
```

## Tips for Setting Target Scores

- **90-95**: Appropriate for critical workflows where every output must be nearly perfect (client-facing, financial, legal)
- **85-89**: Good for creative workflows where some variation is acceptable
- **80-84**: Suitable for internal-only outputs or early-stage optimization
- **Below 80**: Likely indicates the prompt needs a fundamental rewrite, not just tuning

## Tips for Setting Batch Size

- **N=10**: Good default for most skills. Enough to see patterns without excessive cost.
- **N=5**: Acceptable for quick iteration during initial prompt development.
- **N=20+**: Use for final validation of a prompt you believe is ready for production.

## Tips for Setting Max Iterations

- **15-20**: Standard range. Most prompts converge within 10 iterations.
- **25+**: Use for complex skills with many criteria or when starting from a low baseline.
- **Under 10**: Only for minor tweaks to an already-good prompt.
