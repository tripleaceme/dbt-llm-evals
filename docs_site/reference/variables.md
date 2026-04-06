# Variables Reference

All variables are set in your `dbt_project.yml` under the `vars:` key.

## Required Variables

| Variable | Type | Description |
|----------|------|-------------|
| `llm_evals_judge_model` | string | The AI model to use as judge. Must be available in your warehouse. |

## Evaluation Settings

| Variable | Type | Default | Description |
|----------|------|---------|-------------|
| `llm_evals_criteria` | string (JSON) | `'["accuracy", "relevance"]'` | JSON array of criteria to evaluate |
| `llm_evals_sampling_rate` | float | `0.1` | Global sampling rate (0.0 to 1.0) |
| `llm_evals_batch_size` | integer | `1000` | Max captures to evaluate per dbt run |
| `llm_evals_pass_threshold` | integer | `7` | Score >= this is "pass" |
| `llm_evals_warn_threshold` | integer | `5` | Score >= this (but < pass) is "warn" |

## Baseline & Drift Settings

| Variable | Type | Default | Description |
|----------|------|---------|-------------|
| `llm_evals_baseline_sample_size` | integer | `100` | Samples per baseline version |
| `llm_evals_baseline_refresh_days` | integer | `30` | Days before suggesting baseline refresh |
| `llm_evals_drift_stddev_threshold` | integer | `2` | Std deviations for drift alert |
| `llm_evals_drift_lookback_days` | integer | `7` | Days to look back for drift detection |

## AI Model Parameters

| Variable | Type | Default | Description |
|----------|------|---------|-------------|
| `llm_evals_judge_temperature` | float | `0.0` | Judge model temperature (0.0 = deterministic) |
| `llm_evals_judge_max_tokens` | integer | `500` | Max tokens for judge response |

## Storage Configuration

| Variable | Type | Default | Description |
|----------|------|---------|-------------|
| `llm_evals_schema` | string | Target schema | Custom schema for raw tables |

## BigQuery-Specific Variables

| Variable | Type | Description |
|----------|------|-------------|
| `gcp_project_id` | string | Google Cloud project ID |
| `gcp_location` | string | GCP region (e.g., `us-central1`) |
| `llm_evals_dataset` | string | BigQuery dataset containing AI models |
| `ai_connection_id` | string | Full path to Vertex AI remote connection |

## Example: Minimal Configuration

```yaml
vars:
  llm_evals_judge_model: 'llama3-70b'
```

## Example: Full Configuration

```yaml
vars:
  # Judge
  llm_evals_judge_model: 'llama3-70b'
  llm_evals_judge_temperature: 0.0
  llm_evals_judge_max_tokens: 500

  # Evaluation
  llm_evals_criteria: '["accuracy", "relevance", "tone", "completeness"]'
  llm_evals_sampling_rate: 0.1
  llm_evals_batch_size: 500
  llm_evals_pass_threshold: 7
  llm_evals_warn_threshold: 5

  # Baseline & Drift
  llm_evals_baseline_sample_size: 100
  llm_evals_baseline_refresh_days: 30
  llm_evals_drift_stddev_threshold: 2
  llm_evals_drift_lookback_days: 7

  # Storage
  llm_evals_schema: 'llm_evals'
```
