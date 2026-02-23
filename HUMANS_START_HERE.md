# claude-memory-framework: A working framework and persistent memory layer for Claude Code
Empowering Claude Code with long-term context and standardized workflows.

**Architected by [Eugenio Ruiz de la Cuesta](https://www.linkedin.com/in/eugenio-ruiz-de-la-cuesta-fdez/) (orchestrating Claude Opus 4.6)** · v0.2 · [Unlicense](LICENSE)
[@guachoxx](https://github.com/guachoxx)

## The Problem It Solves

Claude Code is powerful but has two fundamental limitations:

1. **Finite context window** — In long sessions, the context fills up and Claude loses track or starts hallucinating.
2. **Amnesia between sessions** — Every new session starts from scratch. Claude doesn't remember what you did yesterday.

The usual outcome: repeating context, saving conversations that need to be reprocessed later, losing technical decisions, and Claude suggesting things that were already discarded.

## The Solution

A structured set of documents that Claude reads on startup and updates before closing. Each session starts by reading clean documents and ends by updating them. Claude never relies on past conversation transcripts or its default memory (MEMORY.md).

```
Session 1                    Session 2                    Session 3
─────────                    ─────────                    ─────────
Read docs → Work →           Read docs → Work →           Read docs → Work →
Update docs                  Update docs                  Update docs
```

**Where** those documents live depends on your chosen **provider**: local `.md` files, ClickUp, Notion, or any other platform. The methodology stays the same.

---

## Providers

A **provider** is the persistence backend for the framework's documents. See [providers/](providers/) for available options:

| Provider | Documents stored in | Best for |
|---|---|---|
| [markdown-files](providers/markdown-files/) | Local `.md` files in the repo | Solo developers, simple setups |
| [clickup](providers/clickup/) | ClickUp Docs and Tasks | Teams using ClickUp |

Some providers are **hybrid**: the root index (`CLAUDE.md`) and module context documents always live on disk (Claude Code reads them on startup), while project and reference documents live in the external platform.

---

## Three Layers of Information

| Layer | What it contains | When it's read |
|-------|-----------------|----------------|
| **Root index** | Navigation (~70 lines) | Always, on startup |
| **Reference documents** | Architecture, conventions, credentials, build commands, testing, lessons | When Claude needs detail |
| **Module context** | Patterns and pitfalls per code module | When Claude works on that module |

---

## Getting Started

1. Choose a provider from `providers/`
2. Follow the provider's `SETUP.md`
3. Read the rest of this guide for the workflow and rules

---

## Daily Workflow

### Starting a session

You don't need to do anything special. Claude reads the root index on startup, then CONFIG.md for your identity and provider settings, then CONVENTIONS for the framework rules, then PROVIDER_CACHE for cached entity IDs (external providers only), and then reads the CURRENT_STATUS of the relevant project. In 30 seconds it knows where to resume.

If you want to be explicit:

> "Read CLAUDE.md and tell me the current status of the [NAME] project and what the next step is."

> **External providers (ClickUp, etc.)**: Claude also reads `claude-memory/PROVIDER_CACHE.md` on startup if it exists. This file caches entity IDs so Claude doesn't need to search for them each session. If the file is missing, Claude generates it automatically. You never need to edit this file.

If you haven't worked with Claude for a while (more than 48h), Claude will ask if the status is still valid before assuming anything.

### During the session

Work as usual. If Claude needs context from a specific module:

> "Read the module context for src/path/to/module."

If you notice Claude starting to hallucinate or lose track:

> "You're losing context. Read CONVENTIONS and CURRENT_STATUS to reorient."

### Closing the session (distillation)

This is the most important part of the framework. Before closing, tell Claude:

> "Consolidate the session: update CURRENT_STATUS with what we've done and define the Next Step. If there were technical changes, update TECHNICAL_REPORT."

These also work: **"Consolidate"** / **"Distill"** / **"Save progress"**

If you forget, Claude should propose it when it detects the context is filling up.

> **Multi-user**: Distillation updates only your projects. Shared documents (LESSONS_LEARNED, module context) are updated for everyone.

What Claude updates depends on the project phase:

| Phase | Primary document updated |
|-------|---------------------|
| Analysis | TECHNICAL_ANALYSIS — findings, constraints, structure |
| Planning | PLAN — phases, files, order |
| Development | TECHNICAL_REPORT — what was built, technical decisions |
| Testing | TESTING — test scenarios, results, verification |
| Always | CURRENT_STATUS — concrete next step |

---

## The 7 Project Documents

For large projects (3+ phases), the framework uses 7 documents with differentiated roles:

```
SPECIFICATIONS → TECHNICAL_ANALYSIS → PLAN          → TECHNICAL_REPORT → TESTING
──────────────   ──────────────────   ──────          ──────────────────   ───────
What is asked?   What exists?         What to do?     What was built?      Does it work?

For team         For Claude           For Claude      For team             For team
Survives         Discarded on close   Discarded       Survives             Survives
```

- **CURRENT_STATUS** — Always present. Current state, what was done, what's left, concrete next step and verification condition.
- **SPECIFICATIONS** — Requirements: what the feature/change must do, acceptance criteria, constraints, out-of-scope. Created early, approved before planning starts. Survives project closure as a reference.
- **TECHNICAL_ANALYSIS** — Research: existing code involved, constraints, data structures, prior design decisions. Claude consults it to avoid repeating analysis.
- **PLAN** — Strategy: ordered phases, files to create/modify, dependencies. Claude consults it to know what to do next.
- **CHANGELOG** — Chronological record of code changes.
- **TECHNICAL_REPORT** — Result: technical documentation of what was built. Aimed at the engineering team. Survives project closure as documentation.
- **TESTING** — Test plan, test cases, and results. Created during development, updated during the testing phase. Survives project closure as a verification record.

### Small Projects (Lite Mode)

For fixes, scoped refactors, or tasks spanning a few sessions: only CURRENT_STATUS + CHANGELOG. Specs can be a section inside CURRENT_STATUS. If the project grows, it gets promoted to full structure.

---

## Module Context

Each code module can have its own context document (`{module}/CLAUDE.md`, ~50 lines max) with: what it does, dependencies, key patterns, files, and pitfalls. Direct, imperative language. Only what Claude needs to avoid mistakes when touching that code.

These always live on disk alongside the code, regardless of provider. See `CONVENTIONS.md` → "Layered Memory Architecture" for the full template.

---

## Key Phrases Claude Understands

| You say | Claude does |
|---------|------------|
| "Consolidate" / "Distill" / "Save progress" | Updates all memory documents according to the current phase |
| "Create a project called X" | Creates the project container in your namespace and initial documents |
| "Where are we with X?" | Reads the project's CURRENT_STATUS |
| "Show me maria's projects" | Reads maria's projects from the project index (read-only) |
| "Close project X" | Archives TECHNICAL_REPORT, cleans up the project container, updates the index |

---

## Key Framework Rules

1. **30-second rule** — Claude must be able to orient itself by reading the root index in ≤30 seconds.
2. **No code without a plan** — Not a single line of code is written until the Plan is approved (in projects with full structure).
3. **No duplication** — Each piece of data lives in one place only. Other documents point to it with references.
4. **No transcripts** — Never save conversations to "remember". Everything is distilled to structured documents.
5. **Cross-referencing** — Before creating something new, Claude reads the existing equivalent first as a reference.
6. **Staleness** — If a CURRENT_STATUS is more than 48h old, Claude asks before assuming it's valid.
7. **Module context is mandatory** — If the core logic of a module is modified, updating its context document is mandatory.
8. **LESSONS_LEARNED as incubator** — When a lesson matures, it moves to the corresponding module's context document and is removed from the general document.
9. **Ownership** — When `current_user` is set, Claude assigns ownership to new projects and can filter by owner. All projects are freely accessible. See "Multi-User Setup" below.

---

## Adapting to Your Project

This framework was designed to be stack-agnostic and provider-agnostic. To adapt it:

1. **Choose a provider** — See `providers/` for available options. If yours isn't there, create one following the guide in `providers/README.md`. External providers benefit from a Provider Cache that Claude generates automatically — see your provider's SETUP.md.
2. **Root index** — Rewrite it with your system's overview.
3. **CONVENTIONS** — Review the rules. Most are language-agnostic. Adjust examples if needed.
4. **Module context** — Adapt the template. The "Dependencies" section is especially valuable in legacy stacks or frameworks with constrained versions.
5. **CREDENTIALS** — Fill in with your environments and credentials. Ensure it has restricted access (`.gitignore` for files, restricted permissions for external providers).
6. **Project documents** — The 7 documents (CURRENT_STATUS, SPECIFICATIONS, TECHNICAL_ANALYSIS, PLAN, CHANGELOG, TECHNICAL_REPORT, TESTING) apply to any type of project: API integration, refactor, new feature, migration, performance optimization.
7. **Multi-user** (optional) — If your team has multiple Claude Code users, set `current_user` for each person. See "Multi-User Setup" below.

---

## Multi-User Setup

If multiple people use Claude Code on the same codebase, the framework supports per-user project ownership.

### How to enable it

Each user sets their identity in `claude-memory/CONFIG.md` (which is gitignored):

```
## User
current_user: Eugenio
```

### What's shared vs. owned

| Concept | Scope |
|---|---|
| Projects, project index entries | **Owned** per user (freely accessible to all) |
| Reference documents (ARCHITECTURE, CONVENTIONS, etc.) | **Shared** across team |
| Module context | **Shared** (code-level) |
| LESSONS_LEARNED | **Shared** across team |

### How it works

When `current_user` is set, Claude assigns ownership to new projects and can filter by owner:
- **Create**: `"Create a project called auth-refactor"` → creates with ownership = current user
- **Read**: Claude can read any project regardless of owner
- **Distill**: Only your project documents are updated (shared docs like LESSONS_LEARNED update normally)
- **Cross-user read**: `"Show me maria's projects"` → reads maria's projects from the project index

See CONVENTIONS → "Multi-User Mode" for the full rules.

---

## Scalability

The framework supports multi-user project ownership out of the box. Set `current_user` in `claude-memory/CONFIG.md` to enable per-user project ownership. Each user's projects are identified by owner while reference documents and module context remain shared. See "Multi-User Setup" above.

## Quick Document Reference

| Document | For whom | Purpose |
|------|----------|---------|
| Root index | Claude | Navigation, first read |
| CONVENTIONS | Claude + Humans | Framework rules |
| Project index | Claude + Humans | Project list and status |
| CURRENT_STATUS | Claude | Where we are, what's next |
| SPECIFICATIONS | Team + Claude | Requirements and acceptance criteria |
| TECHNICAL_ANALYSIS | Claude | Prior research |
| PLAN | Claude | Implementation plan |
| CHANGELOG | Claude + Team | Change history |
| TECHNICAL_REPORT | Team | Deliverable technical documentation |
| TESTING | Team + Claude | Test plan, cases, and results |
| LESSONS_LEARNED | Claude | Mistakes not to repeat |
| Module context | Claude | Local module context |
| CREDENTIALS | Claude | Credentials (restricted access) |
