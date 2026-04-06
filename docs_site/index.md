# dbt-llm-evals

**Evaluate LLM outputs directly in your data warehouse using dbt.**

dbt-llm-evals is a dbt package that enables automated evaluation of AI/LLM-generated content using warehouse-native AI functions. No external API calls, no data egress — everything runs inside your warehouse.

---

## Why dbt-llm-evals?

As teams deploy LLMs in production data pipelines, a critical gap emerges: **how do you know your AI outputs are good?** Traditional software testing doesn't apply to non-deterministic AI outputs. Manual review doesn't scale.

dbt-llm-evals solves this by using an **LLM-as-a-Judge** approach — your warehouse's built-in AI functions evaluate AI outputs against configurable criteria, automatically.

### Key Features

- **Zero data egress** — evaluations run inside your warehouse using native AI functions
- **Automatic baselines** — captures reference outputs on first run, no manual setup
- **Multi-criteria scoring** — evaluate accuracy, relevance, tone, completeness, and more
- **Drift detection** — statistical alerts when output quality degrades
- **Multi-warehouse** — works with Snowflake Cortex, BigQuery Vertex AI, and Databricks AI Functions

---

## How It Works

```
Your AI Model → Capture I/O → Judge Evaluates → Monitor & Alert
```

1. **Capture** — A post-hook on your dbt model automatically captures inputs, outputs, and prompts
2. **Baseline** — On first run, samples are stored as quality benchmarks
3. **Evaluate** — A warehouse AI judge scores each output on your chosen criteria (1-10)
4. **Monitor** — Performance summaries, drift detection, and alerts surface quality issues

---

## Quick Example

```sql
-- models/ai_content/product_descriptions.sql
{{ config(
    materialized='table',
    post_hook="{{ dbt_llm_evals.capture_and_evaluate() }}",
    meta={
        'llm_evals': {
            'enabled': true,
            'input_columns': ['product_name', 'features'],
            'output_column': 'ai_description'
        }
    }
) }}

SELECT
    product_id,
    product_name,
    features,
    snowflake.cortex.complete(
        'llama3-70b',
        concat('Write a product description for: ', product_name,
               '. Features: ', features)
    ) as ai_description
FROM {{ ref('stg_products') }}
```

That's it. Your AI outputs are now being evaluated automatically.

---

## Supported Warehouses

| Warehouse | AI Function | Status |
|-----------|------------|--------|
| Snowflake | Cortex `AI_COMPLETE()` | Fully supported |
| BigQuery | Vertex AI `AI.GENERATE()` | Fully supported |
| Databricks | `ai_query()` | Fully supported |

---

## Get Started

Ready to start evaluating your LLM outputs?

[Quick Start Guide](getting-started/quickstart.md){ .md-button .md-button--primary }
[View on GitHub](https://github.com/paradime-io/dbt-llm-evals){ .md-button }
