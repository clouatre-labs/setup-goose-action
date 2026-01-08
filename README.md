# Setup Goose Action

[![Test Action](https://github.com/clouatre-labs/setup-goose-action/actions/workflows/test.yml/badge.svg)](https://github.com/clouatre-labs/setup-goose-action/actions/workflows/test.yml)
[![GitHub Marketplace](https://img.shields.io/badge/Marketplace-Setup%20Goose%20CLI-blue?logo=github)](https://github.com/marketplace/actions/setup-goose-cli)
[![Security Policy](https://img.shields.io/badge/Security-Policy-blue?logo=github)](SECURITY.md)
[![Composite Action](https://img.shields.io/badge/Composite-Action-green?logo=github)](https://docs.github.com/en/actions/creating-actions/about-custom-actions#composite-actions)
[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Latest Release](https://img.shields.io/github/v/release/clouatre-labs/setup-goose-action)](https://github.com/clouatre-labs/setup-goose-action/releases/latest)

GitHub Action to install and cache [Goose AI agent](https://github.com/block/goose) for use in workflows.

**Available on the [GitHub Marketplace](https://github.com/marketplace/actions/setup-goose-cli)**

> [!IMPORTANT]
> **Prompt Injection Risk:** When AI analyzes user-controlled input (git diffs, code comments, commit messages), malicious actors can embed instructions to manipulate output. This applies to ANY AI tool, not just Goose or this action.
>
> For production use, see [Security Patterns](#security-patterns) below for three defensive tiers (tool output analysis, manual approval, trusted-only execution).

## Usage

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

3. **Configure in your workflow** - map your secret to Goose's expected environment variable (see [Security Patterns](#security-patterns) below)

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

## Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `version` | Goose version to install | No | See [`action.yml`](action.yml#L9) |

## Outputs

| Output | Description |
|--------|-------------|
| `goose-version` | Installed Goose version |
| `goose-path` | Path to Goose binary directory |
| `cache-hit` | Whether the Goose binary was restored from cache (`'true'` or `'false'`) |

## Examples

### Pin to Specific Version

```yaml
- uses: clouatre-labs/setup-goose-action@v1
  with:
    version: '1.14.0'
```

### Using cache-hit output

Conditionally skip steps based on cache status:

```yaml
- name: Setup Goose
  id: setup-goose
  uses: clouatre-labs/setup-goose-action@v1
  with:
    version: '1.19.1'

- name: Skip expensive operation if cached
  if: steps.setup-goose.outputs.cache-hit == 'false'
  run: |
    # Only run on cache miss (first run or version change)
    echo "Binary was freshly installed, running initialization..."
    goose --version

- name: Use cached binary
  if: steps.setup-goose.outputs.cache-hit == 'true'
  run: |
    # Binary was restored from cache
    echo "Binary restored from cache, skipping initialization"
    goose --version
```

## Security

**Safe Pattern:** AI analyzes tool output (ruff, trivy, semgrep), not raw code.

**Unsafe Pattern:** AI analyzes git diffs directly → vulnerable to prompt injection.

See [SECURITY.md](SECURITY.md) for reporting vulnerabilities.

## Security Patterns

This action supports three security tiers for AI-augmented CI/CD:

- **Tier 1 (Maximum Security)**: AI analyzes only tool output (JSON), never raw code. [See workflow](examples/tier1-maximum-security.yml)
- **Tier 2**: AI sees file stats, requires manual approval. [See workflow](examples/tier2-moderate-security.yml)
- **Tier 3**: Full diff analysis, trusted teams only. [See workflow](examples/tier3-full-context.yml)

Read the full explanation: [AI-Augmented CI/CD blog post](https://clouatre.ca/posts/ai-augmented-cicd)

## Features

- **Caching**: Automatically caches Goose binary for faster subsequent runs
- **Version Pinning**: Install specific Goose versions for reproducible builds
- **Lightweight**: Composite action with no external dependencies

## Supported Platforms

| OS | Architecture | Status |
|----|--------------|--------|
| Ubuntu | x64 | Supported |
| Ubuntu | arm64 | Supported |
| macOS | - | Not supported |
| Windows | - | Not supported |

> **Note:** This action only supports Linux runners. macOS runners have a 10x billing multiplier on GitHub Actions, and there's no practical use case for running Goose on macOS in CI (Goose executes prompts and tool calls, nothing platform-specific).

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
