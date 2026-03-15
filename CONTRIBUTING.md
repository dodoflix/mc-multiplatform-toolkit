# Contributing to mc-multiplatform-toolkit

Thank you for your interest in contributing!

## How to Contribute

1. Fork the repository
2. Create a branch: `git checkout -b feat/my-improvement`
3. Make your changes
4. Open a pull request targeting `main`

## What Can Be Contributed

- Improvements to `ci.yml` or `release.yml` reusable workflows
- New reusable workflow files for other use cases
- Documentation improvements
- Bug fixes for workflow logic

## Commit Messages

Use [Conventional Commits](https://www.conventionalcommits.org/):

```
feat: add artifact caching between jobs
fix: correct NeoForge server install path
docs: clarify modrinth-project-id input usage
ci: pin action versions to commit SHAs
```

## Workflow Design Principles

- **All inputs must have defaults** where sensible — callers should need minimal configuration
- **Each job should be skippable** via the `run-*` boolean inputs
- **No hardcoded project-specific values** — everything must come from `inputs.*`
- **Document every input and output** in the workflow's `inputs:` block
- **Fail fast but don't block releases** — use `continue-on-error` only for non-critical publish steps

## Versioning

This toolkit uses [Semantic Versioning](https://semver.org/). Callers may pin to:

- `@main` — latest (recommended for active development)
- `@v1` — major version tag (auto-updated on non-breaking changes)
- `@v1.2.3` — exact version (fully pinned, opt-in to updates manually)

### Version Bump Rules

| Commit type | Version bump |
|-------------|-------------|
| `fix:` | Patch |
| `feat:` | Minor |
| `feat!:` or `BREAKING CHANGE:` | Major |

## Testing Changes

Since this repo contains only workflow files, changes should be tested by:

1. Pointing a test project's caller to your branch: `dodoflix/mc-multiplatform-toolkit/.github/workflows/ci.yml@your-branch`
2. Triggering a run on the test project
3. Verifying all jobs complete as expected

## Questions?

Open a [GitHub Discussion](https://github.com/dodoflix/mc-multiplatform-toolkit/discussions) for questions or ideas.
