# BigQuery Vertex AI Setup

dbt-llm-evals uses [BigQuery Vertex AI integration](https://cloud.google.com/bigquery/docs/ai-application-overview) via `AI.GENERATE()` for evaluations.

## Prerequisites

- Google Cloud project with Vertex AI enabled
- BigQuery dataset with registered AI models
- A configured remote connection to Vertex AI
- Required IAM permissions for Vertex AI and BigQuery

## Configuration

```yaml
# dbt_project.yml
vars:
  llm_evals_judge_model: 'gemini-pro'
  llm_evals_criteria: '["accuracy", "relevance", "tone"]'
  llm_evals_sampling_rate: 0.1
  llm_evals_pass_threshold: 7

  # BigQuery-specific (required)
  gcp_project_id: 'my-gcp-project'
  gcp_location: 'us-central1'
  llm_evals_dataset: 'llm_models'
  ai_connection_id: 'projects/my-project/locations/us-central1/connections/my-vertex-connection'
```

## Setting Up Vertex AI in BigQuery

### 1. Create a Remote Connection

```sql
CREATE OR REPLACE EXTERNAL CONNECTION `my-project.us-central1.my-vertex-connection`
OPTIONS (
  type = 'CLOUD_RESOURCE'
);
```

### 2. Grant Permissions

Grant the connection's service account the `Vertex AI User` role in IAM.

### 3. Register a Model

```sql
CREATE OR REPLACE MODEL `my-project.llm_models.gemini-pro`
REMOTE WITH CONNECTION `my-project.us-central1.my-vertex-connection`
OPTIONS (
  endpoint = 'gemini-pro'
);
```

## How It Works on BigQuery

### AI Function Call

```sql
AI.GENERATE(
    prompt => <judge_prompt>,
    connection_id => 'projects/.../connections/my-vertex-connection',
    endpoint => 'gemini-pro'
).result
```

### JSON Parsing

```sql
SAFE.PARSE_JSON(judge_response) as parsed_json

-- Field extraction:
SAFE_CAST(JSON_VALUE(parsed_json, '$.score') AS INT64) as score
JSON_VALUE(parsed_json, '$.reasoning') as reasoning
SAFE_CAST(JSON_VALUE(parsed_json, '$.confidence') AS FLOAT64) as confidence
```

### Input Data Storage

Inputs are stored as JSON strings using `TO_JSON_STRING(STRUCT(...))`:

```sql
TO_JSON_STRING(STRUCT(
    customer_question,
    context
)) as input_data
```

### Criteria Flattening

```sql
CROSS JOIN UNNEST(["accuracy", "relevance", "tone"]) AS criteria
```

## Example Project

See `examples/example_project_bigquery/` for a complete working example.

## Troubleshooting

!!! warning "Common Issues"
    **"Not found: Connection"** — Verify the `ai_connection_id` variable matches your remote connection path exactly.

    **"Permission denied"** — Ensure the connection's service account has the `Vertex AI User` IAM role.

    **Quota errors** — Vertex AI has per-project quotas. Check your quota in the GCP console if evaluations fail intermittently.
