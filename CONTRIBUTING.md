# Contributing to AgentEval CLI

Thank you for your interest in contributing to AgentEval CLI!

## How to Contribute

### Reporting Bugs

- Use the [Bug Report](https://github.com/AgentEvalHQ/AgentEval.Cli/issues/new?template=bug_report.md) issue template
- Include .NET SDK version (`dotnet --info`), OS, and steps to reproduce

### Suggesting Features

- Use the [Feature Request](https://github.com/AgentEvalHQ/AgentEval.Cli/issues/new?template=feature_request.md) issue template
- For broader AgentEval discussions, visit the [main repo Discussions](https://github.com/AgentEvalHQ/AgentEval/discussions)

### Pull Requests

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/my-feature`)
3. Make your changes
4. Ensure tests pass: `dotnet test`
5. Commit with a clear message
6. Push and open a Pull Request

### Development Setup

```bash
git clone https://github.com/AgentEvalHQ/AgentEval.Cli.git
cd AgentEval.Cli
dotnet restore
dotnet build
dotnet test
```

### Requirements

- .NET 9.0 SDK or later
- All tests must pass
- Follow existing code style (enforced by `.editorconfig`)

## Code of Conduct

This project follows the [Contributor Covenant Code of Conduct](CODE_OF_CONDUCT.md).

## License

By contributing, you agree that your contributions will be licensed under the [MIT License](LICENSE).
