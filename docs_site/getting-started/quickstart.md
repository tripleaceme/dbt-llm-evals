# Quick Start Guide

Get dbt-llm-evals running in 5 minutes.

## Prerequisites

- dbt-core >= 1.6.0
- Access to a supported warehouse with AI functions enabled:
    - Snowflake with Cortex enabled
    - BigQuery with Vertex AI models registered
    - Databricks with AI Functions enabled

## Step 1: Install the Package

Add to your `packages.yml`:

```yaml
packages:
  - git: "https://github.com/paradime-io/dbt-llm-evals.git"
    revision: 1.0.1
```

Then install:

```bash
dbt deps
```

## Step 2: Run Setup

Create the storage tables that hold captures and baselines:

```bash
dbt run --select llm_evals__setup
```

## Step 3: Configure Variables

Add required variables to your `dbt_project.yml`:

=== "Snowflake"

    ```yaml
    vars:
      llm_evals_judge_model: 'llama3-70b'
      llm_evals_criteria: '["accuracy", "relevance", "tone"]'
      llm_evals_sampling_rate: 0.1
      llm_evals_pass_threshold: 7
    ```

=== "BigQuery"

    ```yaml
    vars:
      llm_evals_judge_model: 'gemini-pro'
      llm_evals_criteria: '["accuracy", "relevance", "tone"]'
      llm_evals_sampling_rate: 0.1
      llm_evals_pass_threshold: 7
      gcp_project_id: 'my-project'
      gcp_location: 'us-central1'
      llm_evals_dataset: 'llm_models'
      ai_connection_id: 'projects/.../connections/my-vertex-connection'
    ```

=== "Databricks"

    ```yaml
    vars:
      llm_evals_judge_model: 'llama-2-70b-chat'
      llm_evals_criteria: '["accuracy", "relevance", "tone"]'
      llm_evals_sampling_rate: 0.1
      llm_evals_pass_threshold: 7
    ```

## Step 4: Configure Your AI Model

Add the evaluation post-hook to your model's YAML config:

```yaml
version: 2

models:
  - name: your_ai_model
    config:
      materialized: table
      post_hook: "{{ dbt_llm_evals.capture_and_evaluate() }}"
      meta:
        llm_evals:
          enabled: true
          input_columns:
            - customer_question
            - context
          output_column: 'ai_response'
          sampling_rate: 0.1
```

## Step 5: Run Your Model

```bash
dbt run --select your_ai_model
```

On first run, the package automatically:

- Detects no baseline exists
- Creates a baseline with 100 samples
- Captures inputs and outputs for evaluation

## Step 6: Run Evaluations

```bash
dbt run --select tag:llm_evals
```

This scores all pending captures against your criteria using the AI judge.

## Step 7: Check Results

```sql
-- Performance summary
SELECT * FROM llm_evals__performance_summary
ORDER BY eval_date DESC;

-- Active alerts
SELECT * FROM llm_evals__alerts
WHERE severity = 'ALERT';

-- Low-scoring outputs
SELECT input_data, output_data, criterion, score, reasoning
FROM llm_evals__eval_scores
WHERE score < 5
ORDER BY score ASC;
```

## Next Steps

- [Configuration Reference](configuration.md) — all available variables and options
- [Warehouse Setup](../warehouses/snowflake.md) — detailed per-warehouse instructions
- [Evaluation Criteria](../concepts/evaluation-criteria.md) — customize what gets evaluated
