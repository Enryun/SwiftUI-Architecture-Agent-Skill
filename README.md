# SwiftUI Architecture — Agent Skills

Open-source Agent Skills for structuring **SwiftUI iOS/macOS apps** with a layered stack:

**Runtime when those layers are present: View → ViewModel → UseCase → Manager**

**Construction: App composes and shares dependencies; factories create individual instances.**

Protocol-first dependency injection, feature-based folders, and a recommend-first scaffold workflow for new projects and new features.

## When to use

- Starting a new SwiftUI app with multiple features
- Adding a feature module (screen + business logic + infrastructure)
- Reviewing layer boundaries or dependency direction

For features with real business rules, prefer `View → ViewModel → focused UseCase(s)`. When a UseCase crosses an infrastructure boundary, it depends on a narrow Manager protocol; features without infrastructure do not need a Manager layer. Add factories only when construction complexity warrants them. Simple presentation-only screens may remain simple MVVM.

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
│   └── references/            # Detailed specs and complete example
└── templates/
    └── architecture.mdc       # Optional Cursor rule template
```

## How it works

1. **Propose** — Agent outputs folder tree, protocols, and dependency graph (no files yet).
2. **Approve** — You confirm or adjust the plan.
3. **Scaffold** — Agent creates files following layer rules.

See [`skills/swiftui-architecture/SKILL.md`](skills/swiftui-architecture/SKILL.md) for the full workflow.

## Architecture at a glance

**One main feature View → exactly one owning ViewModel.** When business operations need a separate boundary, that ViewModel may use one or more focused UseCases; a simple feature may use none.

**Construction** (`App` owns composition and shared lifetimes):

```
App
  ├─ Factory.createManager(...) → manager
  ├─ Factory.createUseCase(manager: manager) → useCase
  └─ Factory.makeViewModel(useCase: useCase) → viewModel
```

**Runtime** (each user action):

```
View → ViewModel → UseCase → Manager
```

| Layer | Role |
|-------|------|
| **Factory** | Create one Manager, UseCase, or ViewModel per method from supplied dependencies; select platform implementations when needed. |
| **Manager** | Infrastructure (Common + Feature), protocol-based |
| **UseCase** | Focused business responsibility; calls manager protocols directly, never other UseCases |
| **ViewModel** | `@MainActor` UI state; `@Observable` where supported or `ObservableObject` for older targets; optional `ViewState`; focused UseCase protocols when needed, no other ViewModels |
| **Feature View** | Presents one feature through its one owning ViewModel |
| **Composition View** | Assembles independent feature views using child ViewModels supplied by `App` |
| **Component** | Visual UI receiving values, bindings, and action closures |

`AppFactory` above is an example name. Start with one factory and split into focused domain/platform factories when current complexity warrants it. `App` wires the graph and reuses shared manager instances. Factory methods must not hide complete feature/application graphs or contain business logic.

Navigation is presentation state scoped to each independent stack/window. Views may access it as an explicit exception to infrastructure-manager boundaries; ViewModels and UseCases may not. See [navigation and lifetimes](skills/swiftui-architecture/references/navigation.md).

Check the app's minimum OS versions and Swift toolchain before adopting example APIs. The [macOS 14+ example](skills/swiftui-architecture/references/examples.md) uses Observation; [older-OS guidance](skills/swiftui-architecture/references/observation.md) covers compatible ViewModel observation. Navigation examples also need an availability strategy on targets older than iOS 16/macOS 13.

Details: [`ARCHITECTURE-CHECKS.md`](ARCHITECTURE-CHECKS.md) and [`skills/swiftui-architecture/references/`](skills/swiftui-architecture/references/).

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md).

## License

MIT — see [LICENSE](LICENSE).
