![AgentEval CLI](https://raw.githubusercontent.com/AgentEvalHQ/AgentEval.Cli/main/assets/AgentEvalCli.png)

# AgentEval CLI

[![NuGet](https://img.shields.io/nuget/vpre/AgentEval.Cli.svg)](https://www.nuget.org/packages/AgentEval.Cli)
[![CI](https://github.com/AgentEvalHQ/AgentEval.Cli/actions/workflows/ci.yml/badge.svg)](https://github.com/AgentEvalHQ/AgentEval.Cli/actions/workflows/ci.yml)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://github.com/AgentEvalHQ/AgentEval.Cli/blob/main/LICENSE)
![MAF 1.0.0-rc3](https://img.shields.io/badge/MAF-1.0.0--rc3-blueviolet)
![.NET 9.0 | 10.0](https://img.shields.io/badge/.NET-9.0%20|%2010.0-512BD4)

Command-line interface for [AgentEval](https://github.com/AgentEvalHQ/AgentEval) — evaluate any OpenAI-compatible AI agent from the terminal.

## Installation

```bash
dotnet tool install -g AgentEval.Cli --prerelease
```

### Compatibility

| AgentEval CLI | AgentEval | MAF | .NET |
|---------------|-----------|-----|------|
| 0.2.0-alpha | 0.6.0-beta | 1.0.0-rc3 | 9.0, 10.0 |
| 0.1.0-alpha | 0.5.3-beta | 1.0.0-rc2 | 9.0, 10.0 |

## Quick Start

### Initialize a test dataset

```bash
agenteval init
agenteval init -o my-tests.yaml
agenteval init --format json
```

### Run evaluations

```bash
# Against Azure OpenAI
agenteval eval --azure --endpoint https://myresource.openai.azure.com/ --deployment-name gpt-4o --dataset agenteval.yaml

# Against OpenAI directly
agenteval eval --endpoint https://api.openai.com/v1 --model gpt-4o --dataset agenteval.yaml

# Against a local Ollama model
agenteval eval --endpoint http://localhost:11434/v1 --model llama3 --dataset agenteval.yaml
```

### Stochastic evaluation (multi-run)

```bash
agenteval eval --azure --endpoint https://myresource.openai.azure.com/ --deployment-name gpt-4o --dataset agenteval.yaml --runs 5 --success-threshold 0.9
```

### Export results

```bash
# Single file export
agenteval eval --azure --endpoint https://myresource.openai.azure.com/ --deployment-name gpt-4o --dataset agenteval.yaml --format json -o results.json

# Structured directory export (ADR-002 format)
agenteval eval --azure --endpoint https://myresource.openai.azure.com/ --deployment-name gpt-4o --dataset agenteval.yaml --format directory --output-dir results/
```

### Red team security scanning

```bash
# Run all 9 attack types
agenteval redteam --azure --endpoint https://myresource.openai.azure.com/ --deployment-name gpt-4o --intensity moderate

# Run specific attacks
agenteval redteam --azure --endpoint https://myresource.openai.azure.com/ --deployment-name gpt-4o --attacks PromptInjection,Jailbreak --format sarif
```

### List available metrics and attacks

```bash
agenteval list
agenteval list --type metrics
agenteval list --type attacks
```

## Authentication

AgentEval supports two endpoint modes: **Azure OpenAI** (`--azure`) and **OpenAI-compatible** (`--endpoint`).

### Azure OpenAI (`--azure`)

The `--azure` flag uses `AzureOpenAIClient`. Both `--endpoint` and `--deployment-name` are **required**:

| Setting | Flag | Env var fallback |
|---------|------|------------------|
| Endpoint | `--endpoint` *(required)* | — |
| Deployment | `--deployment-name` *(required)* | — |
| API Key | `--api-key` | `AZURE_OPENAI_API_KEY` |

```bash
# Explicit key
agenteval eval --azure --endpoint https://myresource.openai.azure.com/ --deployment-name gpt-4o --dataset agenteval.yaml --api-key sk-...

# Key from env var
export AZURE_OPENAI_API_KEY=sk-...
agenteval eval --azure --endpoint https://myresource.openai.azure.com/ --deployment-name gpt-4o --dataset agenteval.yaml
```

> **Note:** `--deployment-name` is the name you gave your model deployment in Azure AI Foundry, not the underlying model name.

### OpenAI-compatible (`--endpoint`)

For OpenAI, Ollama, Groq, vLLM, LM Studio, Together.ai, or any OpenAI-compatible API:

```bash
# OpenAI (set OPENAI_API_KEY or use --api-key)
agenteval eval --endpoint https://api.openai.com/v1 --model gpt-4o --dataset agenteval.yaml --api-key sk-...

# Local Ollama (no key needed)
agenteval eval --endpoint http://localhost:11434/v1 --model llama3 --dataset agenteval.yaml
```

## Commands

| Command | Description |
|---------|-------------|
| `eval` | Run evaluations against an AI agent endpoint |
| `init` | Scaffold a sample test dataset file |
| `list` | List available metrics and attack types |
| `redteam` | Run red team security scans |

## Requirements

- .NET 9.0 or 10.0
- An AI agent endpoint (Azure OpenAI, OpenAI, Ollama, or any OpenAI-compatible API)
- Built on [Microsoft.Extensions.AI](https://github.com/dotnet/extensions) (MAF 1.0.0-rc3)

## Documentation

- [AgentEval Documentation](https://agenteval.dev)

## Contributing

Contributions are welcome! Please open an issue or pull request.

For discussions and questions, visit the [AgentEval Discussions](https://github.com/AgentEvalHQ/AgentEval/discussions) on the main repository.

## License

MIT License. See [LICENSE](https://github.com/AgentEvalHQ/AgentEval.Cli/blob/main/LICENSE) for details.
