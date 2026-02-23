# Example: Worked End-to-End Project

This directory shows what a project looks like in practice using the **markdown-files** provider.

## Scenario

A fictional e-commerce platform (`ShopFast`) needs to refactor its authentication system from session-based to JWT. The project spans 4 phases and several sessions.

## What's here

```
example/
├── CLAUDE.md                                  ← Root index (customized for ShopFast)
└── claude-memory/
    ├── CONFIG.md                              ← User identity + provider (would be gitignored)
    ├── ARCHITECTURE.md                        ← System architecture overview
    ├── LESSONS_LEARNED.md                     ← Reusable lessons
    └── projects/
        ├── _INDEX.md                          ← Project index (one entry)
        └── auth-refactor/                     ← Project container
            ├── CURRENT_STATUS.md              ← Where we are now
            ├── SPECIFICATIONS.md              ← Requirements
            ├── TECHNICAL_ANALYSIS.md          ← What we found
            ├── PLAN.md                        ← What we're going to do
            ├── CHANGELOG.md                   ← What changed
            ├── TECHNICAL_REPORT.md            ← What we built
            └── TESTING.md                     ← How we verify it works
```

## How to read this

1. Start with `CLAUDE.md` — this is what Claude reads first
2. Then `CURRENT_STATUS.md` — see where the project stands
3. Browse the other documents to see how they complement each other

Each document is filled with realistic content showing what it looks like mid-project (after Phase 2 of 4 is complete).

> **Note**: Reference documents like CONVENTIONS.md, BUILD_COMMANDS.md, TESTING_METHODOLOGY.md, and CREDENTIALS.md are omitted for brevity. In a real project, they would contain your team's conventions, build commands, and credentials (see the templates in `claude-memory/`).
