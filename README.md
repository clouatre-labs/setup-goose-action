# Setup Goose Action

[![Test](https://github.com/clouatre-labs/setup-goose-action/actions/workflows/test.yml/badge.svg)](https://github.com/clouatre-labs/setup-goose-action/actions/workflows/test.yml)
[![GitHub Marketplace](https://img.shields.io/badge/Marketplace-Setup%20Goose%20CLI-blue?logo=github)](https://github.com/marketplace/actions/setup-goose-cli)
[![Latest Release](https://img.shields.io/github/v/release/clouatre-labs/setup-goose-action)](https://github.com/clouatre-labs/setup-goose-action/releases/latest)
[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)

Install and cache [Goose AI agent](https://github.com/block/goose) in GitHub Actions workflows.

> [!WARNING]
> AI analyzing user-controlled input (diffs, comments, commits) is vulnerable to prompt injection. See [Security Patterns](#security-patterns).

## Usage

```yaml
# Recommended
- uses: clouatre-labs/setup-goose-action@v1

# Pin to specific version
- uses: clouatre-labs/setup-goose-action@v1
  with:
    version: '1.19.1'

# Always use latest release
- uses: clouatre-labs/setup-goose-action@v1
  with:
    check-latest: true
```

Default version: See [`action.yml`](action.yml#L16)

## Prerequisites

1. Get API key from [supported provider](https://block.github.io/goose/docs/getting-started/providers/)
2. Add as repository secret: **Settings → Secrets and variables → Actions**
3. Map secret to environment variable in workflow (see examples below)

## Quick Start - Tier 1 (Maximum Security)

```yaml
name: Secure AI Analysis
on: [pull_request]

permissions:
  contents: read

jobs:
  analyze:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v6

      - name: Lint Code
        run: pipx run ruff check --output-format=json . > lint-results.json || true

      - name: Setup Goose CLI
        uses: clouatre-labs/setup-goose-action@v1

      - name: AI Analysis
        env:
          GOOGLE_API_KEY: ${{ secrets.GOOGLE_API_KEY }}
        run: |
          mkdir -p ~/.config/goose
          cat > ~/.config/goose/config.yaml << 'EOF'
          GOOSE_PROVIDER: google
          GOOSE_MODEL: gemini-2.5-flash
          keyring: false
          EOF
          
          echo "Summarize these linting issues:" > prompt.txt
          cat lint-results.json >> prompt.txt
          goose run --instructions prompt.txt --no-session --quiet > analysis.md

      - name: Upload Analysis Artifact
        uses: actions/upload-artifact@v5
        with:
          name: ai-analysis
          path: analysis.md
```

## Inputs & Outputs

| Input | Description | Default |
|-------|-------------|---------|
| `version` | Goose version (ignored if `check-latest: true`) | See [action.yml](action.yml#L16) |
| `check-latest` | Install latest release (ignores `version`) | `false` |

| Output | Description |
|--------|-------------|
| `goose-version` | Installed version |
| `goose-path` | Binary directory path |
| `cache-hit` | `'true'` if restored from cache |

## Security Patterns

Three defensive tiers for AI-augmented CI/CD:

- **Tier 1**: AI analyzes tool output only (JSON from ruff/trivy/semgrep) - [example](examples/tier1-maximum-security.yml)
- **Tier 2**: AI sees file stats, requires manual approval - [example](examples/tier2-balanced-security.yml)
- **Tier 3**: Full diff analysis, trusted contributors only - [example](examples/tier3-advanced-patterns.yml)

**Safe:** AI analyzes structured tool output  
**Unsafe:** AI analyzes raw diffs (prompt injection risk)

**Checksums:** Not available - upstream [block/goose](https://github.com/block/goose) doesn't publish them ([tracking issue](https://github.com/block/goose/issues/5994))

Report vulnerabilities: [SECURITY.md](SECURITY.md)

## Platform Support

| OS | Arch | Status |
|----|------|--------|
| Linux | x64, arm64 | ✅ |
| macOS, Windows | - | ❌ |

Linux-only by design (10x cost multiplier for macOS runners, no platform-specific use case)

## How It Works

1. Check cache: `goose-{version}-{os}-{arch}`
2. Download from [GitHub releases](https://github.com/block/goose/releases) on cache miss
3. Extract to `~/.local/bin/goose`
4. Add to `$GITHUB_PATH`
5. Verify with `goose --version`

## License

MIT - See [LICENSE](LICENSE)

Not affiliated with Block or the Goose project.
