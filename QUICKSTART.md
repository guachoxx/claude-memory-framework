# Quickstart (5 minutes)

> **How paths work**: This repo IS the `claude-memory/` folder. When you adopt it, copy its contents into a `claude-memory/` folder at the root of your project, and place `CLAUDE.md` at the project root. All documentation refers to paths as they will appear in your target project (e.g., `claude-memory/CONVENTIONS.md`).

## 1. Copy files to your project root

```
your-project/
├── CLAUDE.md                          ← copy from this repo (place at root)
└── claude-memory/
    ├── CONVENTIONS.md                 ← copy from this repo
    ├── ARCHITECTURE.md                ← copy (empty template)
    ├── CREDENTIALS.md                 ← copy (empty template)
    ├── BUILD_COMMANDS.md              ← copy (empty template)
    ├── TESTING_METHODOLOGY.md         ← copy (empty template)
    ├── LESSONS_LEARNED.md             ← copy (empty template)
    └── projects/
        └── _INDEX.md                  ← copy from this repo
```

## 2. Edit root CLAUDE.md

Replace the `## System Overview` section with 3-5 lines about your system:

```markdown
## System Overview
E-commerce platform built with Next.js 14 + PostgreSQL + Stripe.
Monorepo with apps/web (frontend) and apps/api (backend).
Auth via NextAuth.js, deployed on Vercel.
```

That's it. The navigation table and distillation protocol are already in the template.

## 3. Add to .gitignore

Append the contents of [`.gitignore-claude-additions`](.gitignore-claude-additions) to your project's `.gitignore`. At minimum:

```
claude-memory/CREDENTIALS.md
claude-memory/logs/
claude-memory/temp/
.claude/
```

## 4. Test it

Open Claude Code at your project root and say:

> "Read CLAUDE.md and tell me what projects we have active."

Claude should navigate to `_INDEX.md` and report no active projects.

## 5. Create your first project

Tell Claude:

> "Create a project called `my-feature` in claude-memory/projects/"

Claude reads CONVENTIONS.md and creates the structure automatically.

## 6. Work and distill

Work normally. When you're done (or the context is getting full), say:

> **"Consolidate"**

Claude updates the memory files. Next session, it picks up right where you left off.

---

**That's it.** For the full documentation, see [HUMANS_START_HERE.md](HUMANS_START_HERE.md).
