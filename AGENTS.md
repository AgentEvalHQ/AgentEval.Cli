# AgentEval CLI — Agent Instructions

This repository contains the **AgentEval CLI** — a .NET global tool for evaluating AI agents from the command line.

## Repository Structure

- `src/AgentEval.Cli/` — CLI source code (commands, infrastructure, output)
- `tests/AgentEval.Cli.Tests/` — Unit tests (xUnit)
- `AgentEval.Cli.slnx` — Solution file

## Key Facts

- This is a **.NET Tool** (`PackAsTool=true`), installed via `dotnet tool install -g AgentEval.Cli`
- The CLI command name is `agenteval`
- Depends on the **AgentEval NuGet package** (not project references)
- Target frameworks: `net9.0` and `net10.0`
- Central package management via `Directory.Packages.props`
- Tests use xUnit; all tests must pass before merging

## Commands

| Command | File | Purpose |
|---------|------|---------|
| `eval` | `Commands/EvalCommand.cs` | Run evaluations against an AI agent |
| `init` | `Commands/InitCommand.cs` | Scaffold a sample test dataset |
| `list` | `Commands/ListCommand.cs` | List available metrics and attacks |
| `redteam` | `Commands/RedTeamCommand.cs` | Run security scans |

## Build & Test

```bash
dotnet build
dotnet test
dotnet pack src/AgentEval.Cli -c Release
```
