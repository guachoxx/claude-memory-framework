# ShopFast — Claude Memory Index

## System Overview
E-commerce platform built with Node.js 20 + Express + PostgreSQL 16.
Monorepo: `apps/api` (REST API), `apps/web` (React 18 frontend), `packages/shared` (common types).
Auth currently session-based (express-session + connect-pg-simple). Migrating to JWT.

## Boot Sequence
1. Read this file (`CLAUDE.md`) — you are here
2. Read `claude-memory/CONFIG.md` — provider type, user identity
3. Read CONVENTIONS — framework operational rules
4. Read `claude-memory/PROVIDER_CACHE.md` — cached entity IDs (not needed for markdown-files provider)
5. **Read Project Index** (`claude-memory/projects/_INDEX.md`) — overview of active projects
6. Ready to work. Read a project's CURRENT STATUS only when asked.

## Quick Navigation

| I need to...                       | Read                                           |
| ---------------------------------- | ---------------------------------------------- |
| Understand the architecture        | `claude-memory/ARCHITECTURE.md`                |
| Reusable technical lessons         | `claude-memory/LESSONS_LEARNED.md`             |
| See active projects                | `claude-memory/projects/_INDEX.md`             |

## Session Distillation Protocol (CRITICAL)
Context runs out. Before that happens, Claude MUST distill the work into memory documents.
See CONVENTIONS for the full process.

## Memory Persistence Rules
1. ALWAYS distill at the end of every session
2. NEVER duplicate information — use cross-references
3. NEVER rely on conversation transcripts as memory
