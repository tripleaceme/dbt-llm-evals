# Macros Reference

## Core Macros

### capture_and_evaluate()

The main entry point. Used as a `post_hook` on your AI models.

```sql
post_hook: "{{ dbt_llm_evals.capture_and_evaluate() }}"
```

**Behavior**:

1. Reads `meta.llm_evals` config from the model
2. Ensures storage tables exist
3. Checks/creates baseline if needed
4. Captures a sample of inputs and outputs

**Requires**: `meta.llm_evals.enabled: true`, `input_columns`, and `output_column` in model config.

### capture_io_data_simple()

Captures input/output data from a model into `raw_captures`.

```
capture_io_data_simple(model_relation, input_columns, output_column, sampling_rate, prompt_text=none)
```

| Parameter | Type | Description |
|-----------|------|-------------|
| `model_relation` | relation | The dbt model relation |
| `input_columns` | list | Column names to capture as input |
| `output_column` | string | Column containing AI output |
| `sampling_rate` | float | Fraction of rows to capture (0.0-1.0) |
| `prompt_text` | string | Optional prompt template |

### create_baseline_snapshot()

Creates a new baseline version from the current model output.

```
create_baseline_snapshot(model_relation, input_columns, output_column, baseline_version)
```

| Parameter | Type | Description |
|-----------|------|-------------|
| `model_relation` | relation | The dbt model relation |
| `input_columns` | list | Column names to capture |
| `output_column` | string | Output column name |
| `baseline_version` | string | Version tag (e.g., `v1.0`) |

**Behavior**: Deactivates existing baselines for the model, then inserts new samples.

### check_and_create_baseline()

Checks if a baseline version exists; creates one if it doesn't.

```
check_and_create_baseline(model_relation, input_columns, output_column, baseline_version)
```

### ensure_raw_tables_exist()

Idempotently creates `raw_captures` and `raw_baselines` tables. Handles schema evolution by adding missing columns.

### get_package_schema()

Returns the resolved schema name for raw tables. Uses `llm_evals_schema` variable if set, otherwise falls back to the target schema.

---

## Adapter Macros

These are dispatched based on your warehouse type. You typically don't call them directly.

### llm_evals__ai_complete()

Calls the warehouse AI function.

```
llm_evals__ai_complete(model, prompt, options={})
```

| Parameter | Description |
|-----------|-------------|
| `model` | AI model name (e.g., `llama3-70b`) |
| `prompt` | SQL expression for the prompt |
| `options` | Dict with `temperature` and `max_tokens` |

**Dispatches to**:

- Snowflake: `AI_COMPLETE()`
- BigQuery: `AI.GENERATE()`
- Databricks: `ai_query()`

### llm_evals__parse_json_response()

Parses the judge's JSON response into a structured format.

**Dispatches to**:

- Snowflake: `TRY_PARSE_JSON()`
- BigQuery: `SAFE.PARSE_JSON()`
- Databricks: `FROM_JSON()`

### llm_evals__current_timestamp()

Returns a timezone-aware timestamp appropriate for the warehouse.

---

## Judge Macros

### build_judge_prompt()

Constructs the evaluation prompt sent to the AI judge.

```
build_judge_prompt(input_col, output_col, criterion, baseline_examples=none, prompt_data_col=none)
```

| Parameter | Type | Description |
|-----------|------|-------------|
| `input_col` | string | SQL column reference for input data |
| `output_col` | string | SQL column reference for output data |
| `criterion` | string | Evaluation criterion name |
| `baseline_examples` | string | Optional SQL column with baseline text |
| `prompt_data_col` | string | Optional SQL column with prompt template |

**Returns**: A SQL `CONCAT()` expression that builds the full judge prompt.

### get_eval_criteria()

Parses the `llm_evals_criteria` variable from JSON string to a list.

```sql
{% set criteria = dbt_llm_evals.get_eval_criteria() %}
-- Returns: ['accuracy', 'relevance', 'tone']
```
