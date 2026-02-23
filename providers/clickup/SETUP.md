# ClickUp Provider — Setup

> Documents are stored as ClickUp Docs within a dedicated Space. Claude accesses them via the ClickUp MCP server.

## Prerequisites

- A ClickUp workspace with permissions to create Spaces, Folders, Lists, and Docs
- A ClickUp API token (personal or app token) OR a ClickUp MCP server
- Claude Code installed

## ClickUp Structure

Create the following structure in your ClickUp workspace:

```
Workspace
└── Space: "Claude Memory"
    ├── Folder: "Reference"                    ← Layer 2 (reference documents)
    │   ├── Doc: "ARCHITECTURE"
    │   ├── Doc: "CONVENTIONS"
    │   ├── Doc: "BUILD_COMMANDS"
    │   ├── Doc: "TESTING_METHODOLOGY"
    │   ├── Doc: "CREDENTIALS"                 ← Restrict permissions
    │   └── Doc: "LESSONS_LEARNED"
    └── Folder: "Projects"                     ← Project containers
        ├── List: "Project Index"              ← One task per project
        └── Doc: "{project-name}"              ← One Doc per project
            ├── Page: "CURRENT_STATUS"
            ├── Page: "SPECIFICATIONS"
            ├── Page: "TECHNICAL_ANALYSIS"
            ├── Page: "PLAN"
            ├── Page: "CHANGELOG"
            ├── Page: "TECHNICAL_REPORT"
            └── Page: "TESTING"
```

> **Key design decisions**:
> - **Reference documents** = individual Docs (one Doc, one Page each) inside the Reference folder.
> - **Project documents** = Pages inside a single multi-page Doc per project. ClickUp does not support nested Folders, so this is the native way to group project documents.
> - **Layer 3 (Module Context)** always lives on disk alongside the code as `{module}/CLAUDE.md`, regardless of provider.

## Step-by-step Setup

### 1. Create the Space

Create a Space named `Claude Memory` (or embed it in your existing project Space). This is the top-level container for all framework documents.

### 2. Create the Reference folder

Inside the Space, create a Folder named `Reference`. Create one Doc for each reference document:
- ARCHITECTURE
- CONVENTIONS (paste the contents of the framework's CONVENTIONS document)
- BUILD_COMMANDS
- TESTING_METHODOLOGY
- CREDENTIALS (restrict access — this is sensitive)
- LESSONS_LEARNED

### 3. Set up the Root Index

Place `CLAUDE.md` at your project root. This is what Claude reads on startup. It must include:
- System overview
- Boot sequence (pointing to CONFIG.md, CONVENTIONS, and PROVIDER_CACHE)
- Navigation table pointing to document names
- Distillation protocol summary

### 4. Create CONFIG.md

Create `claude-memory/CONFIG.md` with your provider settings. Add it to `.gitignore` (it contains per-user configuration and may contain API keys).

```markdown
# Memory Configuration
> Per-user, per-machine. This file is gitignored — do not commit.
> See `providers/` for available providers and setup instructions.

## User
current_user: Your Name

## Provider
provider: clickup

## Connection
# Option 1: MCP server (preferred — auth handled by MCP config)
mcp_server: clickup

# Option 2: API key (if no MCP server available)
# api_key: pk_your_key_here

## ClickUp Structure
space_name: Claude Memory
reference_folder: Reference
projects_folder: Projects
project_index_list: Project Index
```

### 5. Create the Projects folder

Create a Folder named `Projects`. Inside it:

**a) Create the Project Index List**

Create a List named `Project Index`. Each project will have a task in this list.

**b) Configure statuses on the List (or Space)**

The Project Index must have statuses matching the framework's project lifecycle. Configure these **manually in the ClickUp UI** (statuses cannot be created via API):

| Status name | Type | Framework phase |
|---|---|---|
| `PLANNING` | Open | Analysis + Planning |
| `IN_PROGRESS` | Open | Active development |
| `TESTING` | Open | QA validation |
| `READY` | Open | Pre-release |
| `ON_HOLD` | Open | Temporarily paused |
| `RELEASED` | Closed | Completed, in production |

> **Note**: You can configure these at the Space level (applies to all Lists) or at the List level (override for Project Index only). Space-level is simpler if you don't use other Lists with different statuses.

**c) Creating projects**

Each project is a **Doc** (not a Folder) inside the Projects folder. Claude creates these via MCP tools:
- `clickup_create_document` → creates the project Doc
- `clickup_create_document_page` → creates each Page (CURRENT_STATUS, PLAN, etc.)
- `clickup_create_task` → creates the Project Index entry with status and assignee

### 6. Configure Claude Code access

Claude Code needs an MCP server to read/write ClickUp. Add it to your Claude Code configuration (`.claude/mcp.json` or global settings):

```json
{
  "mcpServers": {
    "clickup": {
      "command": "npx",
      "args": ["-y", "<clickup-mcp-server-package>"],
      "env": {
        "CLICKUP_API_TOKEN": "your-token-here"
      }
    }
  }
}
```

> **Finding an MCP server**: Search the [MCP server registry](https://github.com/modelcontextprotocol/servers) or npm for a ClickUp MCP server that supports reading/writing Docs and Pages, listing Folders, and managing List tasks.

Alternatively, set `api_key` in CONFIG.md if your setup accesses ClickUp directly without an MCP server.

### How Claude interacts with ClickUp

This is a **hybrid provider**: some documents live on disk, others in ClickUp.

| What Claude does | How it does it |
|---|---|
| Read root index (`CLAUDE.md`) | Native file read (on disk) |
| Read config (`CONFIG.md`) | Native file read (on disk, gitignored) |
| Read CONVENTIONS (first provider document) | MCP: `clickup_get_document_pages` |
| Read/update reference documents | MCP: `clickup_get_document_pages`, `clickup_update_document_page` |
| Read/update project Pages | MCP: `clickup_get_document_pages`, `clickup_update_document_page` |
| Read/update project index | MCP: `clickup_get_task`, `clickup_update_task` |
| Read/update module context | Native file read/write (on disk, always) |
| Read cached entity IDs | Native file read: `claude-memory/PROVIDER_CACHE.md` (on disk) |
| Create a new project | MCP: create Doc + Pages + task in Project Index |
| Close a project | MCP: update task status to RELEASED |

### 7. Generate Provider Cache

Once the MCP server is configured and the ClickUp structure is in place, ask Claude to generate the cache:

> "Generate the provider cache for ClickUp"

Claude will query the ClickUp workspace, resolve all entity IDs (Space, Folders, Lists, Docs, Pages), and write them to `claude-memory/PROVIDER_CACHE.md`. This file is gitignored and local to your machine.

**What this gives you**: On subsequent sessions, Claude reads the cache on startup and has instant access to all ClickUp IDs without making MCP search calls.

**If you skip this step**: Claude will still work — it will generate the cache automatically the first time it needs to access ClickUp entities (cold start).

### 8. Test it

Open Claude Code and say:

> "Read CONVENTIONS from ClickUp and tell me what the distillation protocol says."

Claude should follow the boot sequence: read CLAUDE.md → CONFIG.md → CONVENTIONS via MCP → PROVIDER_CACHE (or generate it) → Ready.

## Multi-User Setup (optional)

If multiple people use Claude Code on the same codebase and ClickUp workspace:

### 1. Set current_user in CONFIG.md

Each user sets their identity in their local `CONFIG.md` (which is gitignored):

```markdown
## User
current_user: Eugenio
```

The value must be resolvable by the ClickUp MCP server's `clickup_resolve_assignees` tool (display name or email).

### 2. Project ownership via native Assignee

Project ownership is tracked via the **native Assignee** field on Project Index tasks (not a custom field). When `current_user` is set, Claude:
- **Creates** new tasks with Assignee = `current_user`
- **Filters** the Project Index by Assignee when listing "my projects"
- **Can read** all projects regardless of owner (libre acceso)

### 3. Create per-user Views (recommended)

On the `Project Index` List, create a filtered View for each user:
- View name: `{username}'s Projects`
- Filter: Assignee = {username}

This keeps the ClickUp UI clean. Claude filters programmatically regardless.

### Migration from single-user

If you already have projects without assignees:
1. Set `current_user` in your `CONFIG.md`
2. Assign existing Project Index tasks to the appropriate users
3. No structural changes needed — project Docs stay in the same location

## Access Control

| Framework document | ClickUp permission |
|---|---|
| CREDENTIALS | Restricted to workspace admins |
| All other documents | Normal workspace member access |

ClickUp handles permissions natively at the Space, Folder, or Doc level.
