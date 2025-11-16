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

> [!WARNING]
> AI tools can be manipulated via code comments and commit messages. This example analyzes tool output only. See [examples/](examples/) for other patterns.

## Quick Start

```yaml
name: Secure AI Analysis
on: [pull_request]

permissions:
  contents: read

jobs:
  analyze:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v5

      - name: Lint Code
        run: |
          pipx install uv
          uv tool run ruff check --output-format=json . > lint-results.json || true

      - uses: clouatre-labs/setup-goose-action@v1

      - name: AI Analysis
        env:
          GOOGLE_API_KEY: ${{ secrets.GOOGLE_API_KEY }}
        run: |
          echo "Summarize these linting issues:" > prompt.txt
          cat lint-results.json >> prompt.txt
          goose run --instructions prompt.txt --no-session --quiet > analysis.md

      - uses: actions/upload-artifact@v5
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
      - uses: actions/checkout@v5
      
      - name: Run Security Scanner
        run: |
          pipx install uv
          uv tool run ruff check --select S --output-format=json . > security.json || true
      
      - uses: clouatre-labs/setup-goose-action@v1
      
      - name: AI Analysis
        env:
          ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}
        run: |
          echo "Summarize security findings:" > prompt.txt
          cat security.json >> prompt.txt
          goose run --instructions prompt.txt --no-session --quiet > report.md
      
      - uses: actions/upload-artifact@v5
        with:
          name: security-report
          path: report.md
```

### Pin to Specific Version

```yaml
- uses: clouatre-labs/setup-goose-action@v1
  with:
    version: '1.14.0'
```

## Security

**Safe Pattern:** AI analyzes tool output (ruff, trivy, semgrep), not raw code.

**Unsafe Pattern:** AI analyzes git diffs directly → vulnerable to prompt injection.

See [SECURITY.md](SECURITY.md) for reporting vulnerabilities.  
See [examples/](examples/) for different security tiers.

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
