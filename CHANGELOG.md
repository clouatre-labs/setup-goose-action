# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.0.1] - 2025-11-12

### Changed
- Updated default Goose version from 1.12.1 to 1.14.0
- Simplified README to reduce maintenance overhead by referencing `action.yml` for current default

### Goose v1.14.0 Highlights
- Fix for runaway subagent chains
- Improved CLI argument standardization
- Better error visibility and reduced logging
- New Mistral AI provider support
- GitHub Copilot AI Models support
- Environment variable substitution in auth blocks

See full release notes: https://github.com/block/goose/releases/tag/v1.14.0

## [1.0.0] - 2025-11-06

### Added
- Initial release of setup-goose-action
- Support for Linux and macOS (x64 and arm64)
- Binary caching for faster subsequent runs
- Version pinning support
- Cross-platform compatibility
- Comprehensive documentation and examples
