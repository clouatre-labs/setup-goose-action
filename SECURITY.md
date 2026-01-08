# Security Policy

## Reporting Security Issues

If you discover a security vulnerability in this action, please **do not** open a public issue.

Report privately via:

1. **GitHub Security Advisories** - [Report a vulnerability](https://github.com/clouatre-labs/setup-goose-action/security/advisories/new)
2. **Email** - hugues@linux.com

Please include:
- Description of the vulnerability
- Steps to reproduce
- Potential impact
- Suggested fixes (if available)

## Response Timeline

- **Acknowledgment:** Within 48 hours
- **Initial assessment:** Within 5 business days
- **Fix timeline:** 7-14 days depending on severity

## Supported Versions

| Version | Supported |
|---------|-----------|
| v1.x    | ✅ Yes    |
| < v1.0  | ❌ No     |

## Scope

This policy covers **this GitHub Action only**.

### Out of Scope

- **Goose CLI vulnerabilities** - Report to [block/goose security](https://github.com/block/goose/security)
- **Workflow security patterns** - See [examples/](examples/) for defensive CI/CD architectures
- **Prompt injection in AI workflows** - This is a known risk documented in the README. See [Security Patterns](README.md#security-patterns) for mitigation strategies.

## Security Features

This action:
- Downloads binaries only from official [block/goose releases](https://github.com/block/goose/releases)
- Uses GitHub Actions cache with version-specific keys
- Requires minimal permissions (`contents: read`)
- Does not store or transmit API keys (user-managed via secrets)

## Disclosure Policy

- Please allow reasonable time to address issues before public disclosure
- We will credit you in the fix (unless you prefer anonymity)

Thank you for helping keep this project secure.
