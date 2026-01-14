# Repository Standards

## Principles

All arillso repositories are **public** with MIT License.

Focus on essentials - no unnecessary files.

---

## Recommended Minimal Structure

```text
repository/
├── .github/
│   ├── workflows/
│   │   └── [project-specific].yml
│   ├── CODEOWNERS
│   └── renovate.json
├── AGENTS.md                      # AI instructions (main file)
├── CLAUDE.md                      # Only: @AGENTS.md
├── README.md                      # Main documentation
├── LICENSE                        # MIT License
├── .editorconfig                  # Editor consistency
├── .gitignore                     # Project-specific
└── [project files]
```

---

## Required Files

### 1. README.md

**Content (minimal):**

```markdown
# Project Name

Short description (1-2 sentences).

## Quick Start

[Commands to start]

## License

MIT License
```

**Avoid:**

- Badges (except for packages and Ansible collections)
- Long feature lists (except for Ansible collections listing roles/modules/plugins)
- Code examples (belong in AGENTS.md)

### 2. LICENSE

MIT License for all public repos.

```text
MIT License

Copyright (c) [YEAR] Arillso

Permission is hereby granted...
```

### 3. .editorconfig

See [templates/.editorconfig](./templates/.editorconfig)

### 4. .gitignore

Project-specific, keep minimal:

- Build outputs
- Dependencies (vendor/, node_modules/)
- IDE files (.idea, .vscode)
- OS files (.DS_Store)
- Secrets (.env)

### 5. .github/CODEOWNERS

```text
# Default owner
* @sbaerlocher
```

### 6. .github/renovate.json

Required for all repos:

```json
{
    "$schema": "https://docs.renovatebot.com/renovate-schema.json",
    "extends": ["github>arillso/.github:renovate-base"]
}
```

---

## Renovate Configuration

### Available Presets

| Preset | Extends | Use For |
|--------|---------|---------|
| `renovate-base` | - | All repos (base config) |
| `renovate-go` | base | Go projects |
| `renovate-actions` | base | GitHub Actions (Docker, Go) |
| `renovate-ansible` | base | Ansible collections & roles |

### Usage

**Base (all repos):**

```json
{
    "extends": ["github>arillso/.github:renovate-base"]
}
```

**Go projects:**

```json
{
    "extends": ["github>arillso/.github:renovate-go"]
}
```

**GitHub Actions:**

```json
{
    "extends": ["github>arillso/.github:renovate-actions"]
}
```

**Ansible projects:**

```json
{
    "extends": ["github>arillso/.github:renovate-ansible"]
}
```

See [templates/](./templates/) for copy-paste examples.

---

## Workflows

### Naming Convention

- Lowercase with hyphens
- Short and descriptive

### Standard Workflows

| Workflow | Filename | Description |
|----------|----------|-------------|
| Continuous Integration | `ci.yml` | Linting |
| Tests | `test.yml` | Project-specific tests |
| Deploy | `deploy.yml` or `publish.yml` | Build, publish, create GitHub Release |
| Security | `security.yml` | Trivy, secret scanning |
| CodeQL | `codeql.yml` | CodeQL scanning (required for public repos) |

**Note:** Both `deploy.yml` and `publish.yml` are acceptable for deployment workflows. Use `publish.yml` when the primary focus is publishing packages (Ansible Collections, npm packages, Python packages, etc.).

### Deploy Workflow

Triggered by tags. Combines build/publish and release:

```bash
git tag v1.0.0
git push origin v1.0.0
```

**By project type:**

| Type | What deploy.yml does |
|------|---------------------|
| Docker/Actions | Build image → push to ghcr.io → GitHub Release |
| Ansible Collection | Build collection → publish to Galaxy → GitHub Release |
| Go Binary | Build binary → attach to GitHub Release |

See [templates/workflows/](./templates/workflows/) for examples

### GitHub Actions Security

**All GitHub Actions must be pinned to SHA digest for security.**

Required format:

```yaml
- uses: actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4
```

**Why:**
- Prevents supply chain attacks
- Ensures immutable action versions
- Renovate automatically manages digest updates via `pinDigests: true`

**Never use:**

```yaml
- uses: actions/checkout@v4  # ❌ Not pinned
- uses: actions/checkout@main  # ❌ Mutable reference
```

### Schedule Frequency

Scheduled workflows run **weekly** on Monday:

```yaml
schedule:
    - cron: "0 6 * * 1" # Weekly Monday 06:00 UTC
```

Common schedules:

- `codeql.yml`: Monday 06:00 UTC
- `security.yml`: Monday 02:00 UTC

### By Repo Type

| Repo Type | Workflows |
|-----------|-----------|
| GitHub Actions | `ci.yml`, `test.yml`, `deploy.yml`, `codeql.yml`, `security.yml` |
| Docker Images | `ci.yml`, `deploy.yml`, `security.yml` |
| Ansible Roles | `ci.yml`, `test.yml` |

---

## Ansible-Specific Standards

### Argument Specs

**Required for all Ansible roles.**

Every role must define an argument spec in `meta/argument_specs.yml` to document and validate role variables.

**Benefits:**
- Auto-validates role input parameters
- Provides documentation for role variables
- Enables IDE autocomplete and type checking
- Replaces need for extensive README variable documentation

**Minimal structure:**

```yaml
---
argument_specs:
  main:
    short_description: "Brief description of what the role does"
    description:
      - "Detailed description of the role's purpose and functionality"
    author:
      - "Author Name"
    options:
      variable_name:
        description: "What this variable does"
        type: "str"  # str, int, bool, list, dict, path, etc.
        required: false
        default: "default_value"
```

**Example:**

```yaml
---
argument_specs:
  main:
    short_description: "Configure system timezone"
    description:
      - "Manages system timezone configuration across different Linux distributions"
    author:
      - "Arillso"
    options:
      timezone_name:
        description: "Timezone name (e.g., 'Europe/Zurich', 'America/New_York')"
        type: "str"
        required: true
      timezone_hardware_clock:
        description: "Set hardware clock to UTC or local time"
        type: "str"
        required: false
        default: "UTC"
        choices:
          - "UTC"
          - "local"
```

**Validation:**

Argument specs are automatically validated by `ansible-lint` and during role execution.

### Role README Structure

**Required for all Ansible roles.**

Each role must have a `README.md` file following this minimal structure:

```markdown
# Ansible Role: role_name

Brief description (1-2 sentences) of what the role does.

## Features

- **Feature 1**: Brief description
- **Feature 2**: Brief description
- **Feature 3**: Brief description

## Documentation

For detailed documentation including all variables, examples, and usage instructions, see:

**[https://guide.arillso.io/collections/arillso/collection_name/role_name_role.html](https://guide.arillso.io/collections/arillso/collection_name/role_name_role.html)**

## Quick Start

```yaml
- hosts: servers
  roles:
    - role: arillso.collection_name.role_name
      vars:
        role_key_variable: value
```

## License

MIT

## Author Information

This role was created by [arillso](https://github.com/arillso).
```

**Guidelines:**
- Keep it minimal - comprehensive documentation lives on guide.arillso.io
- **Features section**: 3-5 bullet points highlighting key capabilities
- Only provide a quick start example
- Link to the guide page for complete documentation and all variables
- Guide URL pattern: `https://guide.arillso.io/collections/arillso/{collection}/{role}_role.html`
- No variable documentation in README - users should refer to the guide or `argument_specs.yml`

---

## Formatter by Language

| Language | Formatter | Config File |
|----------|-----------|-------------|
| Go | gofmt | - (built-in) |
| YAML | yamllint | `.yamllint.yml` |
| Shell | shfmt | - |
| Dockerfile | hadolint | - |

Only add config when the language is used in the repo.

See [templates/.golangci.yml](./templates/.golangci.yml) and [templates/.yamllint.yml](./templates/.yamllint.yml).

---

## AI Agent Documentation

AI instructions belong in a separate file, **not** in README.

### Structure

```text
repository/
├── AGENTS.md              # AI instructions (main file)
├── CLAUDE.md              # Only import: @AGENTS.md
└── README.md              # Human-readable docs
```

### CLAUDE.md

```markdown
@AGENTS.md
```

### AGENTS.md

```markdown
# Project Name

## Context

[What the project does, for AI]

## Conventions

[Code style, patterns]

## Structure

[Important folders/files]

## Do Not

[What AI should avoid]
```

---

## CHANGELOG.md

Required for all repos with releases. Used by deployment workflows (`deploy.yml` or `publish.yml`) to generate GitHub Release notes.

Format: [Keep a Changelog](https://keepachangelog.com/)

**Usage in Releases:**
- Release workflows should extract the relevant version section from CHANGELOG.md
- Include it in the GitHub Release notes automatically
- This ensures consistency between CHANGELOG.md and release documentation

```markdown
# Changelog

## [Unreleased]

### Added

- New feature

## [1.0.0] - 2025-01-15

### Added

- Feature X

### Fixed

- Bug Y
```

**Sections:** Added, Changed, Deprecated, Removed, Fixed, Security

---

## Optional (Only When Useful)

### Extended Documentation

For larger projects:

| File | Purpose | When |
|------|---------|------|
| `ARCHITECTURE.md` | System design, components | Complex systems |
| `OPERATIONS.md` | Runbooks, troubleshooting | Production services |

---

## What Does NOT Belong in Every Repo

| File | Reason |
|------|--------|
| docs/ folder | README is usually enough |
| SECURITY.md | Only for public packages |
| Issue/PR Templates | Overkill for small repos |
| CONTRIBUTING.md | Only for community projects |

---

## Template for New Repos

```text
new-repo/
├── .github/
│   ├── workflows/
│   │   ├── ci.yml
│   │   ├── release.yml
│   │   └── codeql.yml
│   ├── CODEOWNERS
│   └── renovate.json
├── AGENTS.md
├── CLAUDE.md
├── CHANGELOG.md
├── .editorconfig
├── .gitignore
├── LICENSE
├── README.md
└── [project files]
```

---

## Summary

1. Less is more
2. README is the main documentation
3. LICENSE always MIT
4. CHANGELOG for all repos (used by release.yml)
5. Workflows only what's needed
6. No template files that stay empty
