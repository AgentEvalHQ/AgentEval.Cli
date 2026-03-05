# Changelog

All notable changes to AgentEval CLI will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [0.2.0-alpha] - 2026-03-05

### Changed

- Upgraded AgentEval dependency from 0.5.4-beta to 0.6.0-beta
- Now built on MAF (Microsoft.Extensions.AI) 1.0.0-rc3

[0.2.0-alpha]: https://github.com/AgentEvalHQ/AgentEval.Cli/releases/tag/v0.2.0-alpha

## [0.1.0-alpha] - 2026-03-01

### Added

- Initial release of AgentEval CLI as a standalone .NET Tool
- `eval` command — run evaluations against any OpenAI-compatible AI agent
  - Azure OpenAI, OpenAI, and Ollama endpoint support
  - Stochastic evaluation with `--runs` and `--threshold` flags
  - Export results to JSON, SARIF, Markdown, JUnit formats
  - Metric selection via `--metrics` flag
- `init` command — scaffold sample test dataset files (YAML/JSON)
- `list` command — list available metrics and attack types
- `redteam` command — run red team security scans
  - Multiple attack types and intensity levels
  - SARIF, JSON, Markdown, JUnit export formats
- Rich console output with colored results and progress
- Depends on AgentEval NuGet package 0.5.3-beta (MAF 1.0.0-rc2)

### Infrastructure

- Extracted from [AgentEval monorepo](https://github.com/AgentEvalHQ/AgentEval) into standalone repository
- CI workflow for .NET 9.0 and 10.0 on Ubuntu and Windows
- Automated release workflow — tag `v*` triggers NuGet publish + GitHub Release
- Central package management via `Directory.Packages.props`

[0.1.0-alpha]: https://github.com/AgentEvalHQ/AgentEval.Cli/releases/tag/v0.1.0-alpha
