# claude-memory-framework

**A persistent memory layer for Claude Code that survives across sessions.**

Claude Code is powerful, but it forgets everything between sessions and loses context in long ones. This framework fixes both problems with a structured system of documents that Claude reads on startup and updates before closing.

```
Session 1                    Session 2                    Session 3
─────────                    ─────────                    ─────────
Read docs → Work →           Read docs → Work →           Read docs → Work →
Update docs                  Update docs                  Update docs
```

**Architected by [Eugenio Ruiz de la Cuesta](https://www.linkedin.com/in/eugenio-ruiz-de-la-cuesta-fdez/) (orchestrating Claude Opus 4.6)** · v0.2 · [Unlicense](LICENSE)
[@guachoxx](https://github.com/guachoxx)

---

## Why

| Problem | Without the framework | With the framework |
|---------|----------------------|-------------------|
| New session | Claude starts from zero | Claude reads CURRENT_STATUS, resumes in 30 seconds |
| Long session | Context fills up, hallucinations start | Distillation protocol saves progress before context runs out |
| Technical decisions | Lost in conversation transcripts | Persisted in structured documents |
| Onboarding Claude to your code | Re-explain every time | Module context documents provide instant context |

## Key Features

- **3-layer memory architecture** — Root index → reference documents → per-module context. Claude loads only what it needs.
- **Session distillation protocol** — Systematic process to save progress before context runs out. Triggered automatically or on demand.
- **5 project documents** — Analysis, Plan, Technical Report, Status, Changelog — each with a clear audience and lifecycle.
- **Provider-agnostic** — Documents can live as local `.md` files, in ClickUp, Notion, or any other platform. The methodology stays the same.
- **Lite mode** — Small projects use just 2 documents. Promotes to full structure if it grows.
- **Stack-agnostic** — Works with any language, framework, or toolchain.
- **Zero dependencies** — No plugins, no extensions, no build steps.

## Providers

A **provider** determines where your framework documents are stored. Choose one:

| Provider | Documents stored in | Best for |
|---|---|---|
| [markdown-files](providers/markdown-files/) | Local `.md` files in the repo | Solo developers, simple setups, full offline access |
| [clickup](providers/clickup/) | ClickUp Docs and Tasks | Teams already using ClickUp |

Want to add your own? See [providers/README.md](providers/README.md) for how to create a new provider.

## Documentation

| Document | What it covers |
|----------|---------------|
| **[QUICKSTART.md](QUICKSTART.md)** | **5-minute setup guide — start here** |
| [HUMANS_START_HERE.md](HUMANS_START_HERE.md) | Full documentation: architecture, workflow, rules, adaptation guide |
| [CONVENTIONS.md](CONVENTIONS.md) | Framework rules, document templates, distillation protocol |
| [providers/](providers/) | Provider-specific setup and mapping guides |

## How It Works

1. **Session starts** — Claude reads the root index (~70 lines), sees active projects, reads the CURRENT_STATUS of the relevant one. Oriented in ≤30 seconds.

2. **During work** — Claude reads deeper documents only as needed: analysis, plan, module context. No wasted tokens on unnecessary context.

3. **Session ends** — The distillation protocol kicks in. Claude updates the relevant memory documents based on the current project phase. Say **"Consolidate"** or Claude proposes it automatically.

4. **Next session** — Starts clean. Reads only distilled documents. Never depends on conversation transcripts.

## Contributing

Issues and PRs welcome. The framework is designed to be forked and adapted — if you find patterns that work for your stack, consider contributing them back. New provider implementations are especially welcome.

## License

[Unlicense](LICENSE)
