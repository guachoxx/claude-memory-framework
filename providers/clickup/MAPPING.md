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

| Framework concept | ClickUp entity |
|---|---|
| Project container | Folder: `{project-name}` inside Projects folder |
| Project index | List: `Project Index` with custom fields |

### Project Index custom fields

| Field | Type | Maps to |
|---|---|---|
| Status | Dropdown: PLANNING, IN_PROGRESS, TESTING, READY, RELEASED, ON_HOLD | Project status |
| Branch | Short text | Git branch |
| Started | Date | Start date |
| Summary | Short text | Project summary |

Each project is a task in this List. This replaces `_INDEX.md`.

## Project Documents

| Framework document | ClickUp entity | Location |
|---|---|---|
| Current Status | Doc: `CURRENT_STATUS` | Projects/{name}/ folder |
| Technical Analysis | Doc: `TECHNICAL_ANALYSIS` | Projects/{name}/ folder |
| Plan | Doc: `PLAN` | Projects/{name}/ folder |
| Technical Report | Doc: `TECHNICAL_REPORT` | Projects/{name}/ folder |
| Changelog | Doc: `CHANGELOG` | Projects/{name}/ folder |

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
- Module context (`{module}/CLAUDE.md`) — read when working on a module

**In ClickUp (MCP tools):**
- Reading Doc content (reference and project documents)
- Updating Doc content
- Creating new Docs (when creating a project or new reference document)
- Listing Docs in a Folder (to find project documents)
- Managing tasks in Lists (for Project Index — creating, updating status)

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
