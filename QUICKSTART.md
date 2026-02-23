# Quickstart (5 minutes)

## 1. Choose a provider

A **provider** determines where your framework documents live. Pick one:

| Provider | Setup guide |
|---|---|
| **Local `.md` files** (recommended for getting started) | [providers/markdown-files/SETUP.md](providers/markdown-files/SETUP.md) |
| **ClickUp** | [providers/clickup/SETUP.md](providers/clickup/SETUP.md) |

> **First time?** Start with **markdown-files**. You can migrate to an external provider later.

## 2. Follow your provider's setup

Each provider has a `SETUP.md` with step-by-step instructions. The markdown-files provider is the simplest:

1. Copy framework files to your project
2. Edit root `CLAUDE.md` with your system overview
3. Update `.gitignore`

## 3. Test it

Open Claude Code at your project root and say:

> "Read CLAUDE.md and tell me what projects we have active."

Claude should navigate to the project index and report no active projects.

## 4. Create your first project

Tell Claude:

> "Create a project called `my-feature`"

Claude reads CONVENTIONS.md and creates the project structure automatically (in your chosen provider).

## 5. Work and distill

Work normally. When you're done (or the context is getting full), say:

> **"Consolidate"**

Claude updates the memory documents. Next session, it picks up right where you left off.

---

**That's it.** For the full documentation, see [HUMANS_START_HERE.md](HUMANS_START_HERE.md).
