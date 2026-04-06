# Contributing to dbt-llm-evals

Thank you for taking the time to contribute! 🎉

The following is a set of guidelines for contributing to dbt-llm-evals. These are
mostly guidelines, not rules. Use your best judgment, and feel free to propose
changes to this document in a pull request.

## How Can I Contribute?

### Reporting Bugs

If you find a bug, please report it by opening an issue on GitHub. Make sure to
include:

- A clear and descriptive title
- Steps to reproduce the problem
- Expected behavior
- Actual behavior
- Any relevant logs or error messages
- Your dbt version and warehouse type

### Suggesting Enhancements

If you have an idea to enhance dbt-llm-evals, we'd love to hear about it! Please
open an issue on GitHub and include:

- A clear and descriptive title
- A detailed description of the proposed enhancement
- Any relevant use cases or examples
- How it would benefit other users

## Getting Started

When you're ready to start working on an issue, follow these steps:

1. Fork the repository
2. Clone your fork: `git clone https://github.com/your-username/dbt-llm-evals.git`
3. Create a new branch: `git checkout -b feature/my-feature-branch` or `git checkout -b fix/my-bug-fix`
4. Install dependencies: `dbt™ deps`

## Development Setup

### Prerequisites
- dbt-core >= 1.6.0
- Access to at least one supported warehouse (Snowflake, BigQuery, or Databricks)
- Warehouse AI functions enabled

### Local Development

```bash
# Install dependencies
dbt deps

# Run models
dbt run --select tag:llm_evals

```

### Running Tests

Make sure all tests pass before submitting a pull request:

```bash
# Install test dependencies
poetry install

# Run Python validation tests
poetry run pytest

# Run compilation checks
dbt compile --select tag:llm_evals
dbt parse
```

## Contribution Guidelines

### Code Style

- Follow the existing code style and dbt best practices
- Use meaningful variable and model names
- Add comments for complex logic
- Keep SQL queries readable with proper indentation
- Ensure your code passes all tests

### Adding New Features

1. **New Evaluation Criteria**
   - Add to `build_judge_prompt` macro in `macros/judge/build_judge_prompt.sql`
   - Update documentation in README
   - Add example to integration tests

2. **New Warehouse Adapter**
   - Create adapter-specific macros in `macros/adapters/[warehouse]/`
   - Implement all required functions: `ai_complete`, `parse_json_response`, `current_timestamp`
   - Add integration test
   - Update README with setup instructions

3. **New Monitoring Features**
   - Add model to `models/monitoring/`
   - Document in schema YAML
   - Add to README

### Testing

- All new features must include tests
- Test on target warehouse if adding adapter-specific code
- Integration tests should use sample data

### Documentation

- Update README.md for user-facing changes
- Update CHANGELOG.md
- Add inline documentation for complex logic
- Update schema YAML files

### Commit Messages

This project uses [Conventional Commits](https://www.conventionalcommits.org/) to automate changelog generation and semantic versioning. Please format your commit messages as:

```
<type>: <description>

[optional body]
```

**Types:**
- `feat:` — new feature (bumps minor version)
- `fix:` — bug fix (bumps patch version)
- `docs:` — documentation changes
- `chore:` — maintenance tasks
- `refactor:` — code restructuring without behavior change
- `test:` — adding or updating tests

**Examples:**
```bash
git commit -m "feat: add support for custom evaluation criteria"
git commit -m "fix: handle null baseline_version in capture macro"
git commit -m "docs: add BigQuery troubleshooting guide"
```

A breaking change adds `!` after the type (e.g., `feat!: redesign capture API`) and bumps the major version.

### Pull Request Process

When you're ready to submit your changes:

1. Make your changes and commit using conventional commit format
2. Push to the branch: `git push origin feature/my-feature-branch`
3. Open a pull request
4. Update documentation if needed
5. Ensure all tests pass (CI runs automatically)
6. Submit PR with clear description of changes
7. Reference any related issues

> **Note:** CHANGELOG.md is generated automatically from conventional commits. You do not need to update it manually.

After creating the pull request, the PR will automatically notify the
maintainers, and they will be able to review your changes.

## Code of Conduct

This project adheres to the Contributor Covenant Code of Conduct.
By participating, you are expected to uphold this code:

- Be respectful and inclusive
- Welcome newcomers
- Focus on constructive feedback
- Assume good intentions

## Getting Help

If you need help or have any questions, feel free to:

- Open an issue on GitHub for bugs or feature requests
- Start a discussion for questions
- Check existing issues before creating new ones

## License

By contributing, you agree that your contributions will be licensed under the Apache 2.0 License.

Thank you for contributing!
