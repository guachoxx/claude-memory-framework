# ClickUp Provider — Document Mapping

> How each framework concept maps to ClickUp entities.

## ClickUp API Limitations

These constraints affect how the framework maps to ClickUp:

1. **No nested Folders** — ClickUp Folders cannot contain other Folders. Project containers cannot be sub-folders of a "Projects" folder.
2. **No custom field creation via API** — Custom fields must be created manually in the ClickUp UI. The framework uses native fields (Status, Assignee) instead of custom fields wherever possible.
3. **Docs support multiple Pages** — A single Doc can contain multiple Pages, each with independent content. This is the native way to group related documents.
4. **Search does not match underscores in Doc/Page names** — `clickup_search` cannot find Docs or Pages whose names contain underscores (e.g., `LESSONS_LEARNED`). Use spaces instead (e.g., `LESSONS LEARNED`). This affects cold start discovery and any search-based lookups.

## Entity Mapping Overview

| Framework concept | ClickUp entity |
|---|---|
| Reference document | Doc (single-page) in Reference folder |
| Project container | Doc (multi-page) in Projects folder |
| Project document (CURRENT STATUS, PLAN, etc.) | Page inside a project Doc |
| Project index | List with tasks (one task per project) |
| Project ownership | Native Assignee on Project Index task |
| Project status | Native Status on Project Index task/list |
| Cross-reference | Link between Docs or `@mention` |
| Module context | **On disk**: `{module}/CLAUDE.md` (not in ClickUp) |
| Root index | **On disk**: `CLAUDE.md` |
| User configuration | **On disk**: `claude-memory/CONFIG.md` (gitignored) |

> **Hybrid provider**: Module context and configuration always live on disk. Reference documents and project documents live in ClickUp. The root index (`CLAUDE.md`) lives on disk as Claude's entry point.

### On-disk Files (ClickUp hybrid provider)

These files and directories persist on disk when using the ClickUp provider:

| Path | Git | Purpose |
|------|-----|---------|
| `CLAUDE.md` | Committed | Root index — Claude's entry point |
| `claude-memory/CONFIG.md` | Gitignored | Provider config, user identity, connection |
| `claude-memory/PROVIDER_CACHE.md` | Gitignored | Cached ClickUp entity IDs (auto-generated) |
| `claude-memory/providers/` | Committed | Framework documentation (MAPPING, SETUP, README) |
| `claude-memory/scripts/permanent/` | Committed | Reusable scripts (extraction, testing, etc.) |
| `claude-memory/templates/` | Committed | Reusable templates (payloads, fixtures, etc.) |
| `claude-memory/logs/` | Gitignored | Working logs and debug output (temporary) |
| `claude-memory/temp/` | Gitignored | Temporary working files |
| `src/{module}/CLAUDE.md` | Committed | Module context (Layer 3) — one per code module |

> **Committed** = shared via git, part of the project. **Gitignored** = local to each machine, not shared.
> Implementations may add project-specific directories (e.g., additional log caches). Add them to `.gitignore` as needed.

### Consolidated View: On Disk vs. In ClickUp

```
ON DISK                                  IN CLICKUP
───────                                  ──────────
CLAUDE.md (root index)                   Space: "Claude Memory"
claude-memory/                           ├── Folder: "Reference"
  ├── CONFIG.md (gitignored)             │   ├── Doc: ARCHITECTURE
  ├── PROVIDER_CACHE.md (gitignored)     │   ├── Doc: CONVENTIONS
  ├── providers/ (committed)             │   ├── Doc: BUILD COMMANDS
  ├── scripts/permanent/ (committed)     │   ├── Doc: TESTING METHODOLOGY
  ├── templates/ (committed)             │   ├── Doc: CREDENTIALS (restricted)
  └── logs/, temp/ (gitignored)          │   └── Doc: LESSONS LEARNED
                                         └── Folder: "Projects"
                                             ├── List: "Project Index"
src/{module}/CLAUDE.md (committed)
                                             ├── Doc: "{project-a}"
                                             │   ├── Page: CURRENT STATUS
                                             │   ├── Page: SPECIFICATIONS
                                             │   ├── Page: TECHNICAL ANALYSIS
                                             │   ├── Page: PLAN
                                             │   ├── Page: CHANGELOG
                                             │   ├── Page: TECHNICAL REPORT
                                             │   └── Page: TESTING
                                             └── Doc: "{project-b}"
                                                 └── Page: CURRENT STATUS
```

## Layer 1: Root Index

| Component | Where | Purpose |
|---|---|---|
| `CLAUDE.md` | **On disk**, project root | Bootstrap entry point. Claude reads this natively on startup. Contains system overview, boot sequence, and navigation. |
| `claude-memory/CONFIG.md` | **On disk**, gitignored | Provider type, connection settings, user identity. Read immediately after `CLAUDE.md`. |

The on-disk `CLAUDE.md` is the authoritative entry point. It directs Claude to read `CONFIG.md` for provider settings, then CONVENTIONS as the first provider document.

## Layer 2: Reference Documents

| Framework document | ClickUp entity | Location |
|---|---|---|
| Architecture | Doc: `ARCHITECTURE` | Reference folder |
| Conventions | Doc: `CONVENTIONS` | Reference folder |
| Build commands | Doc: `BUILD COMMANDS` | Reference folder |
| Testing methodology | Doc: `TESTING METHODOLOGY` | Reference folder |
| Credentials | Doc: `CREDENTIALS` | Reference folder (restricted) |
| Lessons learned | Doc: `LESSONS LEARNED` | Reference folder |

Each reference document is a single-page Doc (one Doc, one Page with the content).

## Layer 3: Module Context

| Framework document | Persisted as | Location |
|---|---|---|
| Module context | `{module}/CLAUDE.md` (~50 lines max) | **On disk**, alongside the code |

Module context **always lives on disk** regardless of provider. Claude Code reads these files natively when working on a module. They are code documentation, not project documentation — they belong with the code they describe.

## Project Containers

Each project is a **multi-page Doc** inside the Projects folder. All project documents (CURRENT STATUS, PLAN, etc.) are **Pages** within that Doc.

| Framework concept | ClickUp entity |
|---|---|
| Project container | Doc: `{project-name}` inside Projects folder |
| Project document | Page inside the project Doc |
| Project index entry | Task in "Project Index" List |

### Why Docs with Pages (not Folders with Docs)

ClickUp does not support nested Folders. The previous approach of "1 Folder per project inside a Projects folder" is impossible. Multi-page Docs are the native ClickUp solution:
- A project is self-contained in a single Doc
- Each document type is a Page within that Doc
- Navigation is natural in the ClickUp UI (sidebar shows Pages)
- API supports full CRUD on Pages: `clickup_create_document_page`, `clickup_update_document_page`, `clickup_get_document_pages`

### Project Index

Each project has a corresponding task in the "Project Index" List. This task uses **native ClickUp fields only** (custom fields cannot be created via API):

| Field | Type | Purpose |
|---|---|---|
| Status | **Native status** (configured on List/Space) | Project lifecycle status |
| Assignee | **Native assignee** | Project owner |
| Description | **Native text** | Structured metadata (see format below) |

**Required statuses** (must be configured manually on the Project Index List or Space):

| Status | Framework phase | ClickUp status type |
|---|---|---|
| `PLANNING` | Analysis + Planning | Open |
| `IN_PROGRESS` | Development | Open |
| `TESTING` | QA validation | Open |
| `READY` | Pre-release | Open |
| `RELEASED` | Completed | Closed |
| `ON_HOLD` | Paused | Open |

**Task description format** for metadata that has no native field:

```
**Branch**: spike/feature-name
**Started**: YYYY-MM
**Summary**: One-line project description
```

## Project Documents

Each project document is a **Page** inside the project's Doc:

| Framework document | ClickUp entity | Location |
|---|---|---|
| Current Status | Page: `CURRENT STATUS` | Doc: `{project-name}` |
| Specifications | Page: `SPECIFICATIONS` | Doc: `{project-name}` |
| Technical Analysis | Page: `TECHNICAL ANALYSIS` | Doc: `{project-name}` |
| Plan | Page: `PLAN` | Doc: `{project-name}` |
| Changelog | Page: `CHANGELOG` | Doc: `{project-name}` |
| Technical Report | Page: `TECHNICAL REPORT` | Doc: `{project-name}` |
| Testing | Page: `TESTING` | Doc: `{project-name}` |

## Cross-references

ClickUp supports two mechanisms:
- **Doc links**: Embed links to other Docs using ClickUp's native linking
- **@mentions**: Reference Docs or tasks inline with `@`

Example in a Page:
```
See @CONVENTIONS → "Session Distillation Protocol"
```

## Version Control

- ClickUp Docs maintain edit history natively
- For formal versioning, note the date and author at the top of each Page update
- CREDENTIALS Doc should have restricted permissions

## Read/Write Operations

Claude uses **two access methods** in this hybrid provider:

**On disk (native file tools):**
- Root index (`CLAUDE.md`) — read on every session startup
- Configuration (`claude-memory/CONFIG.md`) — read on startup for provider settings
- Provider cache (`claude-memory/PROVIDER_CACHE.md`) — read on startup for cached entity IDs
- Module context (`{module}/CLAUDE.md`) — read when working on a module

**In ClickUp (MCP tools):**

| Operation | MCP tool | Parameters |
|---|---|---|
| Read reference Doc | `clickup_get_document_pages` | `(doc_id, [page_id])` |
| Read project Page | `clickup_get_document_pages` | `(doc_id, [page_id])` |
| Update project Page | `clickup_update_document_page` | `(doc_id, page_id, content, content_format="text/md")` |
| Create new Page | `clickup_create_document_page` | `(doc_id, name, content, content_format="text/md")` |
| List Pages in a Doc | `clickup_list_document_pages` | `(doc_id)` |
| Create project Doc | `clickup_create_document` | `(name, parent={id, type="5"}, visibility, create_page)` |
| Read Project Index task | `clickup_get_task` | `(task_id)` |
| Update task status | `clickup_update_task` | `(task_id, status)` |
| Create task | `clickup_create_task` | `(name, list_id, assignees, description)` |

> **Performance note**: With a populated Provider Cache, Claude uses Doc IDs and Page IDs directly — no search or list calls needed. See "Provider Cache" section below.

## Lite Mode

For small projects (fix, scoped refactor, <3 phases), create only:
- Page: `CURRENT STATUS` (with Analysis and Plan sections inline)
- Page: `CHANGELOG`

Both as Pages inside the project Doc. Promote to full structure if the project grows.

## Archiving Completed Projects

When a project reaches RELEASED status:
1. Keep `TECHNICAL REPORT`, `SPECIFICATIONS`, and `TESTING` Pages for reference
2. Update the task in `Project Index` to RELEASED status (closed)
3. Optionally archive or move the project Doc

---

## Provider Cache

### What Is Cached

Every ClickUp entity that Claude needs to access by ID is cached in `claude-memory/PROVIDER_CACHE.md`:

| Cached entity | ClickUp type | Why cached |
|---|---|---|
| Workspace | Workspace ID | Root of all API calls |
| Space | Space ID + name | Container for all framework content |
| Reference Folder | Folder ID | Contains all reference Docs |
| Each reference Doc | Doc ID + Page ID | Direct access to content |
| Projects Folder | Folder ID | Contains all project Docs |
| Project Index List | List ID | Where project tasks live |
| Each project Doc | Doc ID | Parent container for project Pages |
| Each project Page | Page ID | Direct read/write access |
| Each project task | Task ID | Direct status/assignee access |

### Cache Template

```markdown
# Provider Cache
> Auto-generated by Claude. Do not edit manually.
> Provider: clickup
> Generated: YYYY-MM-DD

## Workspace
- Workspace ID: 12345678
- Space ID: 90123456
- Space name: Claude Memory

## Reference Documents
| Document | Doc ID | Page ID | Parent |
|---|---|---|---|
| ARCHITECTURE | abc123 | page001 | Reference (folder: fol456) |
| CONVENTIONS | def789 | page002 | Reference |
| BUILD COMMANDS | ghi012 | page003 | Reference |
| TESTING METHODOLOGY | jkl345 | page004 | Reference |
| CREDENTIALS | mno678 | page005 | Reference |
| LESSONS LEARNED | pqr901 | page006 | Reference |

## Project Infrastructure
| Entity | ID | Type |
|---|---|---|
| Reference Folder | fol456 | Folder |
| Projects Folder | fol789 | Folder |
| Project Index | lst012 | List |

## Projects
| Project | Doc ID | Task ID | Owner | Pages |
|---|---|---|---|---|
| auth-refactor | doc111 | tsk001 | eugenio | CURRENT STATUS: pg101, PLAN: pg102, TECHNICAL ANALYSIS: pg103 |
| api-migration | doc222 | tsk002 | maria | CURRENT STATUS: pg201, CHANGELOG: pg202 |
```

### How Claude Uses the Cache

**Session startup (warm — cache exists)**:
1. Claude reads `CLAUDE.md` (on disk)
2. Claude reads `claude-memory/CONFIG.md` (on disk)
3. Claude reads `claude-memory/PROVIDER_CACHE.md` (on disk) — all IDs available
4. Claude reads CONVENTIONS (ClickUp, using cached Doc ID + Page ID)
5. Claude reads Project Index — overview of active projects (names, statuses, owners)
6. Ready to work. Reads a project's CURRENT STATUS only when the user asks to work on it.

**Session startup (cold — no cache)**:
1. Claude reads `CLAUDE.md` + `CONFIG.md`
2. Claude discovers ClickUp structure (see "Initial Generation" below)
3. Claude reads CONVENTIONS
4. Claude reads Project Index
5. Ready to work

**During work**:
- When Claude creates a new project Doc or Page → appends the new ID to the cache
- When a cached ID returns a 404/not-found error → Claude regenerates the entire cache

### Initial Generation Workflow

When the cache file does not exist, Claude generates it by querying ClickUp using settings from `CONFIG.md`:

1. `clickup_get_workspace_hierarchy` → find the Space by name (from CONFIG.md), get Folder IDs, List IDs
2. For the Reference Folder: `clickup_search` with `asset_types: ["doc"]` scoped to the folder → get reference Doc IDs
3. For each reference Doc: `clickup_list_document_pages` → get Page IDs
4. For the Projects Folder: `clickup_search` with `asset_types: ["doc"]` → get project Doc IDs
5. For each project Doc: `clickup_list_document_pages` → get Page IDs
6. Write the complete cache to `claude-memory/PROVIDER_CACHE.md`

### Update Triggers

| Action | Cache update |
|---|---|
| Create project (new Doc + Pages + task) | Append project row with Doc ID, Task ID, Page IDs |
| Create new Page in existing project | Update the project's Pages cell |
| Archive/close project | Remove the project row |
| Rename a project or Page | Update the affected row |
| Cached ID returns error | Regenerate entire cache |

---

## Multi-User Mode

When `current_user` is set in `CONFIG.md`, Claude uses the native Assignee field on Project Index tasks for ownership.

### How It Works

All project Docs live **flat** inside the Projects folder (no per-user sub-folders — ClickUp does not support nested Folders). Ownership is tracked via the task assignee:

```
Space: "Claude Memory"
├── Folder: "Reference"
└── Folder: "Projects"
    ├── List: "Project Index"
    │   ├── Task: "auth-refactor"    → Assignee: eugenio
    │   ├── Task: "checkout-fix"     → Assignee: maria
    │   └── Task: "api-migration"    → Assignee: eugenio
    ├── Doc: "auth-refactor"
    ├── Doc: "checkout-fix"
    └── Doc: "api-migration"
```

### Ownership Operations

When `current_user` is set, Claude:
- **Creates** new project tasks with Assignee = `current_user` (resolved via `clickup_resolve_assignees`)
- **Filters** the Project Index by Assignee when listing "my projects"
- **Can read** all projects regardless of owner (freely accessible)
- **Creates** project Docs directly inside the Projects folder

### Per-User Views (recommended)

Create a filtered View on the Project Index List for each user:
- View name: `{username}'s Projects`
- Filter: Assignee = {username}

This keeps the ClickUp UI clean. It is optional — Claude filters programmatically regardless.

### Mapping current_user to ClickUp Identity

The `current_user` value in `CONFIG.md` must be resolvable by the ClickUp MCP server. Use one of:
- ClickUp display name (e.g., `Eugenio`)
- ClickUp email (e.g., `eugenio@company.com`)

Claude calls `clickup_resolve_assignees` with the `current_user` value to get the ClickUp user ID for assignee operations.

### Single-User Fallback

When `current_user` is NOT set:
- Project Docs are created inside "Projects" folder (same structure)
- Project Index tasks have no Assignee
- Behaves identically to single-user mode
