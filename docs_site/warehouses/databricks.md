# Databricks AI Functions Setup

dbt-llm-evals uses [Databricks AI Functions](https://docs.databricks.com/en/large-language-models/ai-functions.html) via `ai_query()` for evaluations.

## Prerequisites

- Databricks workspace with AI Functions enabled
- Access to Foundation Model APIs or external model endpoints
- Supported models: `llama-2-70b-chat` and others available via `ai_query()`

## Configuration

```yaml
# dbt_project.yml
vars:
  llm_evals_judge_model: 'llama-2-70b-chat'
  llm_evals_criteria: '["accuracy", "relevance", "tone"]'
  llm_evals_sampling_rate: 0.1
  llm_evals_pass_threshold: 7
```

## How It Works on Databricks

### AI Function Call

```sql
ai_query(
    'llama-2-70b-chat',
    <judge_prompt>,
    NAMED_STRUCT(
        'temperature', 0.0,
        'max_tokens', 500
    )
)
```

### JSON Parsing

```sql
FROM_JSON(
    judge_response,
    'STRUCT<score: INT, reasoning: STRING, confidence: DOUBLE>'
) as parsed_json

-- Field extraction:
CAST(parsed_json.score AS INT) as score
parsed_json.reasoning as reasoning
CAST(parsed_json.confidence AS DOUBLE) as confidence
```

### Input Data Storage

Inputs are stored using `NAMED_STRUCT()`:

```sql
NAMED_STRUCT(
    'customer_question', customer_question,
    'context', context
) as input_data
```

### Criteria Flattening

```sql
LATERAL VIEW EXPLODE(
    FROM_JSON('["accuracy", "relevance", "tone"]', 'ARRAY<STRING>')
) AS criteria
```

## Example Project

See `examples/example_project_databricks/` for a complete working example.

## Troubleshooting

!!! warning "Common Issues"
    **"AI_QUERY function not found"** — AI Functions may not be enabled on your workspace. Contact your Databricks admin.

    **Model not available** — Not all models are available in all regions. Check the Databricks documentation for model availability.

    **Schema parsing errors** — The `FROM_JSON` function requires an explicit schema. If the judge model returns unexpected formats, you may see parse errors.
