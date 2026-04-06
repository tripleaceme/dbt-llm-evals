# How It Works

dbt-llm-evals uses an **LLM-as-a-Judge** pattern to evaluate AI outputs inside your warehouse. Here's the complete workflow.

## The Evaluation Pipeline

```
┌─────────────┐    ┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│  Your Model │───▶│   Capture   │───▶│   Evaluate  │───▶│   Monitor   │
│  (AI output)│    │  (post-hook)│    │  (AI judge) │    │  (alerts)   │
└─────────────┘    └─────────────┘    └─────────────┘    └─────────────┘
```

### Phase 1: Capture

When your dbt model runs, the `capture_and_evaluate()` post-hook fires. It:

1. Reads the `meta.llm_evals` config from your model
2. Checks if a baseline exists for the specified version
3. If no baseline: samples rows and stores them as the reference baseline
4. Captures a sample of inputs, outputs, and prompt data into `raw_captures`

The capture is a simple `INSERT INTO ... SELECT` that runs inside your warehouse — no data leaves.

### Phase 2: Evaluate

When you run `dbt run --select tag:llm_evals`, the evaluation models process pending captures:

1. **`llm_evals__captures`** — processes raw captures into a clean format
2. **`llm_evals__judge_evaluations`** — the core engine:
    - Cross-joins each capture with each evaluation criterion
    - Builds a comprehensive judge prompt with input, output, baseline examples, and scoring instructions
    - Calls the warehouse AI function (e.g., `AI_COMPLETE()` on Snowflake)
    - Parses the JSON response to extract score, reasoning, and confidence
3. **`llm_evals__eval_scores`** — flattens evaluations with capture data for easy querying

### Phase 3: Monitor

Monitoring models aggregate evaluation results:

- **`llm_evals__performance_summary`** — daily metrics per model and criterion
- **`llm_evals__drift_detection`** — statistical comparison of recent vs. historical scores
- **`llm_evals__alerts`** — consolidated alerts for drift, low pass rates, and parse errors

## The Judge Prompt

The judge receives a structured prompt containing:

```
=== ORIGINAL PROMPT ===
(The prompt template used to generate the output)

=== INPUT ===
(The input data as JSON)

=== OUTPUT ===
(The AI-generated output being evaluated)

=== BASELINE EXAMPLES ===
(Reference samples from the baseline for comparison)

=== EVALUATION TASK ===
Criterion: Accuracy
Description: Evaluate if the output is factually accurate...
Scale: 1-10 where 1=completely inaccurate, 10=perfectly accurate

=== RESPONSE FORMAT ===
{"score": <1-10>, "reasoning": "<explanation>", "confidence": <0.0-1.0>}
```

The judge responds with structured JSON that gets parsed and stored.

## Baseline Management

Baselines serve as quality benchmarks. The system:

- **Auto-creates** baselines on first run (no manual setup needed)
- **Versions** baselines (`v1.0`, `v2.0`, etc.) so you can track changes
- **Deactivates** old versions when new ones are created
- Supports **force rebaseline** when your AI model changes significantly

## Scoring Logic

Each evaluation produces a result category:

| Score | Result | Meaning |
|-------|--------|---------|
| >= pass threshold (default 7) | `pass` | Output meets quality standards |
| >= warn threshold (default 5) | `warn` | Output is borderline |
| < warn threshold | `fail` | Output needs attention |
| Parse error | `parse_error` | Judge response couldn't be parsed |

A `needs_review` flag is set when:

- Score is null (parse error)
- Confidence < 0.5
- Reasoning is missing or too short (< 10 chars)
