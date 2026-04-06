# Models Reference

## Core Models

### llm_evals__setup

**Materialization**: View | **Tags**: `llm_evals`, `core`

One-time setup model that creates `raw_captures` and `raw_baselines` storage tables. Run once before using the package.

```bash
dbt run --select llm_evals__setup
```

### llm_evals__captures

**Materialization**: Incremental | **Tags**: `llm_evals`, `core`

Processes raw captures into a clean, queryable format.

| Column | Type | Description |
|--------|------|-------------|
| `capture_id` | string | Unique capture identifier |
| `source_model` | string | dbt model that generated the output |
| `input_data` | variant/string | Input columns as JSON |
| `output_data` | string | AI-generated output |
| `prompt_data` | string | Prompt template used |
| `captured_at` | timestamp | When the output was captured |
| `dbt_invocation_id` | string | dbt run identifier |
| `eval_status` | string | `pending` or `completed` |
| `evaluated_at` | timestamp | When evaluation completed |

### llm_evals__baselines

**Materialization**: Incremental | **Tags**: `llm_evals`, `core`

Baseline samples used for consistency evaluation.

| Column | Type | Description |
|--------|------|-------------|
| `baseline_id` | string | Unique baseline sample identifier |
| `source_model` | string | dbt model this baseline is for |
| `baseline_version` | string | Version tag (e.g., `v1.0`) |
| `baseline_input` | variant/string | Sample input |
| `baseline_output` | string | Sample output |
| `is_active` | boolean | Whether this version is active |
| `baseline_created_at` | timestamp | When created |

### llm_evals__registry

**Materialization**: Incremental | **Tags**: `llm_evals`, `core`

Registry summarizing all models being evaluated.

| Column | Type | Description |
|--------|------|-------------|
| `source_model` | string | dbt model being evaluated |
| `baseline_sample_count` | integer | Number of baseline samples |
| `has_active_baseline` | boolean | Whether an active baseline exists |
| `total_captures` | integer | Total captures |
| `evaluated_count` | integer | Evaluated captures |
| `pending_count` | integer | Pending evaluations |
| `status` | string | `active`, `baseline_only`, `no_baseline`, `inactive` |

---

## Evaluation Models

### llm_evals__judge_evaluations

**Materialization**: Incremental | **Tags**: `llm_evals`, `evaluation`

The core evaluation engine. Processes pending captures by calling the warehouse AI judge.

| Column | Type | Description |
|--------|------|-------------|
| `eval_id` | string | Unique evaluation identifier |
| `capture_id` | string | Reference to the capture |
| `source_model` | string | dbt model that generated the output |
| `criterion` | string | Evaluation criterion (e.g., `accuracy`) |
| `judge_model` | string | AI model used as judge |
| `score` | integer | Evaluation score (1-10) |
| `reasoning` | string | Judge's explanation |
| `confidence` | float | Judge's confidence (0.0-1.0) |
| `needs_review` | boolean | Whether human review is needed |
| `eval_result` | string | `pass`, `warn`, `fail`, `parse_error` |
| `judge_prompt` | string | Full prompt sent to judge |
| `judge_response` | string | Raw judge response |
| `evaluated_at` | timestamp | When evaluated |

### llm_evals__eval_scores

**Materialization**: Table | **Tags**: `llm_evals`, `evaluation`

Flattened view joining evaluations with capture data for easy querying.

| Column | Type | Description |
|--------|------|-------------|
| `eval_id` | string | Evaluation identifier |
| `capture_id` | string | Capture identifier |
| `source_model` | string | Source dbt model |
| `input_data` | variant/string | Original input |
| `output_data` | string | AI-generated output |
| `captured_at` | timestamp | Capture time |
| `criterion` | string | Evaluation criterion |
| `judge_model` | string | Judge model used |
| `score` | integer | Score (1-10) |
| `reasoning` | string | Judge explanation |
| `confidence` | float | Confidence (0.0-1.0) |
| `eval_result` | string | Result category |

---

## Monitoring Models

### llm_evals__performance_summary

**Materialization**: Table | **Tags**: `llm_evals`, `monitoring`, `reporting`

Daily aggregated performance metrics per model and criterion.

Key columns: `eval_date`, `source_model`, `criterion`, `avg_score`, `pass_rate`, `total_evaluations`, `health_status`

### llm_evals__drift_detection

**Materialization**: Table | **Tags**: `llm_evals`, `monitoring`, `reporting`

Statistical drift detection comparing recent performance against historical baselines.

### llm_evals__alerts

**Materialization**: Table | **Tags**: `llm_evals`, `monitoring`, `reporting`

Consolidated alert feed for drift, low pass rates, confidence issues, and parse errors.

### llm_evals__capture_status

**Materialization**: Table | **Tags**: `llm_evals`, `monitoring`, `reporting`

Status tracking for captured data and evaluation progress.
