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
    ├── Doc: "CLAUDE Index"                    ← Root index (Layer 1)
    ├── Folder: "Reference"                    ← Layer 2
    │   ├── Doc: "ARCHITECTURE"
    │   ├── Doc: "CONVENTIONS"
    │   ├── Doc: "BUILD_COMMANDS"
    │   ├── Doc: "TESTING_METHODOLOGY"
    │   ├── Doc: "CREDENTIALS"                 ← Restrict permissions
    │   └── Doc: "LESSONS_LEARNED"
    ├── Folder: "Projects"                     ← Project containers
    │   ├── List: "Project Index"              ← Equivalent to _INDEX.md
    │   └── Folder: "{project-name}"
    │       ├── Doc: "CURRENT_STATUS"
    │       ├── Doc: "TECHNICAL_ANALYSIS"
    │       ├── Doc: "PLAN"
    │       ├── Doc: "TECHNICAL_REPORT"
    │       └── Doc: "CHANGELOG"
    └── Folder: "Module Context"               ← Layer 3
        ├── Doc: "{module-name}"
        └── Doc: "{module-name}"
```

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

### 3. Create the Root Index Doc

At the Space level (not inside a folder), create a Doc named `CLAUDE Index`. This is the equivalent of root `CLAUDE.md`. It contains:
- System overview
- Navigation table with links to each ClickUp Doc
- Distillation protocol summary

### 4. Create the Projects folder

Create a Folder named `Projects`. Inside it, create a List named `Project Index` where each task represents a project with custom fields for Status, Branch, Started date, and Summary.

### 5. Configure Claude Code access

Claude Code needs the ClickUp MCP server to read/write Docs. Add to your Claude Code MCP configuration:

```json
{
  "mcpServers": {
    "clickup": {
      "command": "npx",
      "args": ["-y", "@anthropic/clickup-mcp-server"],
      "env": {
        "CLICKUP_API_TOKEN": "your-token-here"
      }
    }
  }
}
```

> **Note**: Check for the latest ClickUp MCP server package and configuration. The example above is illustrative — use the MCP server that best fits your setup.

### 6. Test it

Open Claude Code and say:

> "Read the CLAUDE Index document in ClickUp and tell me what projects we have active."

Claude should use the ClickUp MCP tools to access the document and navigate to the Project Index.

## Access Control

| Framework document | ClickUp permission |
|---|---|
| CREDENTIALS | Restricted to workspace admins |
| All other documents | Normal workspace member access |

ClickUp handles permissions natively at the Space, Folder, or Doc level.
