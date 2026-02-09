# claude-memory-framework: A working framework and persistent memory layer for Claude Code
Empowering Claude Code with long-term context and standardized workflows.

**Architected by [Eugenio Ruiz de la Cuesta](https://www.linkedin.com/in/eugenio-ruiz-de-la-cuesta-fdez/) (orchestrating Claude Opus 4.6)** · v0.1 · [Unlicense](LICENSE)
[@guachoxx](https://github.com/guachoxx)

## The Problem It Solves

Claude Code is powerful but has two fundamental limitations:

1. **Finite context window** — In long sessions, the context fills up and Claude loses track or starts hallucinating.
2. **Amnesia between sessions** — Every new session starts from scratch. Claude doesn't remember what you did yesterday.

The usual outcome: repeating context, saving conversations that need to be reprocessed later, losing technical decisions, and Claude suggesting things that were already discarded.

## The Solution

A system of structured `.md` files that Claude reads on startup and updates before closing. Each session starts by reading clean documents and ends by updating them. Claude never relies on past conversation transcripts or its default memory (MEMORY.md).

```
Session 1                    Session 2                    Session 3
─────────                    ─────────                    ─────────
Read .md → Work →            Read .md → Work →            Read .md → Work →
Update .md                   Update .md                   Update .md
```

---

## File Structure

```
your-project/
├── CLAUDE.md                          ← Index (Claude reads this first)
├── claude-memory/
│   ├── CONVENTIONS.md                 ← Framework rules
│   ├── ARCHITECTURE.md                ← System architecture
│   ├── CREDENTIALS.md                 ← Credentials (.gitignore)
│   ├── BUILD_COMMANDS.md              ← Build/deploy commands
│   ├── TESTING_METHODOLOGY.md         ← How to test
│   ├── LESSONS_LEARNED.md             ← Mistakes not to repeat
│   └── projects/
│       ├── _INDEX.md                  ← Project list + status
│       ├── my-big-project/
│       │   ├── CURRENT_STATUS.md      ← Current state + next step
│       │   ├── TECHNICAL_ANALYSIS.md    ← Prior research (for Claude)
│       │   ├── PLAN.md                ← Implementation plan (for Claude)
│       │   ├── CHANGELOG.md           ← Change history
│       │   └── TECHNICAL_REPORT.md     ← Deliverable documentation (for the team)
│       └── my-quick-fix/
│           ├── CURRENT_STATUS.md      ← Analysis + plan + status (all in one)
│           └── CHANGELOG.md
└── src/
    └── my-module/
        └── CLAUDE.md                  ← Local module context
```

### Three Layers of Information

| Layer | What it contains | When it's read |
|-------|-----------------|----------------|
| Root `CLAUDE.md` | Navigation index (~70 lines) | Always, on startup |
| `claude-memory/` | Reference documents + projects | When Claude needs detail |
| `src/**/CLAUDE.md` | Patterns and pitfalls per module | When Claude works on that module |

---

## Getting Started

See **[QUICKSTART.md](QUICKSTART.md)** for the 5-minute setup guide.

---

## Daily Workflow

### Starting a session

You don't need to do anything special. Claude reads `CLAUDE.md` on startup, sees the active projects, and reads the `CURRENT_STATUS.md` of the relevant project. In 30 seconds it knows where to resume.

If you want to be explicit, copy and paste:

> "Read CLAUDE.md and tell me the current status of the [NAME] project and what the next step is."

If you haven't worked with Claude for a while (more than 48h), Claude will ask if the status is still valid before assuming anything.

### During the session

Work as usual. If Claude needs context from a specific module:

> "Read src/path/to/module/CLAUDE.md for context."

If you notice Claude starting to hallucinate or lose track:

> "You're losing context. Read CONVENTIONS.md and CURRENT_STATUS.md to reorient."

### Closing the session (distillation)

This is the most important part of the framework. Before closing, tell Claude:

> "Consolidate the session: update CURRENT_STATUS.md with what we've done and define the Next Step. If there were technical changes, update TECHNICAL_REPORT.md."

These also work: **"Consolidate"** / **"Distill"** / **"Save progress"**

If you forget, Claude should propose it when it detects the context is filling up.

What Claude updates depends on the project phase:

| Phase | Primary file updated |
|-------|---------------------|
| Analysis | `TECHNICAL_ANALYSIS.md` — findings, constraints, structure |
| Planning | `PLAN.md` — phases, files, order |
| Development | `TECHNICAL_REPORT.md` — what was built, technical decisions |
| Always | `CURRENT_STATUS.md` — concrete next step |

---

## The 5 Project Documents

For large projects (3+ phases), the framework uses 5 documents with differentiated roles:

```
TECHNICAL_ANALYSIS.md          PLAN.md                    TECHNICAL_REPORT.md
───────────────────          ───────                    ──────────────────
What are we working with? →  What are we going to do? → What have we built?

For Claude                   For Claude                 For the team
Discarded on close           Discarded on close         Survives as documentation
```

- **CURRENT_STATUS.md** — Always present. Current state, what was done, what's left, concrete next step and verification condition.
- **TECHNICAL_ANALYSIS.md** — Research: existing code involved, constraints, data structures, prior design decisions. Claude consults it to avoid repeating analysis.
- **PLAN.md** — Strategy: ordered phases, files to create/modify, dependencies. Claude consults it to know what to do next.
- **TECHNICAL_REPORT.md** — Result: technical documentation of what was built. Aimed at the engineering team. It's the only document that survives project closure.
- **CHANGELOG.md** — Chronological record of code changes.

### Small Projects (Lite Mode)

For fixes, scoped refactors, or tasks spanning a few sessions: only `CURRENT_STATUS.md` + `CHANGELOG.md`. Analysis and plan go as sections inside CURRENT_STATUS. If the project grows, it gets promoted to full structure.

---

## Module CLAUDE.md (in src/)

Each code module can have its own `CLAUDE.md` with local context:

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

---

## Key Phrases Claude Understands

| You say | Claude does |
|---------|------------|
| "Consolidate" / "Distill" / "Save progress" | Updates all memory files according to the current phase |
| "Create a project called X" | Creates the folder and initial files in `projects/` |
| "Where are we with X?" | Reads the project's CURRENT_STATUS |
| "Close project X" | Moves TECHNICAL_REPORT to docs/, cleans up the folder, updates _INDEX |

---

## Key Framework Rules

1. **30-second rule** — Claude must be able to orient itself by reading root CLAUDE.md in ≤30 seconds.
2. **No code without a plan** — Not a single line of code is written until the Plan is approved (in projects with full structure).
3. **No duplication** — Each piece of data lives in one place only. Other files point to it with paths.
4. **No transcripts** — Never save conversations as .docx to "remember". Everything is distilled to .md.
5. **Cross-referencing** — Before creating something new, Claude reads the existing equivalent first as a reference.
6. **Staleness** — If a CURRENT_STATUS is more than 48h old, Claude asks before assuming it's valid.
7. **Module CLAUDE.md is mandatory** — If the core logic of a module is modified, updating its CLAUDE.md is mandatory.
8. **LESSONS_LEARNED as incubator** — When a lesson matures, it moves to the corresponding module's CLAUDE.md and is removed from the general file.

---

## Adapting to Your Project

This framework was designed to be stack-agnostic. To adapt it:

1. **Root CLAUDE.md** — Rewrite it with your system's overview.
2. **claude-memory/CONVENTIONS.md** — Review the rules. Most are language-agnostic. Adjust examples if needed.
3. **Module CLAUDE.md** — Adapt the template. The "Dependencies" section is especially valuable in legacy stacks or frameworks with constrained versions.
4. **CREDENTIALS.md** — Fill in with your environments and credentials. Make sure it's in `.gitignore`.
5. **Project documents** — The 5 documents (TECHNICAL_ANALYSIS, PLAN, TECHNICAL_REPORT, STATUS, CHANGELOG) apply to any type of project: API integration, refactor, new feature, migration, performance optimization.

---

## Scalability

The framework works well for 1-3 developers working with Claude. For larger teams, `_INDEX.md` and `CHANGELOG.md` may generate merge conflicts — in that case, consider per-project changelogs (already the default) and an _INDEX.md with per-line ownership.

## Quick File Reference

| File | For whom | Purpose |
|------|----------|---------|
| `CLAUDE.md` (root) | Claude | Navigation index, first read |
| `CONVENTIONS.md` | Claude + Humans | Framework rules |
| `_INDEX.md` | Claude + Humans | Project list and status |
| `CURRENT_STATUS.md` | Claude | Where we are, what's next |
| `TECHNICAL_ANALYSIS.md` | Claude | Prior research |
| `PLAN.md` | Claude | Implementation plan |
| `TECHNICAL_REPORT.md` | Team | Deliverable technical documentation |
| `CHANGELOG.md` | Claude + Team | Change history |
| `LESSONS_LEARNED.md` | Claude | Mistakes not to repeat |
| `src/**/CLAUDE.md` | Claude | Local module context |
| `CREDENTIALS.md` | Claude | Credentials (do not version) |
