# Markdown Files Provider — Document Mapping

> How each framework concept maps to local `.md` files.

## Layer 1: Root Index and Configuration

| Framework document | Persisted as |
|---|---|
| Root index | `CLAUDE.md` at project root (~70 lines) |
| Configuration | `claude-memory/CONFIG.md` (gitignored, per-user) |

Claude reads `CLAUDE.md` first on every session. It contains the navigation table and distillation protocol summary. Then reads `CONFIG.md` for provider type, user identity, and connection settings.

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

| Framework concept | Single-user | Multi-user |
|---|---|---|
| Project container | `claude-memory/projects/{project-name}/` | `claude-memory/projects/{username}/{project-name}/` |
| Project index | `claude-memory/projects/_INDEX.md` | `claude-memory/projects/{username}/_INDEX.md` |
| Team overview | (same as project index) | `claude-memory/projects/_INDEX.md` (optional) |

Naming: kebab-case for project directories (e.g., `auth-refactor/`, `api-migration/`).

## Project Documents

| Framework document | Persisted as |
|---|---|
| Current Status | `claude-memory/projects/{name}/CURRENT_STATUS.md` |
| Specifications | `claude-memory/projects/{name}/SPECIFICATIONS.md` |
| Technical Analysis | `claude-memory/projects/{name}/TECHNICAL_ANALYSIS.md` |
| Plan | `claude-memory/projects/{name}/PLAN.md` |
| Changelog | `claude-memory/projects/{name}/CHANGELOG.md` |
| Technical Report | `claude-memory/projects/{name}/TECHNICAL_REPORT.md` |
| Testing | `claude-memory/projects/{name}/TESTING.md` |

## Multi-User Mode

When `current_user` is set (via `claude-memory/CONFIG.md`), the project directory structure adds a user namespace:

```
claude-memory/projects/
├── _INDEX.md                              ← Team overview (optional)
├── eugenio/
│   ├── _INDEX.md                          ← Eugenio's project index
│   ├── auth-refactor/
│   │   ├── CURRENT_STATUS.md
│   │   ├── TECHNICAL_ANALYSIS.md
│   │   └── ...
│   └── api-migration/
│       └── ...
└── maria/
    ├── _INDEX.md                          ← Maria's project index
    └── checkout-fix/
        └── ...
```

Claude resolves all project paths using `current_user`. Cross-user references use the full path:
```markdown
See `claude-memory/projects/maria/api-migration/TECHNICAL_REPORT.md`
```

When `current_user` is NOT set, the structure is flat (single-user mode) — no user directories.

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

## Provider Cache

Not applicable. The markdown-files provider accesses all documents via native file tools — no entity IDs to cache. The `claude-memory/PROVIDER_CACHE.md` file is only used by external/hybrid providers.
