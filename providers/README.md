# Providers

## What is a Provider?

A **provider** is the persistence backend where the framework's documents are stored. The framework defines *what* information to maintain and *when* to update it — the provider defines *where* and *how* that information is persisted.

The framework is designed to be provider-agnostic. All core concepts (3-layer architecture, 5 project documents, distillation protocol, lifecycle) work the same regardless of where documents live.

## Available Providers

| Provider | Description | Best for |
|----------|-------------|----------|
| [markdown-files](markdown-files/) | Local `.md` files in the repository | Solo developers, simple setups, full offline access |
| [clickup](clickup/) | ClickUp Docs and Tasks | Teams already using ClickUp for project management |

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

- [ ] **Layer 1**: Root index (the entry point Claude reads first)
- [ ] **Layer 2**: Reference documents (ARCHITECTURE, CONVENTIONS, BUILD_COMMANDS, etc.)
- [ ] **Layer 3**: Module context documents
- [ ] **Project containers**: How projects are organized
- [ ] **Project documents**: CURRENT_STATUS, TECHNICAL_ANALYSIS, PLAN, TECHNICAL_REPORT, CHANGELOG
- [ ] **Project index**: The equivalent of _INDEX.md
- [ ] **Cross-references**: How documents point to each other
- [ ] **Read/write operations**: How Claude accesses documents (native file access, MCP, API, etc.)
- [ ] **Version control**: How changes are tracked
- [ ] **Access control**: Credentials, permissions, sensitive documents
