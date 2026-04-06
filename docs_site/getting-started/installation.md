# Installation

## Package Installation

Add dbt-llm-evals to your `packages.yml`:

```yaml
packages:
  - git: "https://github.com/paradime-io/dbt-llm-evals.git"
    revision: 1.0.1
```

Install the package:

```bash
dbt deps
```

## Requirements

| Requirement | Version |
|-------------|---------|
| dbt-core | >= 1.6.0, < 2.0.0 |
| dbt_utils | Installed automatically |

## Warehouse Requirements

### Snowflake

- Cortex AI functions must be enabled on your account
- Required privilege: `USAGE` on the Cortex functions
- Supported models: `llama3-70b`, `mistral-large`, `mixtral-8x7b`, and others

### BigQuery

- Vertex AI models must be registered in BigQuery
- A remote connection must be configured
- Required: `gcp_project_id`, `gcp_location`, `llm_evals_dataset`, and `ai_connection_id` variables

### Databricks

- AI Functions must be enabled on your workspace
- Supported models: `llama-2-70b-chat` and others available via `ai_query()`

## Initial Setup

After installing the package, run the setup model to create storage tables:

```bash
dbt run --select llm_evals__setup
```

This creates two tables in your target schema (or a custom schema if configured):

- **`raw_captures`** — stores captured inputs/outputs from your AI models
- **`raw_baselines`** — stores baseline samples for comparison

!!! tip "Custom Schema"
    To store evaluation data in a separate schema, set the `llm_evals_schema` variable:
    ```yaml
    vars:
      llm_evals_schema: 'llm_evals'
    ```

## Verifying Installation

Run a quick check to ensure everything is set up:

```bash
# Verify package compiles
dbt compile --select tag:llm_evals

# Check that models parse correctly
dbt parse
```

If both commands succeed without errors, you're ready to [configure your first evaluation](configuration.md).
