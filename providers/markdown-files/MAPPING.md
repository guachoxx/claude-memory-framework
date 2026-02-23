# Markdown Files Provider — Document Mapping

> How each framework concept maps to local `.md` files.

## Layer 1: Root Index

| Framework document | Persisted as |
|---|---|
| Root index | `CLAUDE.md` at project root (~70 lines) |

Claude reads this file first on every session. It contains the navigation table and distillation protocol summary.

## Layer 2: Reference Documents

| Framework document | Persisted as |
|---|---|
| Architecture | `claude-memory/ARCHITECTURE.md` |
| Conventions | `claude-memory/CONVENTIONS.md` |
| Build commands | `claude-memory/BUILD_COMMANDS.md` |
| Testing methodology | `claude-memory/TESTING_METHODOLOGY.md` |
| Credentials | `claude-memory/CREDENTIALS.md` (.gitignored) |
| Lessons learned | `claude-memory/LESSONS_LEARNED.md` |

## Layer 3: Module Context

| Framework document | Persisted as |
|---|---|
| Module context | `src/{module}/CLAUDE.md` (~50 lines max) |

One file per code module, placed alongside the code it describes.

## Project Containers

| Framework concept | Persisted as |
|---|---|
| Project container | Directory: `claude-memory/projects/{project-name}/` |
| Project index | `claude-memory/projects/_INDEX.md` |

Naming: kebab-case for project directories (e.g., `auth-refactor/`, `api-migration/`).

## Project Documents

| Framework document | Persisted as |
|---|---|
| Current Status | `claude-memory/projects/{name}/CURRENT_STATUS.md` |
| Technical Analysis | `claude-memory/projects/{name}/TECHNICAL_ANALYSIS.md` |
| Plan | `claude-memory/projects/{name}/PLAN.md` |
| Technical Report | `claude-memory/projects/{name}/TECHNICAL_REPORT.md` |
| Changelog | `claude-memory/projects/{name}/CHANGELOG.md` |

## Cross-references

Documents reference each other using relative file paths:
```markdown
See `claude-memory/CONVENTIONS.md` → "Session Distillation Protocol"
```

## Version Control

- Changes are tracked via git (commits, branches, PRs)
- Credentials and logs are excluded via `.gitignore`
- All other memory documents are versioned with the codebase

## Read/Write Operations

Claude Code accesses documents natively through its file read/write tools. No MCP server or API integration required.
