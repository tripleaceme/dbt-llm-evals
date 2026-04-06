# Use Case: Content Generation

Evaluate AI-generated marketing copy, product descriptions, and other content at scale.

## The Problem

Your pipeline generates product descriptions, blog summaries, or marketing copy using an LLM. Quality varies — some outputs are great, others miss the mark. You need automated quality gates.

## Implementation

### 1. Product Description Model

```sql
-- models/ai_content/product_descriptions.sql
{{ config(
    materialized='table',
    post_hook="{{ dbt_llm_evals.capture_and_evaluate() }}",
    meta={
        'llm_evals': {
            'enabled': true,
            'input_columns': ['product_name', 'features', 'target_audience'],
            'output_column': 'ai_description',
            'sampling_rate': 0.15
        }
    }
) }}

SELECT
    product_id,
    product_name,
    features,
    target_audience,
    {{ dbt_llm_evals.llm_evals__ai_complete(
        'llama3-70b',
        "concat('Write a compelling product description. '
               'Product: ', product_name,
               '. Features: ', features,
               '. Target audience: ', target_audience)"
    ) }} as ai_description
FROM {{ ref('stg_products') }}
```

### 2. Configuration

```yaml
vars:
  llm_evals_judge_model: 'llama3-70b'
  llm_evals_criteria: '["relevance", "clarity", "tone", "completeness"]'
  llm_evals_pass_threshold: 7
```

## Recommended Criteria

| Criterion | Why |
|-----------|-----|
| `relevance` | Description should match the actual product |
| `clarity` | Easy to read and understand |
| `tone` | Matches brand voice |
| `completeness` | Covers key features and benefits |
| `consistency` | Maintains uniform style across products |

## Quality Gates

Use evaluation scores to gate content before it goes live:

```sql
-- Only publish descriptions that pass all criteria
SELECT
    p.product_id,
    p.ai_description
FROM product_descriptions p
INNER JOIN (
    SELECT
        capture_id,
        MIN(score) as min_score
    FROM llm_evals__eval_scores
    WHERE source_model LIKE '%product_descriptions%'
    GROUP BY capture_id
    HAVING MIN(score) >= 7
) q ON p.product_id = q.capture_id
```

## Tracking Improvements

When you update your prompt or switch models, create a new baseline version:

```yaml
meta:
  llm_evals:
    enabled: true
    baseline_version: 'v2.0'  # New version after prompt change
    input_columns: ['product_name', 'features', 'target_audience']
    output_column: 'ai_description'
```

Compare performance across versions using the monitoring models to validate that changes actually improved output quality.
