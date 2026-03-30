# Security Assurance Case

This document provides the security assurance case for setup-goose-action.

## What the action does

setup-goose-action is a GitHub Actions composite action that downloads a pre-built binary of the [Goose AI agent](https://github.com/block/goose) from the official GitHub Releases page, caches it using the GitHub Actions cache, and adds it to `$GITHUB_PATH`. The action does not execute Goose, does not handle API keys, and performs no AI operations itself.

## Trust boundaries

| Boundary | Direction | Description |
|----------|-----------|-------------|
| GitHub Releases (HTTPS) | Inbound | Goose binary downloaded from `github.com/block/goose/releases` |
| GitHub Actions cache | Bidirectional | Cached binary stored and restored by `actions/cache` |
| `$GITHUB_PATH` | Outbound | Binary location appended to the runner PATH |
| Caller workflow | Inbound | `version` and `check-latest` inputs supplied by the calling workflow |

The action makes no outbound calls beyond the GitHub Releases download URL and the GitHub API (for `check-latest`). It holds no credentials and writes no persistent state outside the runner environment.

## Attack surface

The primary attack surfaces are:

1. **Supply chain -- upstream binary**: The Goose binary is downloaded from `github.com/block/goose/releases`. This is mitigated by using HTTPS exclusively and pinning to a caller-specified version. The `check-latest` input queries the GitHub API for the latest release tag before downloading.

2. **Prompt injection in caller workflows**: Callers that pass user-controlled content (git diffs, commit messages) to Goose are vulnerable to prompt injection. This is a known risk documented in the README with three defensive tiers (see [Security Patterns](README.md#security-patterns)).

3. **GitHub Actions workflow security**: The action's own workflow is scanned with [zizmor](https://github.com/zizmorcore/zizmor) on every PR. All actions are pinned to SHA digests. No `pull_request_target` triggers are used.

## Common weaknesses countered

| Weakness | Status |
|----------|--------|
| MITM on download | Not applicable -- HTTPS enforced; no plain-HTTP fallback |
| Malicious cached binary | Mitigated -- cache key includes version, OS, and arch; a stale or corrupted cache entry results in a fresh download |
| Secret leakage in logs | Mitigated -- zizmor scans all workflows; no secrets are handled by the action itself |
| Dependency confusion | Not applicable -- no package manager dependencies; binary only |
| Version pinning bypass | Mitigated -- `version` input is validated against semver regex before use |

## Supply chain hardening

- All GitHub Actions used by this action and its CI workflow are pinned to SHA digests (enforced by zizmor)
- Renovate bot creates weekly PRs for digest updates
- The Goose binary is downloaded from the official upstream repository only

## Security review

- **Review date:** 2026-03-30
- **Scope:** Full action.yml, CI workflow, trust boundaries, attack surface (as documented in this file)
- **Conclusion:** No exploitable vulnerabilities identified; residual prompt-injection risk is documented in README with mitigations
- **Reviewer:** Project maintainer (self-review; acceptable for solo projects under OpenSSF criteria)
