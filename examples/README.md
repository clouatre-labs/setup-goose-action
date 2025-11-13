# Example Workflows

Real-world examples of using setup-goose-action in GitHub Actions.

## Quick Start

All examples require a Goose-compatible API key as a GitHub secret:
- Google Gemini: `GEMINI_API_KEY`
- OpenAI: `OPENAI_API_KEY`
- Anthropic: `ANTHROPIC_API_KEY`

Add your API key to repository secrets: `Settings > Secrets and variables > Actions > New repository secret`

---

## Examples

### 1. [AI Code Review](pr-review.yml)
Automatically review pull requests for logic issues, bugs, and improvements.

**Use case:** Catch issues before human review, improve code quality

**Triggers:** On PR open/update

**Requirements:** Read access to code, write access to PRs

---

### 2. [Test Generation](test-generation.yml)
Generate unit tests for new code automatically.

**Use case:** Maintain test coverage, reduce manual test writing

**Triggers:** On PR with new source code

**Requirements:** Write access to create PRs

---

### 3. [Documentation Updates](documentation.yml)
Keep documentation in sync with code changes.

**Use case:** Prevent stale docs, automated API documentation

**Triggers:** On push to main

**Requirements:** Write access to commit docs

---

## Configuration Tips

### Performance Optimization
```yaml
- uses: clouatre-labs/setup-goose-action@v1
  with:
    version: '1.14.0'  # Pin version for reproducibility
```

### Multi-Model Strategy
```yaml
env:
  # Use fast model for simple tasks
  GOOSE_MODEL: gemini-2.0-flash-exp
  GOOGLE_API_KEY: ${{ secrets.GEMINI_API_KEY }}
```

### Cost Control
```yaml
# Run only on specific paths
on:
  pull_request:
    paths:
      - 'src/**'
      - '!src/test/**'  # Exclude test files
```

---

## Custom Workflows

Copy and adapt these examples for your needs:
- Code migration (framework upgrades)
- Security scanning (vulnerability detection)
- Refactoring suggestions
- Changelog generation
- Release notes writing

---

## Support

- Report issues: https://github.com/clouatre-labs/setup-goose-action/issues
- Goose docs: https://block.github.io/goose/
- Goose Discord: https://discord.gg/goose-oss
