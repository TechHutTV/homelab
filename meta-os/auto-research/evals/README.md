# Eval Suites — How to Write Binary Criteria

## Overview

Each eval suite is a JSON file named after the skill or workflow it tests. It lives in `auto-research/evals/` and contains an array of binary criteria that an evaluator agent uses to score generated outputs.

## File Structure

```json
{
  "skill_name": "my-skill-name",
  "target_prompt": "relative/path/to/prompt.md",
  "target_score": 90,
  "batch_size": 10,
  "max_iterations": 20,
  "eval_model": "claude-sonnet-4-6",
  "criteria": [
    {
      "id": "C1",
      "question": "Does the output include X?",
      "category": "completeness"
    },
    {
      "id": "C2",
      "question": "Is Y explicitly mentioned or specified?",
      "category": "specificity"
    }
  ]
}
```

### Fields

| Field | Type | Description |
|-------|------|-------------|
| `skill_name` | string | Identifier matching the JSON filename (without extension) |
| `target_prompt` | string | Relative path from M3TA OS root to the prompt being optimized |
| `target_score` | number | Score (0-100) the prompt must reach to "win" |
| `batch_size` | number | How many outputs to generate per iteration (default: 10) |
| `max_iterations` | number | Maximum optimization rounds before stopping |
| `eval_model` | string | Model used to evaluate outputs against criteria |
| `criteria` | array | List of binary criterion objects |

### Criterion Object

| Field | Type | Description |
|-------|------|-------------|
| `id` | string | Unique identifier (C1, C2, etc.) |
| `question` | string | Yes/no question the evaluator answers about the output |
| `category` | string | Category tag for grouping failure analysis |

## Writing Good Criteria

### DO: Keep Criteria Independent
Each criterion should test exactly one thing. If C1 and C3 both check for "timeline included," you're double-counting and inflating scores.

### DO: Use Binary Questions Only
Every criterion must be answerable with YES (pass=1) or NO (fail=0). No "rate from 1-5." No "how well does it..." Binary only. This eliminates probability variance that compounds across criteria and runs.

### DO: Aim for 5-8 Criteria Per Eval
- Fewer than 5: not enough signal to differentiate prompt quality
- More than 8: variance compounds, scores become noisy, and you risk over-constraining

### DO: Focus on Business-Meaningful Outcomes
Good: "Does the output include a shot list for each content pillar?"
Bad: "Does the output use bullet points for the shot list?"

The first tests whether the output is useful. The second tests formatting that could be gamed.

### DON'T: Use Scoring Scales
No Likert scales. No "1-5 ratings." No "partially meets criteria." When you introduce scales, evaluator models introduce their own variance on top of generation variance. Binary keeps the signal clean.

### DON'T: Over-Constrain
If your criteria are too narrow, the optimization loop will "teach to the test." The prompt will learn to parrot specific phrases that pass criteria without producing genuinely useful output. Keep criteria at the right altitude — specific enough to measure, general enough to allow creative solutions.

### DON'T: Overlap Criteria
If two criteria measure the same underlying thing, one of them is redundant. Merge or remove.

## Example

See any `.json` file in this directory for working examples:

- `media-production-brief.json` — Pre-production workflow eval
- `client-onboarding.json` — Lead capture and onboarding eval
- `content-pipeline.json` — Content creation workflow eval
- `social-bot.json` — Social media posting agent eval
- `club-trapeze-social.json` — Club Trapeze social management eval
