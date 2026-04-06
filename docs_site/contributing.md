# Contributing

We welcome contributions! See the full [CONTRIBUTING.md](https://github.com/paradime-io/dbt-llm-evals/blob/main/CONTRIBUTING.md) on GitHub for detailed guidelines.

## Quick Start

1. Fork the repository
2. Clone your fork: `git clone https://github.com/your-username/dbt-llm-evals.git`
3. Create a branch: `git checkout -b feature/my-feature`
4. Install dependencies: `dbt deps`
5. Make your changes
6. Run tests: `poetry install && poetry run pytest`
7. Submit a pull request

## Development Setup

```bash
# Install dbt dependencies
dbt deps

# Install Python test dependencies
poetry install

# Run tests
poetry run pytest

# Verify compilation
dbt compile --select tag:llm_evals
dbt parse
```

## Types of Contributions

- **Bug fixes** — open an issue first, then submit a PR
- **New evaluation criteria** — add to `build_judge_prompt` macro
- **New warehouse adapters** — create macros in `macros/adapters/[warehouse]/`
- **Documentation** — improvements to docs are always welcome
- **Examples** — additional use cases and example projects

## Code Style

- Follow existing SQL formatting conventions
- Use meaningful variable names
- Add comments for complex Jinja logic
- Keep SQL readable with proper indentation

## Questions?

Open an issue on [GitHub](https://github.com/paradime-io/dbt-llm-evals/issues).
