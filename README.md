# claude-memory-framework

**A persistent memory layer for Claude Code that survives across sessions.**

Claude Code is powerful, but it forgets everything between sessions and loses context in long ones. This framework fixes both problems with a structured system of `.md` files that Claude reads on startup and updates before closing.

```
Session 1                    Session 2                    Session 3
─────────                    ─────────                    ─────────
Read .md → Work →            Read .md → Work →            Read .md → Work →
Update .md                   Update .md                   Update .md
```

**Architected by [Eugenio Ruiz de la Cuesta](https://www.linkedin.com/in/eugenio-ruiz-de-la-cuesta-fdez/) (orchestrating Claude Opus 4.6)** · v0.1 · [Unlicense](LICENSE)
[@guachoxx](https://github.com/guachoxx)

---

## Why

| Problem | Without the framework | With the framework |
|---------|----------------------|-------------------|
| New session | Claude starts from zero | Claude reads `CURRENT_STATUS.md`, resumes in 30 seconds |
| Long session | Context fills up, hallucinations start | Distillation protocol saves progress before context runs out |
| Technical decisions | Lost in conversation transcripts | Persisted in structured `.md` files |
| Onboarding Claude to your code | Re-explain every time | Module `CLAUDE.md` files in `src/` provide instant context |

## Key Features

- **3-layer memory architecture** — Root index → reference docs → per-module context. Claude loads only what it needs.
- **Session distillation protocol** — Systematic process to save progress before context runs out. Triggered automatically or on demand.
- **5 project documents** — Analysis, Plan, Technical Report, Status, Changelog — each with a clear audience and lifecycle.
- **Lite mode** — Small projects use just 2 files. Promotes to full structure if it grows.
- **Stack-agnostic** — Works with any language, framework, or toolchain.
- **Zero dependencies** — Just `.md` files. No plugins, no extensions, no build steps.

## Documentation

| Document | What it covers |
|----------|---------------|
| **[QUICKSTART.md](QUICKSTART.md)** | **5-minute setup guide — start here** |
| [HUMANS_START_HERE.md](HUMANS_START_HERE.md) | Full documentation: architecture, workflow, rules, adaptation guide |
| [CONVENTIONS.md](CONVENTIONS.md) | Framework rules, document templates, distillation protocol |

## File Structure

```
your-project/
├── CLAUDE.md                     ← Index (Claude reads this first)
├── claude-memory/
│   ├── CONVENTIONS.md            ← Framework rules
│   ├── ARCHITECTURE.md           ← System architecture
│   ├── CREDENTIALS.md            ← Credentials (.gitignore)
│   ├── BUILD_COMMANDS.md         ← Build/deploy commands
│   ├── TESTING_METHODOLOGY.md    ← How to test
│   ├── LESSONS_LEARNED.md        ← Mistakes not to repeat
│   └── projects/
│       └── {project-name}/
│           ├── CURRENT_STATUS.md ← Where we are + next step
│           ├── TECHNICAL_ANALYSIS.md
│           ├── PLAN.md
│           ├── CHANGELOG.md
│           └── TECHNICAL_REPORT.md
└── src/
    └── {module}/
        └── CLAUDE.md             ← Local module context
```

## How It Works

1. **Session starts** — Claude reads root `CLAUDE.md` (~70 lines), sees active projects, reads `CURRENT_STATUS.md` of the relevant one. Oriented in ≤30 seconds.

2. **During work** — Claude reads deeper files only as needed: analysis docs, plan, module context. No wasted tokens on unnecessary context.

3. **Session ends** — The distillation protocol kicks in. Claude updates the relevant memory files based on the current project phase. Say **"Consolidate"** or Claude proposes it automatically.

4. **Next session** — Starts clean. Reads only distilled `.md` files. Never depends on conversation transcripts.

## Contributing

Issues and PRs welcome. The framework is designed to be forked and adapted — if you find patterns that work for your stack, consider contributing them back.

## License

[Unlicense](LICENSE)
