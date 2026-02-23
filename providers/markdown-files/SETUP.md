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
    ├── CONVENTIONS.md                 ← Framework rules
    ├── ARCHITECTURE.md                ← Empty template
    ├── CREDENTIALS.md                 ← Empty template
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

## How Claude Accesses Documents

Claude Code reads and writes `.md` files directly via its native file tools. No additional configuration is needed.

## Directory Conventions

| Framework concept | Markdown-files implementation |
|---|---|
| Root index | `CLAUDE.md` at project root |
| Reference documents | `claude-memory/*.md` |
| Project container | `claude-memory/projects/{project-name}/` |
| Project index | `claude-memory/projects/_INDEX.md` |
| Module context | `src/{module}/CLAUDE.md` |
| Logs | `claude-memory/logs/` (.gitignored) |
| Temporary files | `claude-memory/temp/` (.gitignored) |
| Credentials | `claude-memory/CREDENTIALS.md` (.gitignored) |
