# GitHub Copilot Instructions — mc-multiplatform-toolkit

> **These instructions are authoritative.** Read this file before making any changes.
> If the codebase has drifted from what is documented, **update this file** as part of the same task.

---

## Development Workflow

### Task Size Gating — Plan Before Acting

Assess scope before touching any workflow file:

**Always plan first when ANY of these are true:**
- Adding a new workflow file or removing an existing one
- Adding/removing/renaming an input or output (affects all callers)
- Task requires changes to more than one workflow file
- The request is ambiguous — e.g. "improve performance" without specifics

**Planning steps:**
1. Use `explore` agent to read the relevant workflow(s) and understand current structure
2. Write a plan to the session plan file; track todos in the SQL `todos` table
3. Confirm with the user before writing YAML

**Start directly (no plan needed):**
- Single-step bug fix (e.g. fix a shell quoting issue)
- Action SHA/version bump (`chore(deps):`)
- Documentation update (README, comments)

---

### Sub-Agent Dispatch — One Agent Per Lifecycle Phase

| Phase | Agent type | Example use |
|---|---|---|
| **Explore** | `explore` | "How does the NeoForge build step work? What inputs does it use?" |
| **Implement** | `general-purpose` | Multi-workflow edits; new job/step logic; complex shell scripts |
| **Validate** | `task` | Push to develop branch → wait for `validate.yml` (actionlint) to pass |
| **Code review** | `code-review` | Review staged YAML before committing; catches shellcheck issues early |

**Parallelise:** Run multiple `explore` agents in one response for independent workflow questions. Never re-read what `explore` already returned.

**Always validate before merging to main** — `validate.yml` runs actionlint + SHA-pin check automatically on push to develop.

---

### Reference Consistency Check

**After every change — before committing — scan for stale references.**

Any time you rename, move, or change a workflow name, input, output, or documented behaviour, grep for it everywhere:

```bash
# Find all references to a renamed file, input, or concept
grep -r "old-name" . --include="*.md" --include="*.yml"
```

Things to check after common change types:

- **Workflow renamed** → grep old name in `README.md`, `CONTRIBUTING.md`, this file, issue templates, and caller workflows in consumer projects
- **Input added/renamed/removed** → update `README.md` inputs table; check existing callers
- **New workflow added** → update this file's **Files** section, `README.md`, `CONTRIBUTING.md`, issue templates
- **self-cd.yml versioning logic changed** → check `README.md` versioning section and this file's **Versioning Strategy**

**Fix every stale reference in the same commit as the original change.** A rename with dangling references is an incomplete commit.

---

### Workflow-Specific "TDD"

This repo has no unit tests, but actionlint acts as the test suite. Follow this cycle:

1. **Specify** the expected behaviour in a comment or plan — what should the new step/job do?
2. **Write** the YAML change
3. **Self-check** with `code-review` agent before pushing — catches SC2086, SC2034, [expression] issues
4. **Push to develop** → `task` agent monitors `validate.yml` run
5. Fix any actionlint failures in a follow-up commit to develop
6. **Merge to main** only when validate passes

For new inputs: document the input in `README.md` inputs table *before* implementing the YAML, so the spec is clear.

---



## Project Overview

A collection of reusable GitHub Actions [`workflow_call`](https://docs.github.com/en/actions/sharing-automations/reusing-workflows) workflows for multi-platform Minecraft mods and plugins. Projects **call** these workflows rather than copy them, so improvements propagate automatically.

**Scope:** Intentionally Minecraft-specific. The integration tests download real game servers; the release workflow publishes to Modrinth. This is by design.

---

## Files

```
.github/workflows/
├── ci.yml          ← Reusable CI: check-changes → unit-tests → build-* → test-* → Codecov upload → summary
├── cd.yml          ← Reusable release: versioning → build → integration-tests → GitHub Release → Modrinth
├── validate.yml    ← Toolkit's own CI: actionlint + SHA-pin check on push/PR to develop/main
└── self-cd.yml     ← Toolkit's own release: triggered by validate passing on main
```

There is **no application code** in this repository — only workflow YAML files.

---

## Codecov Integration

`ci.yml` uploads JaCoCo XML coverage reports to Codecov after unit tests complete.

- Controlled by `upload-coverage` boolean input (default: `true`)
- Requires `CODECOV_TOKEN` repository secret in the **caller** repo — pass via `secrets: inherit`
- `fail_ci_if_error: false` — CI never fails if the token is missing or Codecov is down
- Path glob: `**/build/reports/jacoco/test/jacocoTestReport.xml` and `**/build/reports/jacoco/jacocoTestReport.xml`
- Action pinned to commit SHA: `codecov/codecov-action@<sha>  # v5`

Callers must have `codecov.yml` in their repo root to configure coverage targets (optional but recommended).

---

## Workflow Design Principles

1. **Every input must have a default** — callers should need minimal configuration
2. **Each platform job is independently skippable** via `run-bukkit`, `run-fabric`, `run-forge`, `run-neoforge` boolean inputs
3. **No hardcoded project values** — everything flows from `inputs.*`
4. **Every input and output is documented** in the workflow's `inputs:` / `outputs:` block with a `description:`
5. **Fail fast** — use `continue-on-error: true` only for non-critical publish steps (e.g. Modrinth)
6. **All GitHub Actions are pinned to commit SHAs** — never use `@main`, `@latest`, or floating tags for external actions. Always include a comment with the human-readable version: `actions/checkout@de0fac2...  # v6`

---

## Input Naming Conventions

| Pattern | Example | Description |
|---------|---------|-------------|
| `mod-name` | `DisableVillagerTrade` | PascalCase display name |
| `mod-id` | `disablevillagertrade` | lowercase ID, used in log grep |
| `run-<platform>` | `run-bukkit: true` | boolean gate for each platform |
| `<platform>-mc-versions` | `bukkit-mc-versions` | newline-separated MC version list |
| `*-branch` | `production-branch` | branch name strings |

---

## Adding a New Input

1. Add it to the workflow's `on.workflow_call.inputs:` block with `description:`, `type:`, and `default:` (if optional)
2. Consume it via `${{ inputs.input-name }}` in job steps
3. Document it in `README.md` inputs reference table
4. Document it in `copilot-instructions.md` if it affects Codecov, CI gating, or other cross-cutting concerns
5. If breaking (removes or renames existing input), add `BREAKING CHANGE:` footer to commit

---

## Action Version Pinning

When adding or updating a GitHub Action:

```yaml
# CORRECT — pinned to commit SHA
- uses: actions/checkout@de0fac2e4500dabe0009e67214ff5f5447ce83dd  # v6

# WRONG — floating version
- uses: actions/checkout@v6
- uses: actions/checkout@main
```

To find the commit SHA for a tag:
```bash
curl -s https://api.github.com/repos/actions/checkout/git/ref/tags/v6 | python3 -c "import sys,json; print(json.load(sys.stdin)['object']['sha'])"
```

---

## Testing Changes

Since this repo contains only workflow YAML, test by:

1. Point a test project's caller to your branch:
   ```yaml
   uses: dodoflix/mc-multiplatform-toolkit/.github/workflows/ci.yml@your-branch
   # or for cd.yml:
   uses: dodoflix/mc-multiplatform-toolkit/.github/workflows/cd.yml@your-branch
   ```
2. Trigger a run on the test project (push a commit or `workflow_dispatch`)
3. Verify all jobs complete as expected
4. Restore the caller to `@main` after testing

**Never test by pushing broken YAML to `main`** — all callers depend on `@main`.

---

## Commit Messages

Follow [Conventional Commits](https://www.conventionalcommits.org/):

```
feat: add artifact caching between jobs
fix: correct NeoForge server install path
chore(deps): bump actions/checkout to v6.1.0
docs: clarify modrinth-project-id usage in README
ci: pin actions/upload-artifact to commit SHA
```

| Type | When to use | Version bump |
|------|-------------|-------------|
| `feat` | New input, new job, new capability | Minor |
| `fix` | Bug in workflow logic | Patch |
| `chore` | Deps, tooling, action version bumps | None |
| `docs` | README, comments | None |
| `feat!` / `BREAKING CHANGE:` | Removes/renames existing inputs | Major |

---

## Versioning Strategy

- `@main` — latest (callers using this get updates automatically)
- `@v1` — major version tag; auto-moved on non-breaking changes
- `@v1.2.3` — exact pinned version

When releasing a new version, create a GitHub Release with a semver tag. Update the major version tag (`v1`) to point to the new commit.

---

## Key Technical Notes

- `workflow_call` inputs are only accessible via `${{ inputs.* }}` — they cannot be used in `env:` at the top level; they must be passed to jobs via `with:` or `env:` blocks
- Cross-repo `workflow_call` runs in the **caller's** context — `github.repository`, `github.event_name` etc. reflect the caller, not this repo
- `secrets: inherit` in the caller passes all secrets through automatically within the same owner
