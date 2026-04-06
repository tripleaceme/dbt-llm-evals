# Snowflake Cortex Setup

dbt-llm-evals uses [Snowflake Cortex](https://docs.snowflake.com/en/user-guide/snowflake-cortex) `AI_COMPLETE()` for both AI model outputs and judge evaluations.

## Prerequisites

- Snowflake account with Cortex AI enabled
- `USAGE` privilege on Cortex functions
- A supported Cortex model (e.g., `llama3-70b`, `mistral-large`, `mixtral-8x7b`)

## Configuration

```yaml
# dbt_project.yml
vars:
  llm_evals_judge_model: 'llama3-70b'
  llm_evals_criteria: '["accuracy", "relevance", "tone"]'
  llm_evals_sampling_rate: 0.1
  llm_evals_pass_threshold: 7
  llm_evals_judge_temperature: 0.0
  llm_evals_judge_max_tokens: 500
```

## How It Works on Snowflake

### AI Function Call

The judge evaluation uses:

```sql
AI_COMPLETE(
    'llama3-70b',
    <judge_prompt>,
    OBJECT_CONSTRUCT(
        'temperature', 0.0,
        'max_tokens', 500
    )
)
```

### JSON Parsing

Responses are parsed using Snowflake's semi-structured data functions:

```sql
TRY_PARSE_JSON(judge_response) as parsed_json

-- Field extraction:
TRY_CAST(parsed_json:score::STRING AS INTEGER) as score
parsed_json:reasoning::STRING as reasoning
TRY_CAST(parsed_json:confidence::STRING AS FLOAT) as confidence
```

### Input Data Storage

Input columns are stored as `VARIANT` using `OBJECT_CONSTRUCT()`:

```sql
OBJECT_CONSTRUCT(
    'customer_question', customer_question,
    'context', context
) as input_data
```

### Criteria Flattening

Evaluation criteria are cross-joined using `LATERAL FLATTEN`:

```sql
CROSS JOIN LATERAL FLATTEN(
    input => PARSE_JSON('["accuracy", "relevance", "tone"]')
) criteria
```

## Example Project

A complete Snowflake example is included in the package at `examples/example_project_snowflake/`.

```sql
-- Example: Customer support response evaluation
{{ config(
    materialized='table',
    post_hook="{{ dbt_llm_evals.capture_and_evaluate() }}",
    meta={
        'llm_evals': {
            'enabled': true,
            'input_columns': ['customer_question', 'context'],
            'output_column': 'ai_response',
            'prompt': 'Context: {context}\nQuestion: {customer_question}\nAnswer:'
        }
    }
) }}

SELECT
    ticket_id,
    customer_question,
    context,
    snowflake.cortex.complete(
        'llama3-70b',
        concat('Context: ', context, '\nQuestion: ', customer_question, '\nAnswer:')
    ) as ai_response
FROM {{ ref('stg_support_tickets') }}
```

## Troubleshooting

!!! warning "Common Issues"
    **"Unknown function AI_COMPLETE"** — Cortex may not be enabled on your account or region. Check with your Snowflake admin.

    **Parse errors in evaluations** — Some models may wrap JSON in markdown code blocks. The package handles basic parsing, but if you see many `parse_error` results, try a different judge model.

    **High costs** — Each evaluation criterion generates a separate `AI_COMPLETE()` call. Lower `llm_evals_sampling_rate` or reduce the number of criteria to control costs.
