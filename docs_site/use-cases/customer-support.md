# Use Case: Customer Support QA

Automatically evaluate AI-generated support responses for accuracy, tone, and helpfulness.

## The Problem

Your team uses an LLM to draft responses to customer support tickets. But how do you ensure:

- Responses are factually accurate?
- The tone is professional and empathetic?
- All parts of the customer's question are addressed?

Manual review doesn't scale. Spot-checking misses regressions. dbt-llm-evals automates this.

## Architecture

```
Support Tickets → LLM Response → Capture → Judge Evaluates → Alert on Issues
```

## Implementation

### 1. The AI Model

```sql
-- models/ai_support/support_responses.sql
{{ config(
    materialized='table',
    post_hook="{{ dbt_llm_evals.capture_and_evaluate() }}",
    meta={
        'llm_evals': {
            'enabled': true,
            'input_columns': ['customer_question', 'ticket_category', 'account_tier'],
            'output_column': 'ai_response',
            'prompt': 'You are a customer support agent. Category: {ticket_category}. Account: {account_tier}. Question: {customer_question}',
            'sampling_rate': 0.2
        }
    }
) }}

SELECT
    ticket_id,
    customer_question,
    ticket_category,
    account_tier,
    {{ dbt_llm_evals.llm_evals__ai_complete(
        'llama3-70b',
        "concat('You are a helpful customer support agent for a SaaS company. '
               'Ticket category: ', ticket_category,
               '. Account tier: ', account_tier,
               '. Customer question: ', customer_question,
               '. Provide a helpful, professional response:')"
    ) }} as ai_response
FROM {{ ref('stg_support_tickets') }}
WHERE status = 'open'
```

### 2. Configuration

```yaml
# dbt_project.yml
vars:
  llm_evals_judge_model: 'llama3-70b'
  llm_evals_criteria: '["accuracy", "tone", "helpfulness", "completeness"]'
  llm_evals_sampling_rate: 0.2
  llm_evals_pass_threshold: 7
```

### 3. Monitoring Query

```sql
-- Daily quality report for support responses
SELECT
    eval_date,
    criterion,
    avg_score,
    pass_rate,
    health_status
FROM llm_evals__performance_summary
WHERE source_model LIKE '%support_responses%'
ORDER BY eval_date DESC, criterion;
```

## Recommended Criteria

| Criterion | Why |
|-----------|-----|
| `accuracy` | Ensure responses contain correct information |
| `tone` | Maintain professional, empathetic voice |
| `helpfulness` | Responses should be actionable |
| `completeness` | Address all parts of the customer's question |

## Alerting

Set up alerts for quality drops:

```sql
-- Check for recent alerts
SELECT *
FROM llm_evals__alerts
WHERE source_model LIKE '%support_responses%'
  AND severity = 'ALERT'
ORDER BY alert_created_at DESC;
```

Common alert scenarios:

- **Drift detected** on `tone` — model may have started generating less empathetic responses
- **Low pass rate** on `accuracy` — factual errors increasing, possibly due to stale context data
- **Parse errors increasing** — judge model may need a temperature adjustment
