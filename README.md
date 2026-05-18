# SwiftUI Architecture — Agent Skills

Open-source Agent Skills for structuring **SwiftUI iOS/macOS apps** with a layered stack:

**Factory → Manager → UseCase → ViewModel → View**

Protocol-first dependency injection, feature-based folders, and a recommend-first scaffold workflow for new projects and new features.

## When to use

- Starting a new SwiftUI app with multiple features
- Adding a feature module (screen + business logic + infrastructure)
- Reviewing layer boundaries or dependency direction

## When not to use

- Single-screen prototypes or throwaway demos (simple MVVM is enough)
- Apps using TCA, VIPER, or other stacks unless you are migrating intentionally

## Quick start

### Cursor

```bash
cp -R skills/swiftui-architecture ~/.cursor/skills/
```

Open your Xcode project and say:

> Use the **swiftui-architecture** skill to scaffold a new `[FeatureName]` feature. Propose the folder tree and types first; do not create files until I approve.

### Project-local (share with team)

```bash
mkdir -p .cursor/skills
cp -R skills/swiftui-architecture .cursor/skills/
```

Optional: copy [`templates/architecture.mdc`](templates/architecture.mdc) to `.cursor/rules/` for always-on reminders in that repo.

### Claude Code / Codex / other tools

Copy `skills/swiftui-architecture/` into your tool’s skills directory (see that tool’s Agent Skills documentation). The skill format is portable; only the install path differs.

## Included skill

| Skill | Purpose |
|-------|---------|
| `swiftui-architecture` | Layer rules, scaffold workflow, checklists, anti-patterns |

## Repository layout

```
SwiftUI-Architecture/
├── README.md
├── ARCHITECTURE-CHECKS.md     # Human-readable index of rules
├── skills/swiftui-architecture/
│   ├── SKILL.md               # Agent entrypoint
│   └── references/            # Detailed specs
└── templates/
    └── architecture.mdc       # Optional Cursor rule template
```

## How it works

1. **Propose** — Agent outputs folder tree, protocols, and dependency graph (no files yet).
2. **Approve** — You confirm or adjust the plan.
3. **Scaffold** — Agent creates files following layer rules.

See [`skills/swiftui-architecture/SKILL.md`](skills/swiftui-architecture/SKILL.md) for the full workflow.

## Architecture at a glance

```
App (composition root)
  ↓
Factory — creates managers (availability, config)
  ↓
Manager — infrastructure (Common + Feature), protocol-based
  ↓
UseCase — business logic, orchestrates managers
  ↓
ViewModel — @MainActor @Observable, ViewState enum
  ↓
View — presentation only
```

Details: [`ARCHITECTURE-CHECKS.md`](ARCHITECTURE-CHECKS.md) and [`skills/swiftui-architecture/references/`](skills/swiftui-architecture/references/).

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md).

## License

MIT — see [LICENSE](LICENSE).
