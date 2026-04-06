# Evaluation Criteria

dbt-llm-evals ships with seven built-in evaluation criteria. You can use any combination or add custom criteria.

## Built-in Criteria

### accuracy

Evaluates factual correctness of the output based on the input.

- **Scale**: 1 (completely inaccurate) to 10 (perfectly accurate)
- **Best for**: Factual Q&A, data summarization, information extraction

### relevance

Evaluates whether the output directly addresses the input.

- **Scale**: 1 (not relevant at all) to 10 (highly relevant)
- **Best for**: Search results, recommendations, contextual responses

### tone

Evaluates professional tone appropriateness.

- **Scale**: 1 (inappropriate tone) to 10 (perfect tone)
- **Best for**: Customer-facing content, support responses, formal communications

### completeness

Evaluates whether the output fully addresses all aspects of the input.

- **Scale**: 1 (incomplete) to 10 (comprehensive)
- **Best for**: Complex queries, multi-part questions, detailed instructions

### consistency

Evaluates alignment with baseline examples in quality and style.

- **Scale**: 1 (very inconsistent) to 10 (perfectly consistent)
- **Best for**: Brand voice, style guidelines, template-based outputs
- **Note**: Uses baseline examples for comparison

### helpfulness

Evaluates whether the output is actionable and useful for the user.

- **Scale**: 1 (not helpful) to 10 (extremely helpful)
- **Best for**: Support tickets, how-to content, troubleshooting guides

### clarity

Evaluates structure, readability, and ease of understanding.

- **Scale**: 1 (very unclear) to 10 (perfectly clear)
- **Best for**: Technical documentation, explanations, instructional content

## Configuring Criteria

Set your criteria as a JSON array in `dbt_project.yml`:

```yaml
vars:
  llm_evals_criteria: '["accuracy", "relevance", "tone"]'
```

### Choosing Criteria

Select criteria based on your use case:

| Use Case | Recommended Criteria |
|----------|---------------------|
| Customer support | `accuracy`, `tone`, `helpfulness` |
| Content generation | `relevance`, `clarity`, `completeness` |
| Data enrichment | `accuracy`, `completeness` |
| Chatbot responses | `relevance`, `helpfulness`, `tone` |
| Technical docs | `accuracy`, `clarity`, `completeness` |

!!! tip "Cost Consideration"
    Each criterion generates a separate AI judge call per captured output. More criteria = higher warehouse AI costs. Start with 2-3 key criteria and expand as needed.

## Custom Criteria

You can use any string as a criterion name. The judge will evaluate using a generic prompt:

```yaml
vars:
  llm_evals_criteria: '["accuracy", "brand_voice", "safety"]'
```

For custom criteria, the judge prompt uses:

- **Title**: The criterion name (title-cased)
- **Description**: "Evaluate the quality of the output based on {criterion}"
- **Scale**: 1 (very poor) to 10 (excellent)

The judge model is generally capable of interpreting custom criterion names in context. For best results, use descriptive names that clearly convey what to evaluate.
