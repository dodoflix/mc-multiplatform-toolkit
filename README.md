# mc-multiplatform-toolkit

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

Reusable GitHub Actions [`workflow_call`](https://docs.github.com/en/actions/sharing-automations/reusing-workflows) workflows for Minecraft multi-platform mods and plugins.

Supports **Bukkit/Spigot/Paper**, **Fabric**, **Forge**, and **NeoForge** from a single shared project — maintained in one place so every project benefits from fixes and improvements automatically.

> **Scope note:** The CI and integration test logic is Minecraft-specific (server startup, log parsing, Modrinth publishing). The build and unit-test structure is generic and reusable for any multi-module Gradle project with platform-specific subprojects.

---

## Workflows

### `ci.yml` — Continuous Integration

Runs unit tests, builds all platforms in parallel, and runs integration tests (on PRs and manual dispatch).

**Pipeline:**
```
check-changes → unit-tests → build-bukkit  ─┐
                           → build-fabric  ─┤→ test-* (integration) → summary
                           → build-forge   ─┤
                           → build-neoforge─┘
```

**Caller workflow** (`.github/workflows/ci.yml` in your project):

```yaml
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
      mod-id: yourmodname         # lowercase mod ID (used in log grep)
      java-version: '21'          # optional, default: '21'
      run-bukkit: true            # optional, default: true
      run-fabric: true            # optional, default: true
      run-forge: true             # optional, default: true
      run-neoforge: true          # optional, default: true
    secrets: inherit
```

### `release.yml` — Automated Release & Publishing

Calculates semantic version from commit messages, builds all platforms, runs integration tests, creates a GitHub Release, and publishes to Modrinth (production only).

**Caller workflow** (`.github/workflows/release.yml` in your project):

```yaml
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
        1.20.1
        1.21
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
| `mod-name` | string | **required** | Display name (PascalCase) — used in artifact names |
| `mod-id` | string | **required** | Mod ID (lowercase) — used in log grep patterns |
| `java-version` | string | `'21'` | Java toolchain version |
| `run-bukkit` | boolean | `true` | Enable Bukkit build + integration test |
| `run-fabric` | boolean | `true` | Enable Fabric build + integration test |
| `run-forge` | boolean | `true` | Enable Forge build + integration test |
| `run-neoforge` | boolean | `true` | Enable NeoForge build + integration test |

### `release.yml`

All `ci.yml` inputs, plus:

| Input | Type | Default | Description |
|-------|------|---------|-------------|
| `modrinth-project-id` | string | `''` | Modrinth project ID/slug — publishing skipped if empty |
| `bukkit-mc-versions` | string | `1.21.11` | Newline-separated MC versions for Modrinth Bukkit listing |
| `fabric-mc-versions` | string | `1.21.11` | Space or newline-separated MC versions for Fabric |
| `forge-mc-versions` | string | `1.21.11` | Space or newline-separated MC versions for Forge |
| `neoforge-mc-versions` | string | `1.21.11` | Space or newline-separated MC versions for NeoForge |
| `production-branch` | string | `master` | Branch that triggers production (non-dev) releases |
| `develop-branch` | string | `develop` | Branch that receives `modVersion` sync after production release |
| `modrinth-description-path` | string | `.modrinth/description.md` | Path to Modrinth long-form description file |

---

## Secrets

| Secret / Variable | Where to set | Description |
|-------------------|-------------|-------------|
| `MODRINTH_TOKEN` | Repository **secret** | Modrinth API token |
| `MODRINTH_PROJECT_ID` | Repository **variable** (`vars.*`) | Modrinth project slug or ID |

`GITHUB_TOKEN` is provided automatically by GitHub Actions.

---

## Versioning

Versions are calculated automatically from [Conventional Commits](https://www.conventionalcommits.org/):

| Commit prefix | Version bump |
|---------------|-------------|
| `feat:` | Minor (`1.2.0` → `1.3.0`) |
| `fix:` or any other type | Patch (`1.2.0` → `1.2.1`) |
| `feat!:` or `BREAKING CHANGE:` footer | Major (`1.2.0` → `2.0.0`) |

Dev builds (non-production branch) produce `X.Y.Z-dev.N` (e.g. `2.2.0-dev.271`).

---

## Pinning to a Version

Callers can pin to:

| Reference | Behaviour |
|-----------|----------|
| `@main` | Always uses the latest version (recommended for active projects) |
| `@v1` | Auto-updated on non-breaking changes within v1 |
| `@v1.2.3` | Fully pinned — explicit opt-in for each update |

Releases are published as GitHub Releases with semantic version tags.

---

## Expected Project Structure

Your project must follow this layout for the workflows to work:

```
your-project/
├── gradle/libs.versions.toml   ← version catalog (minecraft, forge, neoforge keys)
├── gradle.properties           ← must contain modVersion=X.Y.Z
├── gradlew
├── build.gradle.kts
├── settings.gradle.kts
├── common/                     ← shared logic (root Gradle subproject)
├── bukkit/
│   ├── gradlew                 ← independent Gradle build
│   └── build/libs/*.jar
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

> **Note:** Each platform must have its own `gradlew` — they run as independent Gradle builds. See [mc-multiplatform-template](https://github.com/dodoflix/mc-multiplatform-template) for a ready-made starter.

---

## Scope & Limitations

This toolkit is designed specifically for Minecraft multi-platform projects. The following are Minecraft-specific and cannot be easily removed:

| Component | MC-specific part |
|-----------|-----------------|
| Integration tests | Downloads Paper/Fabric/Forge/NeoForge servers, parses `logs/latest.log` |
| Modrinth publishing | Uses `mc-publish` with MC loader names (`fabric`, `forge`, `neoforge`, `spigot`, `paper`, `purpur`) |
| Version catalog | Expects `minecraft`, `forge`, `neoforge` keys in `libs.versions.toml` |
| Change detection | Looks for `bukkit/`, `fabric/`, `forge/`, `neoforge/` directory names |

For non-MC multi-module Gradle projects, the `ci.yml` `unit-test` and build jobs are reusable by setting all `run-*` inputs to `false` and implementing your own integration test step.

---

## Projects Using This Toolkit

- [DisableVillagerTrade](https://github.com/dodoflix/DisableVillagerTrade)
- [mc-multiplatform-template](https://github.com/dodoflix/mc-multiplatform-template)

---

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md).

## License

MIT — see [LICENSE](LICENSE).
