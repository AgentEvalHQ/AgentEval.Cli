# AgentEval CLI

[![NuGet](https://img.shields.io/nuget/vpre/AgentEval.Cli.svg)](https://www.nuget.org/packages/AgentEval.Cli)
[![CI](https://github.com/AgentEvalHQ/AgentEval.Cli/actions/workflows/ci.yml/badge.svg)](https://github.com/AgentEvalHQ/AgentEval.Cli/actions/workflows/ci.yml)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

Command-line interface for [AgentEval](https://github.com/AgentEvalHQ/AgentEval) — evaluate any OpenAI-compatible AI agent from the terminal.

## Installation

```bash
dotnet tool install -g AgentEval.Cli --prerelease
```

## Quick Start

### Initialize a test dataset

```bash
agenteval init
agenteval init --output my-tests.yaml
agenteval init --format json
```

### Run evaluations

```bash
# Against Azure OpenAI
agenteval eval --azure --model gpt-4o --dataset agenteval.yaml

# Against OpenAI directly
agenteval eval --endpoint https://api.openai.com/v1 --model gpt-4o --dataset agenteval.yaml

# Against a local Ollama model
agenteval eval --endpoint http://localhost:11434/v1 --model llama3 --dataset agenteval.yaml
```

### Stochastic evaluation (multi-run)

```bash
agenteval eval --azure --model gpt-4o --dataset agenteval.yaml --runs 5 --threshold 0.9
```

### Export results

```bash
agenteval eval --azure --model gpt-4o --dataset agenteval.yaml --format json --output results/
```

### Red team security scanning

```bash
agenteval redteam --azure --model gpt-4o --attacks all --intensity medium
agenteval redteam --azure --model gpt-4o --attacks jailbreak,prompt-injection --format sarif
```

### List available metrics and attacks

```bash
agenteval list --metrics
agenteval list --attacks
```

## Commands

| Command | Description |
|---------|-------------|
| `eval` | Run evaluations against an AI agent endpoint |
| `init` | Scaffold a sample test dataset file |
| `list` | List available metrics and attack types |
| `redteam` | Run red team security scans |

## Requirements

- .NET 9.0 or later
- An AI agent endpoint (Azure OpenAI, OpenAI, Ollama, or any OpenAI-compatible API)

## Documentation

- [AgentEval Documentation](https://agenteval.dev)
- [CLI Reference](https://agenteval.dev/cli)
- [Getting Started](https://agenteval.dev/getting-started.html)

## Contributing

Contributions are welcome! Please open an issue or pull request.

For discussions and questions, visit the [AgentEval Discussions](https://github.com/AgentEvalHQ/AgentEval/discussions) on the main repository.

## License

[MIT](LICENSE) — Copyright © 2026 Jose Luis Latorre
