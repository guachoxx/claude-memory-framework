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

You don't need to do anything special. Claude reads the root index on startup, sees the active projects, and reads the CURRENT_STATUS of the relevant project. In 30 seconds it knows where to resume.

If you want to be explicit:

> "Read CLAUDE.md and tell me the current status of the [NAME] project and what the next step is."

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

What Claude updates depends on the project phase:

| Phase | Primary document updated |
|-------|---------------------|
| Analysis | TECHNICAL_ANALYSIS — findings, constraints, structure |
| Planning | PLAN — phases, files, order |
| Development | TECHNICAL_REPORT — what was built, technical decisions |
| Always | CURRENT_STATUS — concrete next step |

---

## The 5 Project Documents

For large projects (3+ phases), the framework uses 5 documents with differentiated roles:

```
TECHNICAL_ANALYSIS          PLAN                       TECHNICAL_REPORT
───────────────────       ───────                    ──────────────────
What are we working with? → What are we going to do? → What have we built?

For Claude                  For Claude                 For the team
Discarded on close          Discarded on close         Survives as documentation
```

- **CURRENT_STATUS** — Always present. Current state, what was done, what's left, concrete next step and verification condition.
- **TECHNICAL_ANALYSIS** — Research: existing code involved, constraints, data structures, prior design decisions. Claude consults it to avoid repeating analysis.
- **PLAN** — Strategy: ordered phases, files to create/modify, dependencies. Claude consults it to know what to do next.
- **TECHNICAL_REPORT** — Result: technical documentation of what was built. Aimed at the engineering team. It's the only document that survives project closure.
- **CHANGELOG** — Chronological record of code changes.

### Small Projects (Lite Mode)

For fixes, scoped refactors, or tasks spanning a few sessions: only CURRENT_STATUS + CHANGELOG. Analysis and plan go as sections inside CURRENT_STATUS. If the project grows, it gets promoted to full structure.

---

## Module Context

Each code module can have its own context document with local information:

```markdown
# My Module

## What this is
- What it does in 2-3 lines

## Dependencies
- Framework, libraries, references (especially important in legacy stacks)

## Key patterns
- Patterns Claude should follow when working here

## Files
- List of files with a one-line description

## Watch out
- Pitfalls, common mistakes, things NOT to do
```

Maximum ~50 lines. Direct, imperative language. Only what Claude needs to avoid mistakes when touching that code.

> **Note**: Module context documents always live on disk alongside the code, regardless of provider. They are code context, not project context.

---

## Key Phrases Claude Understands

| You say | Claude does |
|---------|------------|
| "Consolidate" / "Distill" / "Save progress" | Updates all memory documents according to the current phase |
| "Create a project called X" | Creates the project container and initial documents |
| "Where are we with X?" | Reads the project's CURRENT_STATUS |
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

---

## Adapting to Your Project

This framework was designed to be stack-agnostic and provider-agnostic. To adapt it:

1. **Choose a provider** — See `providers/` for available options. If yours isn't there, create one following the guide in `providers/README.md`.
2. **Root index** — Rewrite it with your system's overview.
3. **CONVENTIONS** — Review the rules. Most are language-agnostic. Adjust examples if needed.
4. **Module context** — Adapt the template. The "Dependencies" section is especially valuable in legacy stacks or frameworks with constrained versions.
5. **CREDENTIALS** — Fill in with your environments and credentials. Ensure it has restricted access (`.gitignore` for files, restricted permissions for external providers).
6. **Project documents** — The 5 documents (TECHNICAL_ANALYSIS, PLAN, TECHNICAL_REPORT, CURRENT_STATUS, CHANGELOG) apply to any type of project: API integration, refactor, new feature, migration, performance optimization.

---

## Scalability

The framework works well for 1-3 developers working with Claude. For larger teams, the project index and changelogs may generate conflicts — in that case, consider per-project changelogs (already the default) and an index with per-entry ownership.

## Quick Document Reference

| Document | For whom | Purpose |
|------|----------|---------|
| Root index | Claude | Navigation, first read |
| CONVENTIONS | Claude + Humans | Framework rules |
| Project index | Claude + Humans | Project list and status |
| CURRENT_STATUS | Claude | Where we are, what's next |
| TECHNICAL_ANALYSIS | Claude | Prior research |
| PLAN | Claude | Implementation plan |
| TECHNICAL_REPORT | Team | Deliverable technical documentation |
| CHANGELOG | Claude + Team | Change history |
| LESSONS_LEARNED | Claude | Mistakes not to repeat |
| Module context | Claude | Local module context |
| CREDENTIALS | Claude | Credentials (restricted access) |
