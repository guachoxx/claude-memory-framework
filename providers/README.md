# Providers

## What is a Provider?

A **provider** is the persistence backend where the framework's documents are stored. The framework defines *what* information to maintain and *when* to update it — the provider defines *where* and *how* that information is persisted.

The framework is designed to be provider-agnostic. All core concepts (3-layer architecture, 7 project documents, distillation protocol, lifecycle) work the same regardless of where documents live.

## Available Providers

| Provider | Description | Best for |
|----------|-------------|----------|
| [markdown-files](markdown-files/) | Local `.md` files in the repository | Solo developers, simple setups, full offline access |
| [clickup](clickup/) | ClickUp Docs and Tasks | Teams already using ClickUp for project management |

## Hybrid Providers

Most external providers (ClickUp, Notion, etc.) are **hybrid**: some documents always live on disk, while the rest live in the external platform.

**Always on disk (regardless of provider):**
- **Root index** (`CLAUDE.md`) — Claude Code reads this file on startup. It must exist at the project root.
- **Module context** (`{module}/CLAUDE.md`) — Code documentation co-located with the code. Claude reads it natively when working on a module.

**In the external platform:**
- Reference documents (ARCHITECTURE, CONVENTIONS, BUILD_COMMANDS, etc.)
- Project documents (CURRENT_STATUS, TECHNICAL_ANALYSIS, PLAN, etc.)
- Project index

The **markdown-files** provider is the only fully local provider — everything lives on disk. All other providers are hybrid by nature.

**Provider Cache (on disk, gitignored):**
- `claude-memory/PROVIDER_CACHE.md` — Auto-generated file mapping framework document names to provider entity IDs. Eliminates redundant MCP lookups on session startup. Only used by hybrid/external providers. See `CONVENTIONS.md` → "Provider Cache" for rules.

## Choosing a Provider

Consider:
- **Where does your team already work?** — If you live in ClickUp, use the ClickUp provider. If you prefer everything in the repo, use markdown-files.
- **Collaboration needs** — External providers (ClickUp, Notion) offer real-time collaboration. Markdown files use git for collaboration.
- **Offline access** — Markdown files work offline. External providers require connectivity.
- **Claude Code access** — Claude Code can read/write local files natively. External providers require MCP servers or API integrations for Claude to access them directly.

## Provider Structure

Each provider folder contains:

| File | Purpose |
|------|---------|
| `SETUP.md` | Step-by-step instructions to configure the provider |
| `MAPPING.md` | How each framework document maps to the provider's entities |

## Multi-User Support

When `current_user` is configured in the framework, each provider must handle per-user project namespaces. Each provider's `MAPPING.md` and `SETUP.md` document:
- How per-user namespaces map to the provider's entities (directories, folders, etc.)
- How the project index handles ownership (per-user indexes, assignee fields, filtered views, etc.)
- How `current_user` maps to the provider's identity system
- The migration path from single-user to multi-user

See `CONVENTIONS.md` → "Multi-User Mode" for the framework-level rules.

## Creating a New Provider

To add support for a new platform (Notion, Linear, Jira, Google Docs, etc.):

1. Create a folder under `providers/` with the platform name (kebab-case)
2. Create `SETUP.md` with configuration instructions
3. Create `MAPPING.md` with a complete mapping of:
   - Each framework document → platform entity type
   - Each container (project folder, module context) → platform structure
   - Cross-references → how documents link to each other
   - The distillation workflow → how Claude reads/writes on the platform
4. Submit a PR

### Mapping Checklist

Your `MAPPING.md` must cover how the provider handles:

- [ ] **Layer 1**: Root index — always on disk as `CLAUDE.md`. State this explicitly in your mapping.
- [ ] **Layer 2**: Reference documents (ARCHITECTURE, CONVENTIONS, BUILD_COMMANDS, etc.)
- [ ] **Layer 3**: Module context — always on disk as `{module}/CLAUDE.md`. State this explicitly in your mapping.
- [ ] **Project containers**: How projects are organized
- [ ] **Project documents**: CURRENT_STATUS, SPECIFICATIONS, TECHNICAL_ANALYSIS, PLAN, CHANGELOG, TECHNICAL_REPORT, TESTING
- [ ] **Project index**: The equivalent of _INDEX.md
- [ ] **Cross-references**: How documents point to each other
- [ ] **Read/write operations**: How Claude accesses documents (native file tools for on-disk, MCP/API for external)
- [ ] **Version control**: How changes are tracked
- [ ] **Access control**: Credentials, permissions, sensitive documents
- [ ] **Multi-user**: How per-user project namespaces are structured, how ownership is tracked, migration path from single-user
- [ ] **Provider Cache**: Cache template for the provider's entity IDs, generation workflow, update triggers
