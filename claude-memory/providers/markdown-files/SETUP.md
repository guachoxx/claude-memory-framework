# Markdown Files Provider — Setup

> This is the default provider. Documents are stored as `.md` files in the project repository.

## Prerequisites

- A git repository (recommended but not required)
- Claude Code or any AI assistant that can read/write local files

## Installation

### 1. Copy framework files to your project root

```
your-project/
├── CLAUDE.md                          ← Place at project root
└── claude-memory/
    ├── CONFIG.md.example              ← Copy to CONFIG.md and fill in (gitignored)
    ├── CONVENTIONS.md                 ← Framework rules
    ├── ARCHITECTURE.md                ← Empty template
    ├── CREDENTIALS.md.example         ← Copy to CREDENTIALS.md and fill in (gitignored)
    ├── BUILD_COMMANDS.md              ← Empty template
    ├── TESTING_METHODOLOGY.md         ← Empty template
    ├── LESSONS_LEARNED.md             ← Empty template
    └── projects/
        └── _INDEX.md                  ← Project index
```

### 2. Edit root CLAUDE.md

Replace the `## System Overview` section with 3-5 lines about your system:

```markdown
## System Overview
E-commerce platform built with Next.js 14 + PostgreSQL + Stripe.
Monorepo with apps/web (frontend) and apps/api (backend).
Auth via NextAuth.js, deployed on Vercel.
```

### 3. Add to .gitignore

Append to your project's `.gitignore`:

```
claude-memory/CONFIG.md
claude-memory/CREDENTIALS.md
claude-memory/logs/
claude-memory/temp/
.claude/
```

### 4. Test it

Open Claude Code and say:

> "Read CLAUDE.md and tell me what projects we have active."

Claude should navigate to `_INDEX.md` and report no active projects.

### 5. Create your first project

> "Create a project called `my-feature` in claude-memory/projects/"

### 6. Work and distill

Work normally. When done, say **"Consolidate"**. Claude updates the memory documents.

## Multi-User Setup (optional)

If multiple people use Claude Code on the same codebase:

### 1. Set `current_user` in CONFIG.md

Each user copies `claude-memory/CONFIG.md.example` to `claude-memory/CONFIG.md` (gitignored) and sets their identity:

```markdown
# Memory Configuration
> Per-user, per-machine. This file is gitignored — do not commit.

## User
current_user: eugenio

## Provider
provider: markdown-files
```

Add `claude-memory/CONFIG.md` to `.gitignore` so each user keeps their own identity locally.

### 2. Create your user namespace

```
claude-memory/projects/
└── eugenio/
    └── _INDEX.md          ← Create with project name, status, branch columns
```

### 3. Migration from single-user

If you already have projects in the flat `claude-memory/projects/` structure:
1. Create `claude-memory/projects/{your-name}/`
2. Move existing project folders into it
3. Move `_INDEX.md` into your user folder
4. (Optional) Create a new root `_INDEX.md` as a team overview

> When `current_user` is set, Claude automatically scopes all project operations to your namespace.

## How Claude Accesses Documents

Claude Code reads and writes `.md` files directly via its native file tools. No additional configuration is needed.

## Directory Conventions

| Framework concept | Markdown-files implementation |
|---|---|
| Root index | `CLAUDE.md` at project root |
| Configuration | `claude-memory/CONFIG.md` (.gitignored) |
| Reference documents | `claude-memory/*.md` |
| Project container | `claude-memory/projects/{project-name}/` |
| Project index | `claude-memory/projects/_INDEX.md` |
| Module context | `src/{module}/CLAUDE.md` |
| Project container (multi-user) | `claude-memory/projects/{username}/{project-name}/` |
| Project index (multi-user) | `claude-memory/projects/{username}/_INDEX.md` |
| Team overview (multi-user) | `claude-memory/projects/_INDEX.md` |
| Logs | `claude-memory/logs/` (.gitignored) |
| Temporary files | `claude-memory/temp/` (.gitignored) |
| Credentials | `claude-memory/CREDENTIALS.md` (.gitignored) |
