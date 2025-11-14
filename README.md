# Setup Goose Action

[![Test Action](https://github.com/clouatre-labs/setup-goose-action/actions/workflows/test.yml/badge.svg)](https://github.com/clouatre-labs/setup-goose-action/actions/workflows/test.yml)
[![GitHub Marketplace](https://img.shields.io/badge/Marketplace-Setup%20Goose%20CLI-blue?logo=github)](https://github.com/marketplace/actions/setup-goose-cli)
[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Latest Release](https://img.shields.io/github/v/release/clouatre-labs/setup-goose-action)](https://github.com/clouatre-labs/setup-goose-action/releases/latest)

GitHub Action to install and cache [Goose AI agent](https://github.com/block/goose) for use in workflows.

**Available on the [GitHub Marketplace](https://github.com/marketplace/actions/setup-goose-cli)**

## Versioning

This action uses **independent versioning** from Goose itself.

### Version Compatibility

| Action Version | Default Goose Version | Release Date |
|---------------|----------------------|--------------|
| v1.0.3 | 1.14.0 | 2025-11-13 |
| v1.0.1 | 1.14.0 | 2025-11-12 |
| v1.0.0 | 1.12.1 | 2025-11-06 |

### Recommended Usage

```yaml
# Recommended: Get latest v1.x updates automatically
- uses: clouatre-labs/setup-goose-action@v1

# Conservative: Pin to exact version
- uses: clouatre-labs/setup-goose-action@v1.0.3

# Custom Goose version
- uses: clouatre-labs/setup-goose-action@v1
  with:
    version: '1.13.0'
```

**Current default Goose version:** See [`action.yml`](action.yml#L9)

## Prerequisites

1. **Get an API key** from your chosen provider: [Supported Providers](https://block.github.io/goose/docs/getting-started/providers/)

2. **Add it as a repository secret:**
   - Go to **Settings → Secrets and variables → Actions**
   - Click **New repository secret**
   - Name it (e.g., `GEMINI_API_KEY`, `OPENAI_API_KEY`, `ANTHROPIC_API_KEY`)

3. **Configure in your workflow** - map your secret to Goose's expected environment variable (see examples below)

## Quick Start

This workflow runs a review and posts the results as a comment on the PR.

```yaml
name: AI Code Review with PR Comment
on: [pull_request]

permissions:
  pull-requests: write

jobs:
  review:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - uses: clouatre-labs/setup-goose-action@v1

      - name: Configure and Run Review
        id: review
        env:
          GOOGLE_API_KEY: ${{ secrets.GOOGLE_API_KEY }}
        run: |
          # Configure Goose
          mkdir -p ~/.config/goose
          cat > ~/.config/goose/config.yaml << EOF
          GOOSE_PROVIDER: google
          GOOSE_MODEL: gemini-2.5-flash
          keyring: false
          EOF

          # Generate diff and check if empty
          git diff origin/${{ github.base_ref }}...HEAD > changes.diff
          if [ ! -s changes.diff ]; then
            echo "No changes detected." > review-comment.md
            echo "comment-file=review-comment.md" >> $GITHUB_OUTPUT
            exit 0
          fi

          # Create instructions and run review
          echo "Review this diff for bugs and logic errors:" > instructions.txt
          cat changes.diff >> instructions.txt
          goose run --instructions instructions.txt --no-session --quiet > review-comment.md 2>&1
          
          echo "comment-file=review-comment.md" >> $GITHUB_OUTPUT

      - name: Post Review Comment to PR
        uses: actions/github-script@v7
        with:
          script: |
            const fs = require('fs');
            const body = fs.readFileSync('${{ steps.review.outputs.comment-file }}', 'utf8');
            github.rest.issues.createComment({
              owner: context.repo.owner,
              repo: context.repo.repo,
              issue_number: context.issue.number,
              body: body
            });
```

## Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `version` | Goose version to install | No | See [`action.yml`](action.yml#L9) |

## Outputs

| Output | Description |
|--------|-------------|
| `goose-version` | Installed Goose version |
| `goose-path` | Path to Goose binary directory |

## Examples

### Security Scan with Artifact Upload

```yaml
name: Security Scan
on: [push]

permissions:
  contents: read

jobs:
  scan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - uses: clouatre-labs/setup-goose-action@v1
      
      - name: Scan files for security issues
        env:
          ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}
        run: |
          mkdir -p reports
          find . -name "*.py" -o -name "*.js" | head -5 | while read file; do
            echo "Scanning $file..." >&2
            echo "Review this file for security vulnerabilities: $(cat $file)" | \
              goose run --instructions - --no-session --quiet >> reports/security-scan.txt
            echo -e "\n---\n" >> reports/security-scan.txt
          done
      
      - uses: actions/upload-artifact@v4
        with:
          name: security-scan-results
          path: reports/
          retention-days: 30
```

### Pin to Specific Version

```yaml
- uses: clouatre-labs/setup-goose-action@v1
  with:
    version: '1.14.0'
```

## Features

- **Caching**: Automatically caches Goose binary for faster subsequent runs
- **Version Pinning**: Install specific Goose versions for reproducible builds
- **Cross-Platform**: Supports Linux (x64, arm64) and macOS (x64, arm64)
- **Lightweight**: Composite action with no external dependencies

## Supported Platforms

| OS | Architecture | Status |
|----|--------------|--------|
| Ubuntu | x64 | ✅ Supported |
| Ubuntu | arm64 | ✅ Supported |
| macOS | x64 | ✅ Supported |
| macOS | arm64 | ✅ Supported |
| Windows | - | ❌ Not supported |

## How It Works

1. Checks cache for Goose binary matching the specified version and platform
2. If cache miss, downloads Goose binary from official GitHub releases
3. Extracts binary to `~/.local/bin/goose`
4. Adds binary location to `$GITHUB_PATH`
5. Verifies installation with `goose --version`

## Cache Key Format

```
goose-{version}-{os}-{arch}
```

Example: `goose-1.12.1-Linux-X64`

## Troubleshooting

### Binary not found after installation

Ensure you're using the action before attempting to run `goose`:

```yaml
- uses: clouatre-labs/setup-goose-action@v1
- run: goose --version  # This will work
```

### Unsupported version

Check available versions at [Goose Releases](https://github.com/block/goose/releases). Ensure the version exists and has pre-built binaries.

### Cache not working

The cache key includes OS and architecture. If you change runners or platforms, a new cache entry will be created. This is expected behavior.

## Development

This is a composite action (YAML-based) with no compilation required.

### Testing Locally

```bash
# Test in a workflow
git clone https://github.com/clouatre-labs/setup-goose-action
cd setup-goose-action

# Create a test workflow in .github/workflows/test.yml
# Push and verify the action works
```

## Contributing

Contributions are welcome! Please open an issue or PR.

## License

MIT - See [LICENSE](LICENSE)

## Related

- [Goose](https://github.com/block/goose) - Official Goose repository
- [Goose Documentation](https://block.github.io/goose/)
- [GitHub Actions Documentation](https://docs.github.com/en/actions)

## Acknowledgments

Built by [clouatre-labs](https://github.com/clouatre-labs) for the Goose community. Not officially affiliated with Block or the Goose project.
