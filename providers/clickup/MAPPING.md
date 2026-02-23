# ClickUp Provider — Document Mapping

> How each framework concept maps to ClickUp entities.

## Entity Mapping Overview

| Framework concept | ClickUp entity |
|---|---|
| Document | Doc |
| Container | Folder |
| Project container | Folder inside "Projects" |
| Project index | List with tasks (one per project) |
| Reference | Link between Docs or `@mention` |
| Module context | Doc inside "Module Context" folder |

## Layer 1: Root Index

| Framework document | ClickUp entity |
|---|---|
| Root index | Doc: `CLAUDE Index` at Space level |

This Doc is the entry point. It contains navigation links to all other Docs. Claude reads it first on every session.

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

| Framework document | ClickUp entity | Location |
|---|---|---|
| Module context | Doc: `{module-name}` | Module Context folder |

One Doc per code module. Name the Doc after the module (e.g., `auth-service`, `payment-gateway`).

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

Claude accesses ClickUp through the MCP server, which exposes tools for:
- Reading Doc content
- Updating Doc content
- Creating new Docs
- Listing Docs in a Folder
- Managing tasks in Lists (for Project Index)

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
