# Architecture

## Package Structure

```
dbt-llm-evals/
├── macros/
│   ├── adapters/          # Warehouse-specific AI function calls
│   │   ├── dispatch.sql   # Dispatch layer (routes to correct adapter)
│   │   ├── snowflake/     # Snowflake Cortex implementation
│   │   ├── bigquery/      # BigQuery Vertex AI implementation
│   │   └── databricks/    # Databricks AI Functions implementation
│   ├── core/              # Core capture and baseline logic
│   │   ├── capture_io.sql # Main post-hook and capture macros
│   │   ├── baseline_check.sql
│   │   ├── ensure_raw_tables_exist.sql
│   │   └── get_package_schema.sql
│   └── judge/             # Judge prompt construction
│       └── build_judge_prompt.sql
├── models/
│   ├── core/              # Data processing models
│   ├── evaluation/        # AI judge evaluation engine
│   └── monitoring/        # Alerts and performance tracking
└── examples/              # Complete example projects per warehouse
```

## Multi-Warehouse Dispatch

dbt-llm-evals uses dbt's `dispatch` mechanism to support multiple warehouses with a single interface:

```
Your code calls:        dbt_llm_evals.llm_evals__ai_complete()
                                    │
                    ┌───────────────┼───────────────┐
                    ▼               ▼               ▼
            snowflake__         bigquery__      databricks__
          llm_evals__        llm_evals__      llm_evals__
           ai_complete()      ai_complete()    ai_complete()
                    │               │               │
                    ▼               ▼               ▼
              AI_COMPLETE()   AI.GENERATE()    ai_query()
```

This pattern means:

- Models and macros are written once, warehouse-agnostically
- The correct implementation is selected at runtime based on `target.type`
- Adding a new warehouse only requires new adapter macros

## Data Flow

```
┌──────────────┐
│ Your AI Model│
│  (post-hook) │
└──────┬───────┘
       │ INSERT INTO
       ▼
┌──────────────┐     ┌──────────────┐
│ raw_captures │     │ raw_baselines│
│  (table)     │     │  (table)     │
└──────┬───────┘     └──────┬───────┘
       │                    │
       ▼                    ▼
┌──────────────┐     ┌──────────────┐
│  captures    │     │  baselines   │
│(incremental) │     │  (model)     │
└──────┬───────┘     └──────┬───────┘
       │                    │
       └────────┬───────────┘
                ▼
       ┌────────────────┐
       │ judge_         │
       │ evaluations    │  ← AI function called here
       │ (incremental)  │
       └────────┬───────┘
                │
       ┌────────┼────────┐
       ▼        ▼        ▼
   ┌────────┐ ┌──────┐ ┌────────┐
   │summary │ │drift │ │alerts  │
   │(table) │ │(table)│ │(table) │
   └────────┘ └──────┘ └────────┘
```

## Storage Tables

Two raw tables are created by the setup model:

### raw_captures

| Column | Type | Description |
|--------|------|-------------|
| capture_id | VARCHAR | Surrogate key |
| source_model | VARCHAR | dbt model name |
| input_data | VARIANT/STRING | Input columns as JSON |
| output_data | VARCHAR | AI-generated output |
| prompt_data | VARCHAR | Prompt template used |
| captured_at | TIMESTAMP | Capture time |
| dbt_invocation_id | VARCHAR | dbt run identifier |
| eval_status | VARCHAR | `pending` or `completed` |
| evaluated_at | TIMESTAMP | When evaluated |

### raw_baselines

| Column | Type | Description |
|--------|------|-------------|
| baseline_id | VARCHAR | Surrogate key |
| source_model | VARCHAR | dbt model name |
| baseline_version | VARCHAR | Version tag (e.g., `v1.0`) |
| baseline_input | VARIANT/STRING | Sample input |
| baseline_output | VARCHAR | Sample output |
| baseline_created_at | TIMESTAMP | Creation time |
| dbt_invocation_id | VARCHAR | dbt run identifier |
| is_active | BOOLEAN | Whether this version is active |

## Incremental Processing

The evaluation model (`llm_evals__judge_evaluations`) uses incremental materialization:

- Only processes captures with `eval_status = 'pending'`
- Uses `captured_at > max(evaluated_at)` as the incremental filter
- Processes in batches controlled by `llm_evals_batch_size`
- Each criterion is evaluated separately (cross-join with criteria array)

This means evaluations are idempotent and can be resumed if a run fails partway through.
