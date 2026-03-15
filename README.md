# mc-multiplatform-toolkit

Reusable GitHub Actions workflows for Minecraft multi-platform mods/plugins.

Supports **Bukkit/Spigot/Paper**, **Fabric**, **Forge**, and **NeoForge** from a single project.

---

## Workflows

### `ci.yml` — Continuous Integration

Runs unit tests, builds all platforms in parallel, and runs integration tests (on PRs and manual dispatch).

```yaml
# .github/workflows/ci.yml (in your project)
name: CI
on:
  push:
    branches: [master, develop, 'feature/**', 'fix/**']
  pull_request:
    branches: [master, develop]
  workflow_dispatch:

jobs:
  ci:
    uses: dodoflix/mc-multiplatform-toolkit/.github/workflows/ci.yml@main
    with:
      mod-name: YourModName       # PascalCase display name
      mod-id: yourmodname         # lowercase mod ID (used for log grep)
      java-version: '21'          # optional, default: '21'
      run-bukkit: true            # optional, default: true
      run-fabric: true            # optional, default: true
      run-forge: true             # optional, default: true
      run-neoforge: true          # optional, default: true
    secrets: inherit
```

### `release.yml` — Automated Release & Publishing

Calculates semantic version from commit messages, builds all platforms, runs integration tests, creates a GitHub Release, and publishes to Modrinth (production only).

```yaml
# .github/workflows/release.yml (in your project)
name: Release
on:
  push:
    branches: [master, develop]

permissions:
  contents: write

jobs:
  release:
    uses: dodoflix/mc-multiplatform-toolkit/.github/workflows/release.yml@main
    with:
      mod-name: YourModName
      mod-id: yourmodname
      modrinth-project-id: ${{ vars.MODRINTH_PROJECT_ID }}   # set as repo variable
      bukkit-mc-versions: |
        1.21.11
      fabric-mc-versions: "1.21.11"
      forge-mc-versions: "1.21.11"
      neoforge-mc-versions: "1.21.11"
    secrets: inherit
```

---

## Inputs Reference

### `ci.yml`

| Input | Type | Default | Description |
|-------|------|---------|-------------|
| `mod-name` | string | **required** | Display name (PascalCase) — used in artifact names and log grep |
| `mod-id` | string | **required** | Mod ID (lowercase) — used in log grep patterns |
| `java-version` | string | `'21'` | Java version |
| `run-bukkit` | boolean | `true` | Enable Bukkit platform |
| `run-fabric` | boolean | `true` | Enable Fabric platform |
| `run-forge` | boolean | `true` | Enable Forge platform |
| `run-neoforge` | boolean | `true` | Enable NeoForge platform |

### `release.yml`

All `ci.yml` inputs plus:

| Input | Type | Default | Description |
|-------|------|---------|-------------|
| `modrinth-project-id` | string | `''` | Modrinth project ID/slug (skip publishing if empty) |
| `bukkit-mc-versions` | string | `1.21.11` | Newline-separated MC versions for Modrinth Bukkit |
| `fabric-mc-versions` | string | `1.21.11` | Newline-separated MC versions for Modrinth Fabric |
| `forge-mc-versions` | string | `1.21.11` | Newline-separated MC versions for Modrinth Forge |
| `neoforge-mc-versions` | string | `1.21.11` | Newline-separated MC versions for Modrinth NeoForge |
| `production-branch` | string | `master` | Branch that triggers production releases |
| `develop-branch` | string | `develop` | Branch that receives version syncs after production release |
| `modrinth-description-path` | string | `.modrinth/description.md` | Path to Modrinth description file |

---

## Secrets

Set these in your project's repository secrets:

| Secret | Used by | Description |
|--------|---------|-------------|
| `MODRINTH_TOKEN` | `release.yml` | Modrinth API token for publishing |

`GITHUB_TOKEN` is automatically available and used for creating releases.

---

## Versioning

Versions are calculated automatically from [Conventional Commits](https://www.conventionalcommits.org/):

| Commit prefix | Version bump |
|---------------|--------------|
| `feat:` or `feat(scope):` | Minor (`1.2.0` → `1.3.0`) |
| `fix:` or any other | Patch (`1.2.0` → `1.2.1`) |
| `feat!:` or `BREAKING CHANGE:` footer | Major (`1.2.0` → `2.0.0`) |

Dev builds on non-production branches produce `X.Y.Z-dev.N` (e.g. `2.2.0-dev.271`).

---

## Projects Using This Toolkit

- [DisableVillagerTrade](https://github.com/dodoflix/DisableVillagerTrade)
- [mc-multiplatform-template](https://github.com/dodoflix/mc-multiplatform-template)

---

## Expected Project Structure

Your project must follow this layout for the workflows to work:

```
your-project/
├── gradle/libs.versions.toml   ← must contain minecraft, forge, neoforge version keys
├── gradle.properties           ← must contain modVersion=X.Y.Z
├── gradlew
├── build.gradle.kts
├── settings.gradle.kts
├── common/                     ← shared logic (Gradle subproject)
├── bukkit/
│   ├── gradlew
│   └── build/libs/*.jar        ← produced by ./gradlew build
├── fabric/
│   ├── gradlew
│   └── build/libs/*.jar
├── forge/
│   ├── gradlew
│   └── build/libs/*.jar
└── neoforge/
    ├── gradlew
    └── build/libs/*.jar
```
