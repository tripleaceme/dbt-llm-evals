# Configuration

dbt-llm-evals is configured through dbt variables and model-level meta config.

## Global Variables

Set these in your `dbt_project.yml`:

### Required

| Variable | Description | Example |
|----------|-------------|---------|
| `llm_evals_judge_model` | AI model used as the judge | `'llama3-70b'` |

### Evaluation Settings

| Variable | Default | Description |
|----------|---------|-------------|
| `llm_evals_criteria` | `'["accuracy", "relevance"]'` | JSON array of evaluation criteria |
| `llm_evals_sampling_rate` | `0.1` | Fraction of outputs to capture (0.0-1.0) |
| `llm_evals_batch_size` | `1000` | Max captures to evaluate per run |
| `llm_evals_pass_threshold` | `7` | Score >= this is a "pass" |
| `llm_evals_warn_threshold` | `5` | Score >= this (but < pass) is a "warning" |

### Baseline & Drift

| Variable | Default | Description |
|----------|---------|-------------|
| `llm_evals_baseline_sample_size` | `100` | Number of samples in each baseline |
| `llm_evals_baseline_refresh_days` | `30` | Days before baseline refresh suggestion |
| `llm_evals_drift_stddev_threshold` | `2` | Standard deviations for drift alert |
| `llm_evals_drift_lookback_days` | `7` | Days to look back for drift detection |

### AI Model Parameters

| Variable | Default | Description |
|----------|---------|-------------|
| `llm_evals_judge_temperature` | `0.0` | Temperature for judge model (0.0 = deterministic) |
| `llm_evals_judge_max_tokens` | `500` | Max tokens for judge response |

### BigQuery-Specific

| Variable | Description |
|----------|-------------|
| `gcp_project_id` | Google Cloud project ID |
| `gcp_location` | GCP region (e.g., `us-central1`) |
| `llm_evals_dataset` | BigQuery dataset for AI models |
| `ai_connection_id` | Vertex AI remote connection ID |

## Model-Level Configuration

Configure each AI model via the `meta.llm_evals` block:

```yaml
models:
  - name: your_ai_model
    config:
      post_hook: "{{ dbt_llm_evals.capture_and_evaluate() }}"
      meta:
        llm_evals:
          enabled: true
          input_columns:
            - question
            - context
          output_column: 'ai_response'
          prompt: >-
            Context: {context}
            Question: {question}
            Answer:
          sampling_rate: 0.2
          baseline_version: 'v1.0'
          force_rebaseline: false
```

### Model Meta Fields

| Field | Required | Default | Description |
|-------|----------|---------|-------------|
| `enabled` | Yes | `false` | Enable evaluation for this model |
| `input_columns` | Yes | — | List of input column names |
| `output_column` | Yes | — | Column containing AI output |
| `prompt` | No | `null` | Prompt template (for evaluation context) |
| `sampling_rate` | No | Global setting | Override global sampling rate |
| `baseline_version` | No | `'v1.0'` | Baseline version identifier |
| `force_rebaseline` | No | `false` | Force creation of new baseline |

## Example: Full Configuration

```yaml
# dbt_project.yml
vars:
  llm_evals_judge_model: 'llama3-70b'
  llm_evals_criteria: '["accuracy", "relevance", "tone", "completeness"]'
  llm_evals_sampling_rate: 0.1
  llm_evals_pass_threshold: 7
  llm_evals_warn_threshold: 5
  llm_evals_baseline_sample_size: 100
  llm_evals_judge_temperature: 0.0
  llm_evals_judge_max_tokens: 500
```

## Evaluation Criteria

The `llm_evals_criteria` variable accepts a JSON array. Built-in criteria:

- `accuracy` — factual correctness
- `relevance` — addresses the input appropriately
- `tone` — professional appropriateness
- `completeness` — fully addresses all aspects
- `consistency` — matches baseline quality/style
- `helpfulness` — actionable and useful
- `clarity` — clear and well-structured

You can use any combination:

```yaml
vars:
  llm_evals_criteria: '["accuracy", "relevance", "tone"]'
```

Custom criterion names are also supported — the judge will evaluate based on the criterion name with a generic prompt.
