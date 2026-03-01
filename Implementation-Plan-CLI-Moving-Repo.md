# Implementation Plan: Moving AgentEval.Cli to Its Own Repository

**Date:** March 1, 2026  
**Status:** Planning  
**Target Repo:** `https://github.com/AgentEvalHQ/AgentEval.Cli`  
**Source Repo:** `https://github.com/AgentEvalHQ/AgentEval` (monorepo)

---

## Table of Contents

1. [Overview](#1-overview)
2. [Current State Analysis](#2-current-state-analysis)
3. [Phase 1 — Copy Source Files to New Repo](#3-phase-1--copy-source-files-to-new-repo)
4. [Phase 2 — Extract CLI Tests to Dedicated Project](#4-phase-2--extract-cli-tests-to-dedicated-project)
5. [Phase 3 — Copy Infrastructure Files](#5-phase-3--copy-infrastructure-files)
6. [Phase 4 — Manual Adjustments (Post-Copy)](#6-phase-4--manual-adjustments-post-copy)
7. [Phase 5 — Clean Up the Monorepo](#7-phase-5--clean-up-the-monorepo)
8. [Target Repository Structure](#8-target-repository-structure)
9. [Dependency Map](#9-dependency-map)
10. [Checklist](#10-checklist)

---

## 1. Overview

The `AgentEval.Cli` project currently lives inside the AgentEval monorepo at `src/AgentEval.Cli/`. It builds against local `ProjectReference`s to the 5 library sub-projects. Its tests are colocated in the shared `tests/AgentEval.Tests/Cli/` folder alongside 20+ other test folders.

**Goal:** Move the CLI project and its tests into the standalone `AgentEvalHQ/AgentEval.Cli` repository, where the CLI builds against the **AgentEval NuGet package** instead of project references.

**Key constraint:** The new repo must be self-contained — `dotnet build` and `dotnet test` must work independently of the monorepo.

---

## 2. Current State Analysis

### 2.1 CLI Source Files (9 files)

All located under `src/AgentEval.Cli/`:

| File | Namespace | Purpose |
|------|-----------|---------|
| `Program.cs` | `AgentEval.Cli` | CLI entry point, System.CommandLine root command |
| `ExitCodes.cs` | `AgentEval.Cli` | Constants: Success=0, TestFailure=1, UsageError=2, RuntimeError=3 |
| `Commands/EvalCommand.cs` | `AgentEval.Cli.Commands` | Main `eval` command — runs evaluations, stochastic tests, exports |
| `Commands/InitCommand.cs` | `AgentEval.Cli.Commands` | `init` command — scaffolds config files |
| `Commands/ListCommand.cs` | `AgentEval.Cli.Commands` | `list` command — lists available metrics, attacks |
| `Commands/RedTeamCommand.cs` | `AgentEval.Cli.Commands` | `redteam` command — security scanning |
| `Infrastructure/EndpointFactory.cs` | `AgentEval.Cli.Infrastructure` | Creates `IChatClient` from CLI flags (Azure/OpenAI/Ollama) |
| `Infrastructure/ExportHandler.cs` | `AgentEval.Cli.Infrastructure` | Export orchestration (formats, directories) |
| `Output/ConsoleReporter.cs` | `AgentEval.Cli.Output` | Rich console output for evaluation results |

### 2.2 CLI Test Files (7 files)

All located under `tests/AgentEval.Tests/Cli/`:

| File | What It Tests | Core Library Types Used |
|------|---------------|------------------------|
| `EvalCommandTests.cs` | Eval command parsing & execution | `EvaluationReport`, `TestResultSummary`, `IChatClient` |
| `InitCommandTests.cs` | Init command scaffold output | None (self-contained) |
| `ListCommandTests.cs` | List command output | None (captures stderr) |
| `MetricSelectionTests.cs` | Metric flag parsing | `EvaluationOptions` |
| `ProgramTests.cs` | Root command wiring | None |
| `RedTeamCommandTests.cs` | Red team command & exporters | `Attack`, `Intensity`, report exporters |
| `StochasticFlagTests.cs` | --runs flag parsing | None |

**Key finding:** The CLI tests are **completely self-contained** within the `Cli/` subfolder. They do NOT use any shared test helpers from `TestHelpers/`. They reference AgentEval core types only for model/enum types (not test infrastructure).

### 2.3 Current Project References (monorepo)

The CLI csproj currently has 5 `ProjectReference`s:

```xml
<ProjectReference Include="../AgentEval.Abstractions/AgentEval.Abstractions.csproj" />
<ProjectReference Include="../AgentEval.Core/AgentEval.Core.csproj" />
<ProjectReference Include="../AgentEval.DataLoaders/AgentEval.DataLoaders.csproj" />
<ProjectReference Include="../AgentEval.MAF/AgentEval.MAF.csproj" />
<ProjectReference Include="../AgentEval.RedTeam/AgentEval.RedTeam.csproj" />
```

The test project conditionally references the CLI:
```xml
<ItemGroup Condition="'$(TargetFramework)' != 'net8.0'">
  <ProjectReference Include="..\..\src\AgentEval.Cli\AgentEval.Cli.csproj" />
</ItemGroup>
```

### 2.4 CLI Target Frameworks

- **CLI project:** `net9.0;net10.0` (no net8.0 — System.CommandLine 2.x needs 9.0+)
- **Tests:** Currently `net8.0;net9.0;net10.0` (CLI tests only run on net9.0+ due to conditional ref)

---

## 3. Phase 1 — Copy Source Files to New Repo

Copy the entire `src/AgentEval.Cli/` directory into the new repository.

### Files to Copy

```
src/AgentEval.Cli/
├── AgentEval.Cli.csproj
├── Program.cs
├── ExitCodes.cs
├── Commands/
│   ├── EvalCommand.cs
│   ├── InitCommand.cs
│   ├── ListCommand.cs
│   └── RedTeamCommand.cs
├── Infrastructure/
│   ├── EndpointFactory.cs
│   └── ExportHandler.cs
└── Output/
    └── ConsoleReporter.cs
```

### Destination in New Repo

```
AgentEval.Cli/           ← repo root
└── src/
    └── AgentEval.Cli/
        ├── AgentEval.Cli.csproj
        ├── Program.cs
        ├── ExitCodes.cs
        ├── Commands/
        ├── Infrastructure/
        └── Output/
```

### Copy Command

```powershell
# From monorepo root
Copy-Item -Recurse "src/AgentEval.Cli/*" "<new-repo-path>/src/AgentEval.Cli/" `
    -Exclude "bin","obj"
```

---

## 4. Phase 2 — Extract CLI Tests to Dedicated Project

The 7 CLI test files live in the shared `tests/AgentEval.Tests/Cli/` folder. They need their **own test project** in the new repo.

### 4.1 Files to Copy

```
tests/AgentEval.Tests/Cli/
├── EvalCommandTests.cs
├── InitCommandTests.cs
├── ListCommandTests.cs
├── MetricSelectionTests.cs
├── ProgramTests.cs
├── RedTeamCommandTests.cs
└── StochasticFlagTests.cs
```

### 4.2 Destination in New Repo

```
AgentEval.Cli/
└── tests/
    └── AgentEval.Cli.Tests/
        ├── AgentEval.Cli.Tests.csproj   ← NEW (create this)
        ├── EvalCommandTests.cs
        ├── InitCommandTests.cs
        ├── ListCommandTests.cs
        ├── MetricSelectionTests.cs
        ├── ProgramTests.cs
        ├── RedTeamCommandTests.cs
        └── StochasticFlagTests.cs
```

### 4.3 New Test Project File to Create

```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <TargetFrameworks>net9.0;net10.0</TargetFrameworks>
    <IsPackable>false</IsPackable>
    <IsTestProject>true</IsTestProject>
    <RootNamespace>AgentEval.Tests.Cli</RootNamespace>
  </PropertyGroup>

  <ItemGroup>
    <PackageReference Include="Microsoft.NET.Test.Sdk" />
    <PackageReference Include="xunit" />
    <PackageReference Include="xunit.runner.visualstudio">
      <IncludeAssets>runtime; build; native; contentfiles; analyzers; buildtransitive</IncludeAssets>
      <PrivateAssets>all</PrivateAssets>
    </PackageReference>
    <PackageReference Include="coverlet.collector">
      <IncludeAssets>runtime; build; native; contentfiles; analyzers; buildtransitive</IncludeAssets>
      <PrivateAssets>all</PrivateAssets>
    </PackageReference>
  </ItemGroup>

  <ItemGroup>
    <ProjectReference Include="..\..\src\AgentEval.Cli\AgentEval.Cli.csproj" />
  </ItemGroup>

  <!-- AgentEval core types used by some tests (models, enums) -->
  <ItemGroup>
    <PackageReference Include="AgentEval" />
  </ItemGroup>

</Project>
```

**Notes:**
- `RootNamespace` is `AgentEval.Tests.Cli` to match existing test namespaces
- No `net8.0` — the CLI itself doesn't target it
- References the CLI via `ProjectReference` (local) and AgentEval via `PackageReference` (NuGet)

### 4.4 Test Dependencies on Core AgentEval Types

These are the AgentEval core types the tests reference (they'll come via the AgentEval NuGet package):

| Test File | Core Types Used |
|-----------|----------------|
| `EvalCommandTests.cs` | `EvaluationReport`, `TestResultSummary`, `IChatClient` |
| `MetricSelectionTests.cs` | `EvaluationOptions` |
| `RedTeamCommandTests.cs` | `Attack`, `Intensity`, `JsonReportExporter`, `SarifReportExporter`, `MarkdownReportExporter`, `JUnitReportExporter` |

All other test files (InitCommand, ListCommand, Program, StochasticFlag) are fully self-contained.

### 4.5 Copy Command

```powershell
Copy-Item -Recurse "tests/AgentEval.Tests/Cli/*" "<new-repo-path>/tests/AgentEval.Cli.Tests/"
```

---

## 5. Phase 3 — Copy Infrastructure Files

These files from the monorepo root need to be copied to the new repo root. They'll be adjusted in Phase 4.

### 5.1 Files to Copy (Copy → Adjust Later)

| Source File | Copy As | Needs Adjustment? |
|-------------|---------|-------------------|
| `global.json` | `global.json` | ✅ Yes — may simplify SDK version |
| `Directory.Build.props` | `Directory.Build.props` | ✅ Yes — update repo URL, remove monorepo-specific settings |
| `Directory.Packages.props` | `Directory.Packages.props` | ✅ Yes — strip to CLI-relevant packages only |
| `.gitignore` | `.gitignore` | ⚠️ Review — should be fine as-is for .NET |
| `.editorconfig` | `.editorconfig` | ✅ Copy as-is |
| `LICENSE` | `LICENSE` | ✅ Copy as-is (MIT) |

### 5.2 Files to Create Fresh (Do NOT Copy)

| File | Reason |
|------|--------|
| `AgentEval.Cli.sln` | New solution with only CLI + CLI.Tests projects |
| `README.md` | CLI-specific README (already in the new repo from creation) |
| `CONTRIBUTING.md` | Can reference main repo or create CLI-specific version |
| `SECURITY.md` | Can reference main repo's security policy |
| `CODE_OF_CONDUCT.md` | Copy or reference main repo |
| `.github/workflows/ci.yml` | New CI workflow (CLI-only, simpler) |
| `.github/ISSUE_TEMPLATE/` | Create or reference main repo |
| `.github/PULL_REQUEST_TEMPLATE.md` | Create simplified version |
| `AGENTS.md` | CLI-specific agent instructions |

### 5.3 Files to Optionally Copy

| Source File | Purpose | Copy? |
|-------------|---------|-------|
| `assets/AgentEvalNugetLogoAE.png` | NuGet package icon | ✅ Yes — needed for `PackageIcon` in Directory.Build.props |
| `.github/copilot-instructions.md` | Copilot context | ⚠️ Create CLI-specific version |

### 5.4 Copy Commands

```powershell
$src = "C:\git\joslat\AgentEval"
$dst = "<new-repo-path>"

# Infrastructure files
Copy-Item "$src\global.json"              "$dst\global.json"
Copy-Item "$src\Directory.Build.props"    "$dst\Directory.Build.props"
Copy-Item "$src\Directory.Packages.props" "$dst\Directory.Packages.props"
Copy-Item "$src\.gitignore"               "$dst\.gitignore"
Copy-Item "$src\.editorconfig"            "$dst\.editorconfig"
Copy-Item "$src\LICENSE"                  "$dst\LICENSE"

# Assets
New-Item -ItemType Directory -Path "$dst\assets" -Force
Copy-Item "$src\assets\AgentEvalNugetLogoAE.png" "$dst\assets\"

# Community files
Copy-Item "$src\CODE_OF_CONDUCT.md"       "$dst\CODE_OF_CONDUCT.md"
```

---

## 6. Phase 4 — Manual Adjustments (Post-Copy)

> **⚠️ These steps are done MANUALLY after the copy. The CLI must build against the AgentEval NuGet package, not project references.**

### 6.1 Convert ProjectReferences → PackageReference

**File:** `src/AgentEval.Cli/AgentEval.Cli.csproj`

**Remove** the 5 `ProjectReference` entries:
```xml
<!-- REMOVE ALL OF THESE -->
<ProjectReference Include="../AgentEval.Abstractions/AgentEval.Abstractions.csproj" />
<ProjectReference Include="../AgentEval.Core/AgentEval.Core.csproj" />
<ProjectReference Include="../AgentEval.DataLoaders/AgentEval.DataLoaders.csproj" />
<ProjectReference Include="../AgentEval.MAF/AgentEval.MAF.csproj" />
<ProjectReference Include="../AgentEval.RedTeam/AgentEval.RedTeam.csproj" />
```

**Replace** with a single NuGet reference:
```xml
<ItemGroup>
  <PackageReference Include="AgentEval" />
</ItemGroup>
```

The AgentEval umbrella NuGet package contains all 5 library DLLs, so one reference covers everything.

Also **remove** `InternalsVisibleTo` (tests will be in a separate project with their own reference):
```xml
<!-- REMOVE — no longer needed in standalone repo -->
<InternalsVisibleTo Include="AgentEval.Tests" />
```

If CLI tests need access to internals, add the new test project name instead:
```xml
<InternalsVisibleTo Include="AgentEval.Cli.Tests" />
```

### 6.2 Adjust Directory.Build.props

**File:** `Directory.Build.props`

Update these fields:
```xml
<RepositoryUrl>https://github.com/AgentEvalHQ/AgentEval.Cli</RepositoryUrl>
<PackageProjectUrl>https://github.com/AgentEvalHQ/AgentEval.Cli</PackageProjectUrl>
```

Keep everything else (LangVersion, Nullable, Authors, License, etc.).

### 6.3 Trim Directory.Packages.props

**File:** `Directory.Packages.props`

Strip down to only the packages the CLI and its tests actually need:

```xml
<Project>
  <PropertyGroup>
    <ManagePackageVersionsCentrally>true</ManagePackageVersionsCentrally>
  </PropertyGroup>

  <ItemGroup>
    <!-- AgentEval (the core NuGet package) -->
    <PackageVersion Include="AgentEval" Version="0.6.0-beta" />

    <!-- CLI dependencies -->
    <PackageVersion Include="System.CommandLine" Version="2.0.3" />
    <PackageVersion Include="Microsoft.Extensions.AI.OpenAI" Version="10.3.0" />
    <PackageVersion Include="Azure.AI.OpenAI" Version="2.8.0-beta.1" />

    <!-- Testing -->
    <PackageVersion Include="Microsoft.NET.Test.Sdk" Version="18.3.0" />
    <PackageVersion Include="xunit" Version="2.9.3" />
    <PackageVersion Include="xunit.runner.visualstudio" Version="2.8.2" />
    <PackageVersion Include="coverlet.collector" Version="8.0.0" />
  </ItemGroup>
</Project>
```

Remove all packages only used by the monorepo (MAF agents, YamlDotNet, PdfSharp, Verify.Xunit, etc.).

### 6.4 Create the Solution File

```powershell
cd <new-repo-path>
dotnet new sln -n AgentEval.Cli
dotnet sln add src/AgentEval.Cli/AgentEval.Cli.csproj
dotnet sln add tests/AgentEval.Cli.Tests/AgentEval.Cli.Tests.csproj
```

### 6.5 Create CI Workflow

**File:** `.github/workflows/ci.yml`

```yaml
name: CI

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

env:
  DOTNET_SKIP_FIRST_TIME_EXPERIENCE: true
  DOTNET_CLI_TELEMETRY_OPTOUT: true
  DOTNET_NOLOGO: true

jobs:
  build:
    runs-on: ubuntu-latest
    timeout-minutes: 10

    strategy:
      fail-fast: false
      matrix:
        dotnet-version: [ '9.0.x', '10.0.x' ]

    steps:
    - uses: actions/checkout@v4

    - name: Setup .NET ${{ matrix.dotnet-version }}
      uses: actions/setup-dotnet@v4
      with:
        dotnet-version: ${{ matrix.dotnet-version }}
        dotnet-quality: ${{ matrix.dotnet-version == '10.0.x' && 'preview' || '' }}

    - name: Restore
      run: dotnet restore

    - name: Build
      run: dotnet build --configuration Release --no-restore

    - name: Test
      run: dotnet test --configuration Release --no-build --verbosity normal

  build-windows:
    runs-on: windows-latest
    timeout-minutes: 10

    strategy:
      fail-fast: false
      matrix:
        dotnet-version: [ '9.0.x', '10.0.x' ]

    steps:
    - uses: actions/checkout@v4

    - name: Setup .NET ${{ matrix.dotnet-version }}
      uses: actions/setup-dotnet@v4
      with:
        dotnet-version: ${{ matrix.dotnet-version }}
        dotnet-quality: ${{ matrix.dotnet-version == '10.0.x' && 'preview' || '' }}

    - name: Restore
      run: dotnet restore

    - name: Build
      run: dotnet build --configuration Release --no-restore

    - name: Test
      run: dotnet test --configuration Release --no-build --verbosity normal
```

### 6.6 Verify Build & Tests Pass

```powershell
cd <new-repo-path>
dotnet restore
dotnet build
dotnet test
dotnet pack src/AgentEval.Cli -c Release
```

All 7 test files must pass. The packed `.nupkg` must contain the `agenteval` tool.

### 6.7 Update README

The new repo README should include:
- Installation: `dotnet tool install -g AgentEval.Cli`
- Quick start usage examples
- Link to main AgentEval repo for Discussions
- Link to `agenteval.dev/cli` for docs
- Badge: NuGet version, CI status, license

---

## 7. Phase 5 — Clean Up the Monorepo

After the CLI is working independently in its new repo:

### 7.1 Remove CLI Source from Monorepo

```powershell
# In the AgentEval monorepo
Remove-Item -Recurse "src/AgentEval.Cli"
```

### 7.2 Remove CLI Tests from Monorepo

```powershell
Remove-Item -Recurse "tests/AgentEval.Tests/Cli"
```

### 7.3 Remove CLI from Solution

```powershell
dotnet sln AgentEval.sln remove src/AgentEval.Cli/AgentEval.Cli.csproj
```

### 7.4 Update Test Project

Remove the conditional CLI reference from `tests/AgentEval.Tests/AgentEval.Tests.csproj`:

```xml
<!-- REMOVE THIS BLOCK -->
<ItemGroup Condition="'$(TargetFramework)' != 'net8.0'">
  <ProjectReference Include="..\..\src\AgentEval.Cli\AgentEval.Cli.csproj" />
</ItemGroup>
```

### 7.5 Verify Monorepo Still Builds

```powershell
dotnet build
dotnet test
```

Confirm: 0 CLI-related tests, no broken references.

---

## 8. Target Repository Structure

```
AgentEval.Cli/                          ← repo root
├── AgentEval.Cli.sln
├── global.json
├── Directory.Build.props
├── Directory.Packages.props
├── .gitignore
├── .editorconfig
├── LICENSE
├── CODE_OF_CONDUCT.md
├── README.md
├── AGENTS.md                           ← CLI-specific agent instructions
├── assets/
│   └── AgentEvalNugetLogoAE.png
├── .github/
│   ├── workflows/
│   │   └── ci.yml
│   ├── ISSUE_TEMPLATE/
│   │   ├── bug_report.md
│   │   └── feature_request.md
│   └── PULL_REQUEST_TEMPLATE.md
├── src/
│   └── AgentEval.Cli/
│       ├── AgentEval.Cli.csproj
│       ├── Program.cs
│       ├── ExitCodes.cs
│       ├── Commands/
│       │   ├── EvalCommand.cs
│       │   ├── InitCommand.cs
│       │   ├── ListCommand.cs
│       │   └── RedTeamCommand.cs
│       ├── Infrastructure/
│       │   ├── EndpointFactory.cs
│       │   └── ExportHandler.cs
│       └── Output/
│           └── ConsoleReporter.cs
└── tests/
    └── AgentEval.Cli.Tests/
        ├── AgentEval.Cli.Tests.csproj
        ├── EvalCommandTests.cs
        ├── InitCommandTests.cs
        ├── ListCommandTests.cs
        ├── MetricSelectionTests.cs
        ├── ProgramTests.cs
        ├── RedTeamCommandTests.cs
        └── StochasticFlagTests.cs
```

---

## 9. Dependency Map

### 9.1 CLI Source → AgentEval Core (NuGet) Dependencies

| CLI File | AgentEval Namespaces Used | Key Types |
|----------|--------------------------|-----------|
| `EvalCommand.cs` | `AgentEval.Comparison`, `AgentEval.Core`, `AgentEval.DataLoaders`, `AgentEval.Exporters`, `AgentEval.MAF`, `AgentEval.Models`, `AgentEval.Output` | `StochasticRunner`, `MAFEvaluationHarness`, `EvaluationOptions`, `DatasetLoaderFactory`, `EvaluationReport`, `ExportFormat` |
| `ListCommand.cs` | `AgentEval.RedTeam` | `Attack.All`, `Attack.AvailableNames` |
| `RedTeamCommand.cs` | `AgentEval.Core`, `AgentEval.RedTeam`, `AgentEval.RedTeam.Reporting` | `RedTeamRunner`, `Attack`, `Intensity`, `ScanOptions` |
| `ExportHandler.cs` | `AgentEval.Core`, `AgentEval.Exporters`, `AgentEval.Models` | `ExportFormat`, `ResultExporterFactory`, `DirectoryExporter`, `EvaluationReport` |
| `ConsoleReporter.cs` | `AgentEval.Models` | `TestSummary` |
| `InitCommand.cs` | — | None (self-contained) |
| `EndpointFactory.cs` | — | None (Azure/OpenAI only) |
| `Program.cs` | — | None (System.CommandLine only) |
| `ExitCodes.cs` | — | None (constants only) |

### 9.2 External NuGet Dependencies

| Package | Used By |
|---------|---------|
| `AgentEval` (umbrella) | All core functionality |
| `System.CommandLine` | Program.cs, all Commands |
| `Microsoft.Extensions.AI.OpenAI` | EndpointFactory.cs |
| `Azure.AI.OpenAI` | EndpointFactory.cs |

### 9.3 Test → Core Dependencies

| Test File | AgentEval Core Types Needed (via NuGet) |
|-----------|----------------------------------------|
| `EvalCommandTests.cs` | `EvaluationReport`, `TestResultSummary`, `IChatClient` |
| `MetricSelectionTests.cs` | `EvaluationOptions` |
| `RedTeamCommandTests.cs` | `Attack`, `Intensity`, report exporters |
| Others (4 files) | None — fully self-contained |

---

## 10. Checklist

### Phase 1: Copy Source (Scripted)
- [ ] Create `src/AgentEval.Cli/` in new repo
- [ ] Copy all 9 source files (exclude bin/obj)
- [ ] Verify files copied correctly

### Phase 2: Extract Tests (Scripted + Create)
- [ ] Create `tests/AgentEval.Cli.Tests/` in new repo
- [ ] Copy all 7 test files from `tests/AgentEval.Tests/Cli/`
- [ ] Create `AgentEval.Cli.Tests.csproj` (new file)

### Phase 3: Copy Infrastructure (Scripted)
- [ ] Copy `global.json`
- [ ] Copy `Directory.Build.props`
- [ ] Copy `Directory.Packages.props`
- [ ] Copy `.gitignore`
- [ ] Copy `.editorconfig`
- [ ] Copy `LICENSE`
- [ ] Copy `CODE_OF_CONDUCT.md`
- [ ] Copy `assets/AgentEvalNugetLogoAE.png`

### Phase 4: Manual Adjustments ⚠️
- [ ] Convert CLI csproj: ProjectReferences → `PackageReference Include="AgentEval"`
- [ ] Update/remove `InternalsVisibleTo`
- [ ] Update `Directory.Build.props` repo URLs
- [ ] Trim `Directory.Packages.props` to CLI-only packages
- [ ] Create `AgentEval.Cli.sln` solution file
- [ ] Create `.github/workflows/ci.yml`
- [ ] Run `dotnet build` — verify success
- [ ] Run `dotnet test` — verify all 7 tests pass
- [ ] Run `dotnet pack` — verify `.nupkg` is correct
- [ ] Update `README.md` with CLI-specific content

### Phase 5: Clean Up Monorepo
- [ ] Remove `src/AgentEval.Cli/` from monorepo
- [ ] Remove `tests/AgentEval.Tests/Cli/` from monorepo
- [ ] Remove CLI from `AgentEval.sln`
- [ ] Remove CLI conditional reference from `AgentEval.Tests.csproj`
- [ ] Run `dotnet build` on monorepo — verify success
- [ ] Run `dotnet test` on monorepo — verify no broken references

### Final Verification
- [ ] New repo: `dotnet build` ✅
- [ ] New repo: `dotnet test` ✅ (all 7 tests)
- [ ] New repo: `dotnet pack` ✅
- [ ] Monorepo: `dotnet build` ✅
- [ ] Monorepo: `dotnet test` ✅ (no CLI tests)
- [ ] CI pipeline green on both repos

---

*Last updated: March 1, 2026*
