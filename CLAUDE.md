# Claude Memory Index

## System Overview

## Configuration

<!-- User identity for multi-user project separation.
     Claude resolves current_user in this order:
     1. .user file at project root (gitignored) — preferred for teams
     2. current_user field below — fine for solo use
     When empty/absent, the framework operates in single-user mode. -->
current_user:

## Quick Navigation

| I need to...                       | Read                                   |
| ---------------------------------- | -------------------------------------- |
| Understand the architecture        | ARCHITECTURE document                  |
| Credentials, endpoints, connection | CREDENTIALS document                   |
| Build commands                     | BUILD_COMMANDS document                |
| Testing methodology                | TESTING_METHODOLOGY document           |
| Conventions and memory rules       | `CONVENTIONS.md`                       |
| Reusable technical lessons         | LESSONS_LEARNED document               |
| See active projects and status     | Project index                          |
| Available providers                | `providers/`                           |
| Provider-specific setup            | `providers/{provider}/SETUP.md`        |
| Full documentation for humans      | `HUMANS_START_HERE.md`                 |
| Multi-user setup                   | `CONVENTIONS.md` → "Multi-User Mode"   |
| Provider cache (external providers)| `claude-memory/PROVIDER_CACHE.md`      |

> **Provider note**: Where each document lives depends on the configured provider. See `providers/` for details. For the default markdown-files provider, see `providers/markdown-files/MAPPING.md`.

## Per-module Context
Module context documents describe the code **as it is**: patterns, pitfalls, key files. Consult them when working on that module. These always live on disk alongside the code, regardless of provider.

## Session Distillation Protocol (CRITICAL)
Context runs out. Before that happens, **Claude MUST distill** the work into memory documents. The user can ask "consolidate" or "distill the session" at any time:

1. **SPECIFICATIONS** → Update if requirements or acceptance criteria changed
2. **TECHNICAL_ANALYSIS** → Enrich if there were new findings (existing code, constraints, data structures)
3. **PLAN** → Update if the phase plan changed or new dependencies were discovered
4. **TECHNICAL_REPORT** → Document what was built (for the engineering team)
5. **TESTING** → Record test scenarios, results, and verification against specifications
6. **CURRENT_STATUS** → Update ALWAYS: what was done, what's left, concrete next step
7. **CHANGELOG** → Record code changes made
8. **LESSONS_LEARNED** → Add reusable lessons across projects
9. **Module context** → Update if the module's code changed
10. **PROVIDER_CACHE** → Update if new external entities were created (projects, documents)

**Rule**: If the user doesn't ask, Claude must propose it proactively when it detects the context is filling up or at the end of a significant block of work. The next session ONLY reads distilled documents — never conversation transcripts.

> **Multi-user**: Distillation targets only the `current_user`'s project namespace. Shared documents (LESSONS_LEARNED, module context) are updated normally regardless of user.

See `CONVENTIONS.md` → "Session Distillation Protocol" for the full process.

## Memory Persistence Rules
1. **ALWAYS** distill at the end of every session or significant block of work
2. **ALWAYS** persist findings in the corresponding document (see table in CONVENTIONS.md)
3. **NEVER** duplicate information — use cross-references between documents
4. **NEVER** rely on conversation transcripts as memory — all valuable information must be in the structured documents
5. **ALWAYS** scope project operations to the current user's namespace when `current_user` is configured
6. **ALWAYS** update the provider cache when creating new external entities (projects, documents) in hybrid providers
