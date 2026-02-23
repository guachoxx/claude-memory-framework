# ClickUp Provider — Document Mapping

> How each framework concept maps to ClickUp entities.

## Entity Mapping Overview

| Framework concept | ClickUp entity |
|---|---|
| Document (reference / project) | Doc |
| Container | Folder |
| Project container | Folder inside "Projects" |
| Project index | List with tasks (one per project) |
| Reference | Link between Docs or `@mention` |
| Module context | **On disk**: `{module}/CLAUDE.md` (not in ClickUp) |
| Root index | **On disk**: `CLAUDE.md` + optional ClickUp Doc `CLAUDE Index` |

> **Hybrid provider**: Module context always lives on disk. Reference documents and project documents live in ClickUp. The root index lives on disk as `CLAUDE.md` (Claude's entry point) and may optionally be mirrored as a ClickUp Doc for team navigation.

### Consolidated View: On Disk vs. In ClickUp

```
ON DISK                                IN CLICKUP
───────                                ──────────
CLAUDE.md (root index)                 Space: "Claude Memory"
.user (identity, gitignored)           ├── Doc: "CLAUDE Index" (optional)
claude-memory/                         ├── Folder: "Reference"
  └── PROVIDER_CACHE.md (gitignored)   │   ├── Doc: ARCHITECTURE
                                       │   ├── Doc: CONVENTIONS
src/{module}/CLAUDE.md                 │   ├── Doc: BUILD_COMMANDS
  (one per code module)                │   ├── Doc: TESTING_METHODOLOGY
                                       │   ├── Doc: CREDENTIALS (restricted)
                                       │   └── Doc: LESSONS_LEARNED
                                       └── Folder: "Projects"
                                           ├── List: "Project Index"
                                           └── Folder: "{project-name}"
                                               ├── Doc: CURRENT_STATUS
                                               ├── Doc: SPECIFICATIONS
                                               ├── Doc: TECHNICAL_ANALYSIS
                                               ├── Doc: PLAN
                                               ├── Doc: CHANGELOG
                                               ├── Doc: TECHNICAL_REPORT
                                               └── Doc: TESTING
```

## Layer 1: Root Index

The root index has two components in this hybrid provider:

| Component | Where | Purpose |
|---|---|---|
| `CLAUDE.md` | **On disk**, project root | Bootstrap entry point. Claude reads this natively on startup. Contains system overview, provider instructions, and navigation. |
| `CLAUDE Index` (optional) | ClickUp Doc, Space level | Team-facing navigation hub with links to all ClickUp Docs. Useful for humans browsing the Space. |

The on-disk `CLAUDE.md` is the authoritative entry point. It should contain a note telling Claude to use MCP tools to access project and reference documents in ClickUp.

## Layer 2: Reference Documents

| Framework document | ClickUp entity | Location |
|---|---|---|
| Architecture | Doc: `ARCHITECTURE` | Reference folder |
| Conventions | Doc: `CONVENTIONS` | Reference folder |
| Build commands | Doc: `BUILD_COMMANDS` | Reference folder |
| Testing methodology | Doc: `TESTING_METHODOLOGY` | Reference folder |
| Credentials | Doc: `CREDENTIALS` | Reference folder (restricted) |
| Lessons learned | Doc: `LESSONS_LEARNED` | Reference folder |

## Layer 3: Module Context

| Framework document | Persisted as | Location |
|---|---|---|
| Module context | `{module}/CLAUDE.md` (~50 lines max) | **On disk**, alongside the code |

Module context **always lives on disk** regardless of provider. Claude Code reads these files natively when working on a module. They are code documentation, not project documentation — they belong with the code they describe.

## Project Containers

| Framework concept | Single-user | Multi-user |
|---|---|---|
| Project container | Folder: `{project-name}` inside Projects | Folder: `{project-name}` inside `Projects/{username}/` |
| Project index | List: `Project Index` | Same List, filtered by Owner (Assignee) |

### Project Index custom fields

| Field | Type | Maps to |
|---|---|---|
| Status | Dropdown: PLANNING, IN_PROGRESS, TESTING, READY, RELEASED, ON_HOLD | Project status |
| Branch | Short text | Git branch |
| Started | Date | Start date |
| Summary | Short text | Project summary |
| Owner | Assignee (People) | Project owner (multi-user mode) |

Each project is a task in this List. This replaces `_INDEX.md`.

## Project Documents

| Framework document | ClickUp entity | Location |
|---|---|---|
| Current Status | Doc: `CURRENT_STATUS` | Projects/{name}/ folder |
| Specifications | Doc: `SPECIFICATIONS` | Projects/{name}/ folder |
| Technical Analysis | Doc: `TECHNICAL_ANALYSIS` | Projects/{name}/ folder |
| Plan | Doc: `PLAN` | Projects/{name}/ folder |
| Changelog | Doc: `CHANGELOG` | Projects/{name}/ folder |
| Technical Report | Doc: `TECHNICAL_REPORT` | Projects/{name}/ folder |
| Testing | Doc: `TESTING` | Projects/{name}/ folder |

## Cross-references

ClickUp supports two mechanisms:
- **Doc links**: Embed links to other Docs using ClickUp's native linking
- **@mentions**: Reference Docs or tasks inline with `@`

Example in a Doc:
```
See @CONVENTIONS → "Session Distillation Protocol"
```

## Version Control

- ClickUp Docs maintain edit history natively
- For formal versioning, note the date and author at the top of each Doc update
- CREDENTIALS Doc should have restricted permissions instead of `.gitignore`

## Read/Write Operations

Claude uses **two access methods** in this hybrid provider:

**On disk (native file tools):**
- Root index (`CLAUDE.md`) — read on every session startup
- Provider cache (`claude-memory/PROVIDER_CACHE.md`) — read on startup for cached entity IDs
- Module context (`{module}/CLAUDE.md`) — read when working on a module

**In ClickUp (MCP tools):**
- Reading Doc content (reference and project documents)
- Updating Doc content
- Creating new Docs (when creating a project or new reference document)
- Listing Docs in a Folder (to find project documents)
- Managing tasks in Lists (for Project Index — creating, updating status)

> **Performance note**: With a populated Provider Cache, Claude skips the MCP search/list calls that would otherwise be needed to resolve entity IDs. See "Provider Cache" section below.

## Lite Mode in ClickUp

For small projects, create only:
- Doc: `CURRENT_STATUS` (with Analysis and Plan sections inline)
- Doc: `CHANGELOG`

Both inside a single project Folder. Promote to full structure if the project grows.

## Archiving Completed Projects

When a project reaches RELEASED status:
1. Move `TECHNICAL_REPORT` Doc to a "Documentation" folder (or your team's docs Space)
2. Archive the project Folder in ClickUp
3. Update the task in `Project Index` List to RELEASED status

## Provider Cache

### What Is Cached

Every ClickUp entity that Claude needs to access by ID is cached in `claude-memory/PROVIDER_CACHE.md`:

| Cached entity | ClickUp type | Why cached |
|---|---|---|
| Workspace | Workspace ID | Root of all API calls |
| Space | Space ID + name | Container for all framework content |
| Reference Folder | Folder ID | Contains all reference Docs |
| Each reference Doc | Doc ID | Direct access without searching |
| Projects Folder | Folder ID | Parent of all project containers |
| Project Index List | List ID | Where project tasks live |
| Each project Folder | Folder ID | Direct navigation to project |
| Each project Doc | Doc ID | Direct read/write without listing |

### ClickUp Cache Template

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
| Document | Doc ID | Parent |
|---|---|---|
| ARCHITECTURE | abc123 | Reference (folder: fol456) |
| CONVENTIONS | def789 | Reference |
| BUILD_COMMANDS | ghi012 | Reference |
| TESTING_METHODOLOGY | jkl345 | Reference |
| CREDENTIALS | mno678 | Reference |
| LESSONS_LEARNED | pqr901 | Reference |

## Project Infrastructure
| Entity | ID | Type |
|---|---|---|
| Reference Folder | fol456 | Folder |
| Projects Folder | fol789 | Folder |
| Project Index | lst012 | List |

## Projects
| Project | Folder ID | Documents |
|---|---|---|
| auth-refactor | fol345 | CURRENT_STATUS: doc111, PLAN: doc222, TECHNICAL_ANALYSIS: doc333 |
| api-migration | fol678 | CURRENT_STATUS: doc444, CHANGELOG: doc555 |
```

### How Claude Uses the Cache

**Session startup (with cache)**:
1. Claude reads `CLAUDE.md` (on disk)
2. Claude reads `claude-memory/PROVIDER_CACHE.md` (on disk)
3. Claude now has all ClickUp IDs — no MCP search calls needed
4. To read CURRENT_STATUS for a project: uses the Doc ID directly via `clickup_get_document_pages`

**Session startup (without cache)**:
1. Claude reads `CLAUDE.md`
2. No cache found — Claude generates it (see "Initial Generation" below)

**During work**:
- When Claude creates a new project Folder or Doc → appends the new ID to the cache
- When a cached ID returns a 404/not-found error → Claude regenerates the entire cache

### Initial Generation Workflow

When the cache file does not exist, Claude generates it by querying ClickUp:

1. `clickup_get_workspace_hierarchy` → get Space ID, Folder IDs, List IDs
2. For the Reference Folder: `clickup_search` with `asset_types: ["doc"]` scoped to the Reference Folder → get all reference Doc IDs
3. For the Projects Folder: list sub-Folders → get each project Folder ID
4. For each project Folder: `clickup_search` or list Docs → get project Doc IDs
5. Write the complete cache to `claude-memory/PROVIDER_CACHE.md`

### Update Triggers

| Action | Cache update |
|---|---|
| Create project (new Folder + Docs) | Append project row + Doc IDs |
| Create new Doc in existing project | Update the project's Documents cell |
| Archive/close project | Remove the project row |
| Rename a project or document | Update the affected row |
| Cached ID returns error | Regenerate entire cache |

---

## Multi-User Mode

When `current_user` is set (via `.user` file or `CLAUDE.md`), the ClickUp structure adds a per-user Folder inside Projects:

```
Space: "Claude Memory"
├── Folder: "Reference"
└── Folder: "Projects"
    ├── List: "Project Index"              ← Shared list, with Owner (Assignee) field
    ├── Folder: "eugenio"
    │   └── Folder: "auth-refactor"
    │       ├── Doc: "CURRENT_STATUS"
    │       └── ...
    └── Folder: "maria"
        └── Folder: "checkout-fix"
            └── ...
```

### Shared Project Index with Owner Field

The Project Index List is shared across all users. The `Owner` field (Assignee type) identifies who owns each project.

When `current_user` is set, Claude:
- **Creates** new project tasks with Owner = `current_user` (resolved via `clickup_resolve_assignees`)
- **Filters** the Project Index by Owner when listing "my projects"
- **Can read** all projects (any Owner) when asked to view another user's work
- **Creates** project Folders inside `Projects/{current_user}/`

### Per-User Views (recommended)

Create a filtered View on the Project Index List for each user:
- View name: `{username}'s Projects`
- Filter: Owner = {username}

This keeps the ClickUp UI clean. It is optional — Claude filters programmatically regardless.

### Mapping current_user to ClickUp Identity

The `current_user` value must be resolvable by the ClickUp MCP server. Use one of:
- ClickUp display name (e.g., `eugenio`)
- ClickUp email (e.g., `eugenio@company.com`)

Claude calls `clickup_resolve_assignees` with the `current_user` value to get the ClickUp user ID for assignee operations.

### Single-User Fallback

When `current_user` is NOT set:
- Project Folders are created directly inside "Projects" (no user subfolder)
- Project Index tasks have no Owner assigned
- Behaves identically to single-user mode
