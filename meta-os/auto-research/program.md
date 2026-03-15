# Auto Research — M3TA OS Skill Optimizer

## Objective

You are an autonomous optimization agent. Your job is to iteratively improve a target skill prompt by running it multiple times, evaluating outputs against a binary eval suite, scoring results, mutating the prompt to improve scores, and repeating until the target score is achieved.

You operate in a closed loop. No human intervention is required between iterations. You read the skill, run it, score it, fix it, and repeat.

## Inputs

Before starting, confirm you have access to:

- **Target skill prompt**: `skills/<skill-name>/prompt.md`
- **Eval suite**: `evals/<skill-name>.json`
- **Run config**: `run-config.json` (or override values provided at invocation)

## Process

### Step 1 — Load

1. Read the target skill from `skills/<skill-name>/prompt.md`
2. Read the eval suite from `evals/<skill-name>.json`
3. Read `run-config.json` for default parameters
4. Initialize iteration counter at 0
5. Create results directory at `results/<skill-name>/` if it does not exist

### Step 2 — Generate

For each iteration:

1. Using the current version of the skill prompt, generate **N outputs** (default N=10, configurable via `batch_size` in the eval or `default_batch_size` in run-config)
2. Each output should be generated independently — do not let outputs influence each other
3. Store all N raw outputs in memory for evaluation

### Step 3 — Evaluate

For each of the N outputs, evaluate against **every criterion** in the eval suite:

1. Read the criterion question (e.g., "Does the shot list include all three pillars?")
2. Answer strictly **yes** (pass=1) or **no** (fail=0) — no partial credit, no "mostly", no "partially"
3. Record the result per output per criterion

### Step 4 — Score

Calculate the aggregate score for this iteration:

```
score = (total_passes / (N * criteria_count)) * 100
```

Also calculate:
- **Per-criterion pass rate**: how often each criterion passed across all N outputs
- **Per-output score**: how many criteria each individual output passed
- **Median output score**: the middle output score when sorted
- **Mode output score**: the most common output score
- **Min/Max spread**: the range between worst and best individual outputs

### Step 5 — Log

Write results to `results/<skill-name>/run-<timestamp>.json` with the following structure:

```json
{
  "skill_name": "<skill-name>",
  "iteration": 1,
  "timestamp": "2026-03-15T10:30:00Z",
  "prompt_version": "<hash or version number>",
  "prompt_snapshot": "<full text of prompt used>",
  "batch_size": 10,
  "criteria_count": 6,
  "max_possible": 60,
  "total_passes": 48,
  "score": 80.0,
  "per_criterion": [
    { "criteria_id": "shot-list-pillars", "pass_rate": 0.9 },
    { "criteria_id": "equipment-models", "pass_rate": 0.7 }
  ],
  "per_output_scores": [5, 6, 4, 6, 5, 5, 6, 4, 5, 6],
  "median_score": 5,
  "mode_score": 5,
  "min_score": 4,
  "max_score": 6,
  "mutation_applied": null,
  "mutation_rationale": null
}
```

### Step 6 — Decide

- **If score >= target_score**: STOP. Save the current prompt as the winning version. Write a summary to `results/<skill-name>/winner.json`.
- **If score < target_score AND iteration < max_iterations**: proceed to mutation (Step 7).
- **If score < target_score AND iteration >= max_iterations**: STOP. Save the best-scoring prompt version seen across all iterations. Write a summary noting the target was not reached.

### Step 7 — Mutate

Analyze failure patterns and modify the prompt:

1. **Identify weakest criteria**: sort criteria by pass rate, focus on the lowest
2. **Analyze failure outputs**: read the outputs that failed the weakest criteria — what went wrong?
3. **Hypothesize a fix**: formulate a specific change to the prompt that should address the failure
4. **Apply one mutation**: make a single, targeted change to the prompt
5. **Document the mutation**: record what was changed and why

Return to Step 2 with the mutated prompt.

## Mutation Strategy

### What to Do

- Add explicit instructions addressing the most common failure pattern
- Rephrase ambiguous instructions that are being misinterpreted
- Add examples or clarifications for criteria that fail intermittently
- Reorder instructions to put critical requirements earlier in the prompt
- Add a checklist section at the end of the prompt for must-have items

### What Not to Do

- Never remove a constraint that was consistently passing — only add or modify
- Never make more than one substantive change per iteration (isolate variables)
- Never make the prompt longer than `max_prompt_length_chars` (default: 5000)
- Never add instructions that directly quote eval criteria verbatim (anti-gaming)
- Never optimize for a single criterion at the expense of overall quality

### Handling Plateaus

If the score has not improved for 3 consecutive iterations:

1. Try a fundamentally different prompt structure (rewrite, not patch)
2. Consider whether the eval criteria themselves are contradictory
3. Log a plateau warning in the results
4. If plateau persists for 5 more iterations, stop and report

## Scoring Rules

- All evals are **BINARY** (pass=1, fail=0) — no partial credit
- Score = (total_passes / max_possible) * 100
- A "run" = N generations evaluated against all criteria
- Keep running until score >= target or max_iterations reached
- The **winning prompt** is the one with the highest score across all iterations, not necessarily the last one

## Anti-Gaming Rules

Watch for these failure modes and flag them if detected:

1. **Teaching to the test**: The output starts parroting eval criteria verbatim (e.g., a shot list that says "this includes all three pillars: video, drone, photo" as a literal sentence rather than actually including them naturally)
2. **Checkbox stuffing**: The output includes a laundry list of items that technically satisfy criteria but provide no real value
3. **Over-constrained outputs**: The prompt becomes so specific that outputs lose flexibility and sound robotic
4. **Quality floor violation**: Outputs that pass all criteria but would not actually serve their business purpose if used by a human

If any of these are detected:
- Log the gaming pattern in the results
- Roll back to the last non-gaming prompt version
- Add an anti-gaming instruction to the prompt
- Continue optimization from the rolled-back version

## Completion

When optimization is complete (either target reached or max iterations exhausted):

1. Save the best prompt to `skills/<skill-name>/prompt.md` (if `auto_save_winner` is true)
2. Generate a changelog at `results/<skill-name>/changelog.md` documenting:
   - Starting score
   - Each mutation attempted and its effect on the score
   - Final score
   - Total iterations run
   - Time elapsed
3. Output a summary to the console with the final score and key statistics
