# Next Steps: Release, Distribution & Remaining Tasks

**Date:** March 1, 2026
**Repo:** `https://github.com/AgentEvalHQ/AgentEval.Cli`

---

## 1. Implementation Plan Status — What's Done

### Phase 1–3: Copy Files ✅ Complete

All 9 source files, 7 test files, and infrastructure files are in place.

### Phase 4: Manual Adjustments ✅ Complete

| Task | Status |
|------|--------|
| 6.1 Convert ProjectReferences → PackageReference (`AgentEval 0.5.3-beta`) | ✅ Done |
| 6.2 Adjust Directory.Build.props (repo URLs → AgentEvalHQ/AgentEval.Cli) | ✅ Done |
| 6.3 Trim Directory.Packages.props (20+ → 8 CLI-only packages) | ✅ Done |
| 6.4 Create solution file (`AgentEval.Cli.slnx`) | ✅ Done |
| 6.5 Create CI workflow (`.github/workflows/ci.yml`) | ✅ Done |
| 6.6 Verify Build & Tests Pass (464/464 passed, `dotnet pack` ✅) | ✅ Done |
| 6.7 Update README.md | ✅ Done |

### Phase 5: Clean Up Monorepo ⏳ Pending (separate repo)

This phase happens on the **AgentEval monorepo**, not this repo. Steps:

- [ ] Remove `src/AgentEval.Cli/` from monorepo
- [ ] Remove `tests/AgentEval.Tests/Cli/` from monorepo
- [ ] Remove CLI from `AgentEval.sln`
- [ ] Remove CLI conditional reference from `AgentEval.Tests.csproj`
- [ ] Verify monorepo still builds and tests pass

### Minor Community Files Not Yet Created (Optional)

These were listed in the target structure (Section 8) but are not blockers:

| File | Priority | Notes |
|------|----------|-------|
| `CODE_OF_CONDUCT.md` | Low | Copy from monorepo or reference it |
| `AGENTS.md` | Low | CLI-specific agent instructions |
| `.github/ISSUE_TEMPLATE/bug_report.md` | Low | Create when public |
| `.github/ISSUE_TEMPLATE/feature_request.md` | Low | Create when public |
| `.github/PULL_REQUEST_TEMPLATE.md` | Low | Create when public |
| `CONTRIBUTING.md` | Low | Create or reference monorepo |
| `SECURITY.md` | Low | Reference monorepo |

---

## 2. How to Release AgentEval.Cli

AgentEval.Cli is a **.NET Tool** — a special type of NuGet package that installs as a command-line executable via `dotnet tool install`. The csproj is already configured with `PackAsTool=true` and `ToolCommandName=agenteval`.

### Release Options Analysis

#### Option A: NuGet.org as .NET Tool ⭐ RECOMMENDED (Primary)

**What:** Pack the project as a NuGet package and push to nuget.org. Users install via `dotnet tool install`.

**How users install:**
```bash
dotnet tool install -g AgentEval.Cli              # stable
dotnet tool install -g AgentEval.Cli --prerelease  # pre-release
dotnet tool update -g AgentEval.Cli                # update
```

**Pros:**
- This is the **standard, idiomatic way** to distribute .NET CLI tools — it's what Microsoft designed the tooling for
- Zero friction for .NET developers (they already have `dotnet` CLI)
- Automatic dependency resolution
- Version management built-in (`dotnet tool update`)
- Local tool support (`dotnet tool install AgentEval.Cli` without `-g`)
- Already works — the csproj has `PackAsTool`, `ToolCommandName`, `PackageId` configured
- CI/CD automation is straightforward (one `dotnet nuget push` command)

**Cons:**
- Requires .NET SDK installed (but your target audience — .NET AI developers — already has it)
- Pre-release versions require `--prerelease` flag

**Effort:** Very low — just `dotnet pack` + `dotnet nuget push`

#### Option B: GitHub Releases with Self-Contained Binaries

**What:** Build self-contained executables (no .NET SDK required) for each platform and attach them to GitHub Releases.

**How users install:**
```bash
# Download from GitHub Releases
curl -L https://github.com/AgentEvalHQ/AgentEval.Cli/releases/download/v0.1.0/agenteval-linux-x64 -o agenteval
chmod +x agenteval
```

**Pros:**
- No .NET SDK required at runtime
- Good for CI/CD environments that don't have .NET
- GitHub Releases page provides a nice download UI
- Can provide AOT-compiled native binaries (faster startup)

**Cons:**
- Larger binary sizes (60-80 MB self-contained, or 10-20 MB with AOT + trimming)
- Must build for each RID: `win-x64`, `linux-x64`, `osx-x64`, `osx-arm64`
- More complex CI pipeline
- No automatic update mechanism
- AOT compilation with trimming may break reflection-heavy libraries (AgentEval uses DI, serialization)

**Effort:** Medium — needs multi-RID build matrix, release workflow, testing per platform

#### Option C: Package Managers (winget, Homebrew, Chocolatey, Scoop)

**What:** Publish to platform-specific package managers.

**How users install:**
```bash
winget install AgentEval.Cli          # Windows
brew install agenteval                # macOS
choco install agenteval               # Windows (Chocolatey)
scoop install agenteval               # Windows (Scoop)
```

**Pros:**
- Native discoverability on each platform
- Familiar install experience for non-.NET users

**Cons:**
- High maintenance — each package manager has its own manifest format, review process, and update cadence
- winget requires a separate manifest repo PR to microsoft/winget-pkgs
- Homebrew requires a tap or formula PR
- Audience overlap: your users are .NET developers who prefer `dotnet tool install`
- Depends on Option B (needs self-contained binaries)

**Effort:** High per platform — ongoing maintenance burden

#### Option D: Docker Image

**What:** Publish a Docker image to GitHub Container Registry or Docker Hub.

```bash
docker run ghcr.io/agentevalHQ/agenteval eval --azure --model gpt-4o --dataset tests.yaml
```

**Pros:**
- Great for CI/CD pipelines (GitHub Actions, Azure DevOps)
- Consistent environment

**Cons:**
- Overkill for a CLI tool at this stage
- Docker overhead for a simple command-line invocation
- Poor interactive experience

**Effort:** Low-medium — Dockerfile + GHCR workflow

### Recommendation

**Start with Option A (NuGet.org) — it's the clear winner.**

Reasoning:
1. The project is **already configured** as a .NET Tool (`PackAsTool=true`)
2. Your audience is .NET developers building AI agents — they have the .NET SDK
3. Microsoft designed the `.NET Tool` ecosystem specifically for this use case
4. It requires the least effort and maintenance
5. It matches how AgentEval itself is distributed (NuGet)
6. Version management is built-in

**Add Option B (GitHub Releases) later** when you have demand from non-.NET users or CI environments. Skip C and D for now.

---

## 3. Release Workflow — Step by Step

### 3.1 Manual Release (First Time)

```powershell
# 1. Set the version in the csproj (currently 0.1.0-alpha)
#    Update <Version> in src/AgentEval.Cli/AgentEval.Cli.csproj

# 2. Build and pack
dotnet pack src/AgentEval.Cli -c Release

# 3. Verify the nupkg
dotnet tool install -g AgentEval.Cli --add-source src/AgentEval.Cli/bin/Release --version 0.1.0-alpha

# 4. Test it works
agenteval --help
agenteval list --metrics

# 5. Uninstall test install
dotnet tool uninstall -g AgentEval.Cli

# 6. Push to NuGet.org
dotnet nuget push src/AgentEval.Cli/bin/Release/AgentEval.Cli.0.1.0-alpha.nupkg \
    --api-key <YOUR_NUGET_API_KEY> \
    --source https://api.nuget.org/v3/index.json
```

### 3.2 Automated Release (GitHub Actions)

Create `.github/workflows/release.yml` to automate on git tag push:

```yaml
name: Release

on:
  push:
    tags: [ 'v*' ]

env:
  DOTNET_SKIP_FIRST_TIME_EXPERIENCE: true
  DOTNET_CLI_TELEMETRY_OPTOUT: true
  DOTNET_NOLOGO: true

jobs:
  release:
    runs-on: ubuntu-latest
    timeout-minutes: 15
    permissions:
      contents: write  # for GitHub Release

    steps:
    - uses: actions/checkout@v4

    - name: Setup .NET
      uses: actions/setup-dotnet@v4
      with:
        dotnet-version: '9.0.x'

    - name: Extract version from tag
      id: version
      run: echo "VERSION=${GITHUB_REF_NAME#v}" >> $GITHUB_OUTPUT

    - name: Restore
      run: dotnet restore

    - name: Build
      run: dotnet build --configuration Release --no-restore

    - name: Test
      run: dotnet test --configuration Release --no-build --verbosity normal

    - name: Pack
      run: dotnet pack src/AgentEval.Cli -c Release /p:Version=${{ steps.version.outputs.VERSION }}

    - name: Push to NuGet.org
      run: >
        dotnet nuget push src/AgentEval.Cli/bin/Release/*.nupkg
        --api-key ${{ secrets.NUGET_API_KEY }}
        --source https://api.nuget.org/v3/index.json
        --skip-duplicate

    - name: Create GitHub Release
      uses: softprops/action-gh-release@v2
      with:
        generate_release_notes: true
        files: src/AgentEval.Cli/bin/Release/*.nupkg
```

**Setup required:**
1. Go to https://www.nuget.org/account/apikeys and create an API key scoped to `AgentEval.Cli`
2. Add it as a GitHub repo secret named `NUGET_API_KEY`
3. To release: `git tag v0.1.0-alpha && git push origin v0.1.0-alpha`

### 3.3 Versioning Strategy

| Version Pattern | Meaning | NuGet Behavior |
|----------------|---------|----------------|
| `0.1.0-alpha` | Early pre-release | Requires `--prerelease` to install |
| `0.1.0-beta` | Feature-complete pre-release | Requires `--prerelease` to install |
| `0.1.0-rc.1` | Release candidate | Requires `--prerelease` to install |
| `0.1.0` | Stable release | Default install target |

**Recommendation:** Start with `0.1.0-beta` to match the AgentEval core package's beta status. Move to stable `1.0.0` when AgentEval itself goes stable.

---

## 4. Pre-Release Checklist

Before the first public release:

- [ ] **Decide version number** — suggest `0.1.0-beta` to match AgentEval's beta status
- [ ] **Update `<Version>` in csproj** — currently `0.1.0-alpha`
- [ ] **Add `<PackageReadmeFile>` to csproj** — NuGet warned about missing readme
- [ ] **Create NuGet API key** scoped to `AgentEval.Cli` package
- [ ] **Add `NUGET_API_KEY` secret** to the GitHub repo
- [ ] **Create `.github/workflows/release.yml`** — automated release on tag push
- [ ] **Test the nupkg locally** — install, run commands, uninstall
- [ ] **Push initial commit** to GitHub and verify CI is green
- [ ] **Tag and release** — `git tag v0.1.0-beta && git push origin v0.1.0-beta`
- [ ] **Verify on nuget.org** — package appears, install works

---

## 5. Remaining Work Items (Prioritized)

### High Priority (Do Before First Release)

| # | Task | Details |
|---|------|---------|
| 1 | Add PackageReadmeFile to csproj | NuGet warned about missing readme in the nupkg. Add `<PackageReadmeFile>README.md</PackageReadmeFile>` and include README.md in the pack |
| 2 | Create release workflow | `.github/workflows/release.yml` for automated NuGet publishing on tag |
| 3 | Push to GitHub & verify CI | Ensure the CI workflow runs green on main |
| 4 | First release to NuGet.org | Tag `v0.1.0-beta`, push, verify |

### Medium Priority (Do Soon After)

| # | Task | Details |
|---|------|---------|
| 5 | Clean up monorepo (Phase 5) | Remove CLI source + tests from AgentEvalHQ/AgentEval |
| 6 | Community files | CODE_OF_CONDUCT.md, CONTRIBUTING.md, SECURITY.md |
| 7 | GitHub repo settings | Branch protection rules, issue templates, PR template |
| 8 | Cross-link repos | Add CLI link to main AgentEval README, update agenteval.dev |

### Low Priority (Nice to Have)

| # | Task | Details |
|---|------|---------|
| 9 | GitHub Releases with binaries (Option B) | Self-contained builds for non-.NET users |
| 10 | AGENTS.md | CLI-specific agent instructions for Copilot |
| 11 | Package manager distribution | winget, Homebrew — only if demand exists |
| 12 | Docker image | Only if requested for CI/CD use cases |

---

## 6. Quick Reference — Common Operations

```powershell
# Build
dotnet build AgentEval.Cli.slnx

# Test
dotnet test AgentEval.Cli.slnx

# Pack (Release)
dotnet pack src/AgentEval.Cli -c Release

# Local install for testing
dotnet tool install -g AgentEval.Cli --add-source src/AgentEval.Cli/bin/Release --version <VERSION>

# Uninstall
dotnet tool uninstall -g AgentEval.Cli

# Push to NuGet
dotnet nuget push src/AgentEval.Cli/bin/Release/AgentEval.Cli.<VERSION>.nupkg --api-key <KEY> --source https://api.nuget.org/v3/index.json

# Tag a release
git tag v<VERSION>
git push origin v<VERSION>
```

---

*Last updated: March 1, 2026*
