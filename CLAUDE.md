# Claude Memory Index

## System Overview

## Quick Navigation

| I need to...                      | Read                                   |
| --------------------------------- | -------------------------------------- |
| Understand the architecture       | `claude-memory/ARCHITECTURE.md`        |
| Credentials, endpoints, connection| `claude-memory/CREDENTIALS.md`         |
| Build commands                    | `claude-memory/BUILD_COMMANDS.md`      |
| Testing methodology               | `claude-memory/TESTING_METHODOLOGY.md` |
| Conventions and memory rules      | `claude-memory/CONVENTIONS.md`         |
| Reusable technical lessons        | `claude-memory/LESSONS_LEARNED.md`     |
| See active projects and status    | `claude-memory/projects/_INDEX.md`     |
| Save and access logs              | `claude-memory/logs/`                  |
| External documentation            | `docs/`                                |

## Per-module Context
The CLAUDE.md files inside `src/` contain module-specific technical context (patterns, pitfalls, key files). Consult them when working on that module.

## Session Distillation Protocol (CRITICAL)
Context runs out. Before that happens, **Claude MUST distill** the work into memory files. The user can ask "consolidate" or "distill the session" at any time:

1. **TECHNICAL_ANALYSIS.md** → Enrich if there were new findings (existing code, constraints, data structures)
2. **PLAN.md** → Update if the phase plan changed or new dependencies were discovered
3. **TECHNICAL_REPORT.md** → Document what was built (for the engineering team)
4. **CURRENT_STATUS.md** → Update ALWAYS: what was done, what's left, concrete next step
5. **CHANGELOG.md** → Record code changes made
6. **LESSONS_LEARNED.md** → Add reusable lessons across projects
7. **Module CLAUDE.md** (src/) → Update if the module's code changed

**Rule**: If the user doesn't ask, Claude must propose it proactively when it detects the context is filling up or at the end of a significant block of work. The next session ONLY reads distilled .md files — never conversation transcripts.

See `claude-memory/CONVENTIONS.md` → "Session Distillation Protocol" for the full process.

## Memory Persistence Rules
1. **ALWAYS** distill at the end of every session or significant block of work
2. **ALWAYS** persist findings in the corresponding file (see table in CONVENTIONS.md)
3. **NEVER** duplicate information — use cross-references between files
4. **NEVER** rely on .docx transcripts as memory — all valuable information must be in the .md files
