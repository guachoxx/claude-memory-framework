# Claude Memory Index

## System Overview

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
| External documentation             | `docs/`                                |

> **Provider note**: Where each document lives depends on the configured provider. See `providers/` for details. For the default markdown-files provider, see `providers/markdown-files/MAPPING.md`.

## Per-module Context
Module context documents describe the code **as it is**: patterns, pitfalls, key files. Consult them when working on that module. These always live on disk alongside the code, regardless of provider.

## Session Distillation Protocol (CRITICAL)
Context runs out. Before that happens, **Claude MUST distill** the work into memory documents. The user can ask "consolidate" or "distill the session" at any time:

1. **TECHNICAL_ANALYSIS** → Enrich if there were new findings (existing code, constraints, data structures)
2. **PLAN** → Update if the phase plan changed or new dependencies were discovered
3. **TECHNICAL_REPORT** → Document what was built (for the engineering team)
4. **CURRENT_STATUS** → Update ALWAYS: what was done, what's left, concrete next step
5. **CHANGELOG** → Record code changes made
6. **LESSONS_LEARNED** → Add reusable lessons across projects
7. **Module context** → Update if the module's code changed

**Rule**: If the user doesn't ask, Claude must propose it proactively when it detects the context is filling up or at the end of a significant block of work. The next session ONLY reads distilled documents — never conversation transcripts.

See `CONVENTIONS.md` → "Session Distillation Protocol" for the full process.

## Memory Persistence Rules
1. **ALWAYS** distill at the end of every session or significant block of work
2. **ALWAYS** persist findings in the corresponding document (see table in CONVENTIONS.md)
3. **NEVER** duplicate information — use cross-references between documents
4. **NEVER** rely on conversation transcripts as memory — all valuable information must be in the structured documents
