# Setup Goose Action

<p align="center">
  <a href="https://github.com/clouatre-labs/setup-goose-action/actions/workflows/test.yml"><img alt="CI" src="https://github.com/clouatre-labs/setup-goose-action/actions/workflows/test.yml/badge.svg"></a>
  <a href="https://github.com/marketplace/actions/setup-goose-cli"><img alt="GitHub Marketplace" src="https://img.shields.io/badge/Marketplace-Setup%20Goose%20CLI-blue?logo=github"></a>
  <a href="LICENSE"><img alt="License: MIT" src="https://img.shields.io/badge/License-MIT-blue.svg"></a>
  <a href="https://www.bestpractices.dev/projects/11555"><img alt="OpenSSF Best Practices" src="https://www.bestpractices.dev/projects/11555/badge"></a>
</p>

<p align="center">A GitHub Action that installs and caches the <a href="https://github.com/block/goose">Goose AI agent</a> for CI/CD workflows. OpenSSF silver certified: fewer than 1% of open source projects reach this level.</p>

**Available on the [GitHub Marketplace](https://github.com/marketplace/actions/setup-goose-cli)**

> [!IMPORTANT]
> **Prompt Injection Risk:** When AI analyzes user-controlled input (git diffs, code comments, commit messages), malicious actors can embed instructions to manipulate output. This applies to ANY AI tool, not just Goose or this action.
>
> For production use, see [Security Patterns](#security-patterns) below for three defensive tiers (tool output analysis, manual approval, trusted-only execution).

## Features

- **Caching**: Automatically caches the Goose binary for faster subsequent runs
- **Version Pinning**: Install specific Goose versions for reproducible builds
- **Lightweight**: Composite action with no external dependencies

## Usage

```yaml
# Recommended: Get latest v1.x updates automatically
- uses: clouatre-labs/setup-goose-action@v1

# Pin to exact version (SHA recommended for supply chain integrity)
- uses: clouatre-labs/setup-goose-action@35f35c3a8f08aa333486693114938ec643bf8310  # v1.0.7

# Custom Goose version
- uses: clouatre-labs/setup-goose-action@v1
  with:
    version: '1.28.0'
```

**Current default Goose version:** See [`action.yml`](action.yml#L9)

## Prerequisites

1. **Get an API key** from your chosen provider: [Supported Providers](https://block.github.io/goose/docs/getting-started/providers/)

2. **Add it as a repository secret:**
   - Go to **Settings > Secrets and variables > Actions**
   - Click **New repository secret**
   - Name it (e.g., `GEMINI_API_KEY`, `OPENAI_API_KEY`, `ANTHROPIC_API_KEY`)

3. **Configure in your workflow** by mapping your secret to Goose's expected environment variable (see [Security Patterns](#security-patterns) below)

## Quick Start: Tier 1 (Maximum Security)

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

## Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `version` | Goose version to install | No | See [`action.yml`](action.yml#L9) |

## Outputs

| Output | Description |
|--------|-------------|
| `goose-version` | Installed Goose version |
| `goose-path` | Path to Goose binary directory |

## Security Patterns

This action supports three security tiers for AI-augmented CI/CD:

- **Tier 1 (Maximum Security)**: AI analyzes only tool output (JSON), never raw code. [See workflow](examples/tier1-maximum-security.yml)
- **Tier 2**: AI sees file stats, requires manual approval. [See workflow](examples/tier2-moderate-security.yml)
- **Tier 3**: Full diff analysis, trusted teams only. [See workflow](examples/tier3-full-context.yml)

**Safe Pattern:** AI analyzes tool output (ruff, trivy, semgrep), not raw code.

**Unsafe Pattern:** AI analyzes git diffs directly, which is vulnerable to prompt injection.

Read the full explanation: [AI-Augmented CI/CD blog post](https://clouatre.ca/posts/ai-augmented-cicd)

See [SECURITY.md](SECURITY.md) for reporting vulnerabilities.

## Supported Platforms

| OS | Architecture | Status |
|----|--------------|--------|
| Ubuntu | x64 | Supported |
| Ubuntu | arm64 | Supported |
| macOS | N/A | Not supported |
| Windows | N/A | Not supported |

> **Note:** This action only supports Linux runners. macOS runners have a 10x billing multiplier on GitHub Actions, and Goose executes prompts and tool calls with nothing platform-specific.

## How It Works

1. Checks cache for Goose binary matching the specified version and platform
2. If cache miss, downloads Goose binary from official GitHub releases
3. Extracts binary to `~/.local/bin/goose`
4. Adds binary location to `$GITHUB_PATH`
5. Verifies installation with `goose --version`

## Troubleshooting

### Binary not found after installation

Ensure you're using the action before attempting to run `goose`:

```yaml
- uses: clouatre-labs/setup-goose-action@v1
- run: goose --version  # This will work
```

### Unsupported version

Check available versions at [Goose Releases](https://github.com/block/goose/releases). Ensure the version exists and has pre-built binaries.

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md) for the full contribution guide, including commit signing requirements, coding standards, and the PR process.

## License

MIT. See [LICENSE](LICENSE).

## Related

- [AI-Augmented CI/CD](https://clouatre.ca/posts/ai-augmented-cicd/): 3-tier security model for AI code review in CI/CD pipelines
- [Goose](https://github.com/block/goose): Official Goose repository
- [Goose Documentation](https://block.github.io/goose/)
- [GitHub Actions Documentation](https://docs.github.com/en/actions)
- [Setup Kiro Action](https://github.com/clouatre-labs/setup-kiro-action): Similar action for Kiro CLI (AWS-native, SIGV4 auth)
- [Setup Q CLI Action](https://github.com/clouatre-labs/setup-q-cli-action): Similar action for Amazon Q Developer CLI

## Acknowledgments

Built by [clouatre-labs](https://github.com/clouatre-labs) for the Goose community. Not officially affiliated with Block or the Goose project.
