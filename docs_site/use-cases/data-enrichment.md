# Use Case: Data Enrichment

Evaluate AI-powered data classification, extraction, and enrichment pipelines.

## The Problem

Your pipeline uses an LLM to:

- Classify support tickets by category
- Extract entities from unstructured text
- Enrich records with AI-generated summaries
- Categorize products or transactions

These enrichments feed downstream analytics. Inaccurate classifications corrupt your data. You need to detect quality issues before they propagate.

## Implementation

### 1. Ticket Classification Model

```sql
-- models/ai_enrichment/ticket_classification.sql
{{ config(
    materialized='incremental',
    unique_key='ticket_id',
    post_hook="{{ dbt_llm_evals.capture_and_evaluate() }}",
    meta={
        'llm_evals': {
            'enabled': true,
            'input_columns': ['ticket_subject', 'ticket_body'],
            'output_column': 'ai_category',
            'prompt': 'Classify this support ticket. Subject: {ticket_subject}. Body: {ticket_body}. Categories: billing, technical, account, feature_request, other',
            'sampling_rate': 0.1
        }
    }
) }}

SELECT
    ticket_id,
    ticket_subject,
    ticket_body,
    {{ dbt_llm_evals.llm_evals__ai_complete(
        'llama3-70b',
        "concat('Classify this support ticket into exactly one category: '
               'billing, technical, account, feature_request, other. '
               'Subject: ', ticket_subject,
               '. Body: ', ticket_body,
               '. Respond with only the category name.')"
    ) }} as ai_category
FROM {{ ref('stg_tickets') }}
{% if is_incremental() %}
WHERE created_at > (SELECT MAX(created_at) FROM {{ this }})
{% endif %}
```

### 2. Configuration

```yaml
vars:
  llm_evals_judge_model: 'llama3-70b'
  llm_evals_criteria: '["accuracy", "consistency"]'
  llm_evals_pass_threshold: 8  # Higher bar for data enrichment
```

## Recommended Criteria

| Criterion | Why |
|-----------|-----|
| `accuracy` | Classification must be correct |
| `consistency` | Same type of ticket should get same category |

!!! tip "Higher Thresholds"
    For data enrichment, consider a higher `pass_threshold` (8+) since incorrect classifications directly impact downstream analytics.

## Monitoring Classification Drift

```sql
-- Detect if classification accuracy is dropping
SELECT
    eval_date,
    avg_score,
    pass_rate,
    total_evaluations,
    health_status
FROM llm_evals__performance_summary
WHERE source_model LIKE '%ticket_classification%'
  AND criterion = 'accuracy'
ORDER BY eval_date DESC
LIMIT 14;
```

## Common Patterns

### Entity Extraction

```yaml
meta:
  llm_evals:
    enabled: true
    input_columns: ['raw_text']
    output_column: 'extracted_entities'
    prompt: 'Extract all company names, amounts, and dates from: {raw_text}'
```

### Sentiment Analysis

```yaml
meta:
  llm_evals:
    enabled: true
    input_columns: ['review_text']
    output_column: 'sentiment_label'
    prompt: 'Classify sentiment as positive, negative, or neutral: {review_text}'
```

For enrichment tasks, the `accuracy` and `consistency` criteria are the most important — they ensure your AI-enriched data remains reliable for downstream consumers.
