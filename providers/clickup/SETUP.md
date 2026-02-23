# ClickUp Provider — Setup

> Documents are stored as ClickUp Docs within a dedicated Space. Claude accesses them via the ClickUp MCP server.

## Prerequisites

- A ClickUp workspace with permissions to create Spaces, Folders, Lists, and Docs
- A ClickUp API token (personal or app token)
- The ClickUp MCP server configured for Claude Code (see below)

## ClickUp Structure

Create the following structure in your ClickUp workspace:

```
Workspace
└── Space: "Claude Memory" (or your project name)
    ├── Doc: "CLAUDE Index"                    ← Optional: team navigation hub
    ├── Folder: "Reference"                    ← Layer 2
    │   ├── Doc: "ARCHITECTURE"
    │   ├── Doc: "CONVENTIONS"
    │   ├── Doc: "BUILD_COMMANDS"
    │   ├── Doc: "TESTING_METHODOLOGY"
    │   ├── Doc: "CREDENTIALS"                 ← Restrict permissions
    │   └── Doc: "LESSONS_LEARNED"
    └── Folder: "Projects"                     ← Project containers
        ├── List: "Project Index"              ← Equivalent to _INDEX.md
        └── Folder: "{project-name}"
            ├── Doc: "CURRENT_STATUS"
            ├── Doc: "SPECIFICATIONS"
            ├── Doc: "TECHNICAL_ANALYSIS"
            ├── Doc: "PLAN"
            ├── Doc: "CHANGELOG"
            ├── Doc: "TECHNICAL_REPORT"
            └── Doc: "TESTING"
```

> **Layer 3 (Module Context)** always lives on disk alongside the code as `{module}/CLAUDE.md`, regardless of provider. Claude Code reads these files natively. They are NOT stored in ClickUp because they are code context, not project context. See `CONVENTIONS.md` → "Layered Memory Architecture".

## Step-by-step Setup

### 1. Create the Space

Create a Space named `Claude Memory` (or embed it in your existing project Space). This is the top-level container for all framework documents.

### 2. Create the Reference folder

Inside the Space, create a Folder named `Reference`. Create one Doc for each reference document:
- ARCHITECTURE
- CONVENTIONS (paste the contents of the framework's CONVENTIONS.md)
- BUILD_COMMANDS
- TESTING_METHODOLOGY
- CREDENTIALS (restrict access — this is sensitive)
- LESSONS_LEARNED

### 3. Set up the Root Index

The root index has two parts in this hybrid provider:

**a) On disk — `CLAUDE.md` (mandatory)**

Place `CLAUDE.md` at your project root. This is what Claude reads on startup. It must include:
- System overview
- A note that reference and project documents live in ClickUp (with Space name)
- Navigation table pointing to document names (Claude uses MCP to find them)
- Distillation protocol summary

**b) In ClickUp — `CLAUDE Index` Doc (optional, for team navigation)**

At the Space level (not inside a folder), create a Doc named `CLAUDE Index` with links to all ClickUp Docs. This is useful for humans browsing the Space, but Claude's entry point is always the on-disk `CLAUDE.md`.

### 4. Create the Projects folder

Create a Folder named `Projects`. Inside it, create a List named `Project Index` where each task represents a project with custom fields for Status, Branch, Started date, and Summary.

### 5. Configure Claude Code access

Claude Code needs an MCP server to read/write ClickUp Docs. Add a ClickUp MCP server to your Claude Code configuration (`.claude/mcp.json` or global settings):

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

> **Finding an MCP server**: Search the [MCP server registry](https://github.com/modelcontextprotocol/servers) or npm for a ClickUp MCP server that supports reading and writing Docs, listing Folders, and managing List tasks. The configuration above is a template — replace `<clickup-mcp-server-package>` with the actual package name.

### How Claude interacts with ClickUp

This is a **hybrid provider**: some documents live on disk, others in ClickUp.

| What Claude does | How it does it |
|---|---|
| Read root index (`CLAUDE.md`) | Native file read (on disk) |
| Read/update reference documents (ARCHITECTURE, etc.) | MCP tools: read/write ClickUp Docs |
| Read/update project documents (CURRENT_STATUS, etc.) | MCP tools: read/write ClickUp Docs |
| Read/update project index | MCP tools: list/update tasks in ClickUp List |
| Read/update module context | Native file read/write (on disk, always) |
| Read cached entity IDs | Native file read: `claude-memory/PROVIDER_CACHE.md` (on disk) |
| Create a new project | MCP tools: create Folder + Docs + task in List |
| Close a project | MCP tools: archive Folder, update task status |

Claude uses the ClickUp MCP tools exactly like it uses file tools — the distillation protocol and lifecycle rules work identically.

### 6. Generate Provider Cache

Once the MCP server is configured and the ClickUp structure is in place, ask Claude to generate the cache:

> "Generate the provider cache for ClickUp"

Claude will query the ClickUp workspace, resolve all entity IDs (Space, Folders, Lists, Docs), and write them to `claude-memory/PROVIDER_CACHE.md`. This file is gitignored and local to your machine.

**What this gives you**: On subsequent sessions, Claude reads the cache on startup and has instant access to all ClickUp IDs without making MCP search calls. This saves time and tokens.

**If you skip this step**: Claude will still work — it will generate the cache automatically the first time it needs to access ClickUp entities. The explicit step here just front-loads the generation.

### 7. Test it

Open Claude Code and say:

> "Read the CLAUDE Index document in ClickUp and tell me what projects we have active."

Claude should use the ClickUp MCP tools to access the document and navigate to the Project Index.

## Multi-User Setup (optional)

If multiple people use Claude Code on the same codebase and ClickUp workspace:

### 1. Create a `.user` file at the project root

Each user creates this file with their ClickUp display name or email:
```
eugenio
```

Add `.user` to `.gitignore`. The value must be resolvable by the ClickUp MCP server's `clickup_resolve_assignees` tool.

### 2. Add the Owner field to Project Index

On the `Project Index` List, add a custom field:
- Name: `Owner`
- Type: Assignee (People)

### 3. Create user Folders

Inside the `Projects` Folder, create one Folder per user:
```
Projects/
├── eugenio/
├── maria/
└── ...
```

### 4. Create per-user Views (recommended)

On the `Project Index` List, create a filtered View for each user:
- Filter by: Owner = {username}
- Name: `{username}'s Projects`

### 5. Migration from single-user

If you already have project Folders directly inside `Projects`:
1. Create a Folder with your username inside `Projects`
2. Move existing project Folders into your user Folder
3. Set Owner on existing Project Index tasks

> When `current_user` is set, Claude automatically creates projects inside your user Folder and sets you as Owner on the Project Index.

## Access Control

| Framework document | ClickUp permission |
|---|---|
| CREDENTIALS | Restricted to workspace admins |
| All other documents | Normal workspace member access |

ClickUp handles permissions natively at the Space, Folder, or Doc level.
