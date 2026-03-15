# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Added
- Reusable `ci.yml` workflow for multi-platform Minecraft mod/plugin builds
- Reusable `release.yml` workflow for automated versioning and Modrinth publishing
- Inputs: `mod-name`, `mod-id`, `java-version`, `run-bukkit/fabric/forge/neoforge`, `modrinth-project-id`, `*-mc-versions`, `production-branch`, `develop-branch`
- `check-changes` job to skip builds on non-code commits
- Unit test job with JaCoCo coverage reporting
- Parallel platform build jobs (Bukkit, Fabric, Forge, NeoForge)
- Integration test jobs with real server startup validation
- Automated GitHub Release with semantic versioning from Conventional Commits
- Modrinth publishing with `mc-publish` action
- `develop` branch sync after production release
