---
name: swiftui-architecture
description: >-
  Scaffold and extend SwiftUI apps using Factory, Manager, UseCase, ViewModel,
  and View layers with protocol-first dependency injection. Use when starting a
  new SwiftUI project, adding a feature module, reviewing layer boundaries,
  folder structure, or clean architecture on iOS or macOS.
---

# SwiftUI Architecture

Use this skill for **new SwiftUI apps**, **new features**, and **architecture reviews**.

## When not to use

- Single-screen prototypes → prefer simple MVVM.
- Projects committed to TCA, VIPER, or other architectures unless explicitly migrating.

## Architecture decision table

Use the smallest structure that keeps responsibilities clear.

| Situation | Recommended |
|----------|-------------|
| **Small screen** (simple UI state, no real business rules) | Simple MVVM: View + ViewModel |
| **Medium feature** (business logic growing, rules, orchestration) | View + ViewModel + UseCase |
| **Feature with infra** (API/storage/filesystem/device) | Manager protocol(s) + UseCase + ViewModel |
| **Multiple implementations / platform differences** (availability/config selection, complex wiring) | Select implementations in focused factories; compose dependencies in `App` |

## Non-negotiable rules

1. **Unidirectional dependencies:**
   - Runtime flow is `View → ViewModel → UseCase → Manager`.
   - `App` wires dependencies and owns shared lifetimes; factories create individual instances from supplied dependencies.
2. **Protocol-first:** Initializers take protocol types, not concrete managers/use cases.
3. **ViewModels** are `@MainActor` + `@Observable`; use a nested `ViewState` enum.
4. **ViewModels never access managers** — only use case protocols.
5. **Views never access use cases or managers** — only the view model.
6. **Factories create individual instances** — each method creates one Manager, UseCase, or ViewModel and accepts its dependencies as arguments. Start with one factory; split by domain or platform when current complexity warrants it. Names such as `Factory`, `AppFactory`, and `[Domain]Factory` are conventions, not architectural requirements. Platform implementation selection is allowed; business logic and hidden feature/application graphs are forbidden.
7. **Recommend-first:** Propose folder tree + types before creating files.

## Scaffold workflow

### Phase 1 — Propose (no file creation)

1. Confirm feature name and domain (e.g. `Catalog` / `ItemList`).
2. List files to add under:
   - `Pages/`
   - `Factory/` (factory protocols + implementations)
   - `Manager/`
   - `UseCase/`
   - `Component/`
   - `Constants/`
   - `Utility/`

   Adapt roots to the project's existing layout.
3. Output:
   - Folder tree
   - Protocol names per layer
   - Dependency graph (what `App` wires and shares)
   - `ViewState` cases for the ViewModel
4. **Stop** and ask for approval.

### Phase 2 — Implement (after approval)

5. Create protocols before implementations.
6. Wire the composition root in `App`, calling factories for individual instances.
7. ViewModel + View last.
8. Extract shared UI to `Component/` only when used by 2+ features.

## New feature checklist (short)

- [ ] Factory — reuse existing factories; add individual creation methods for new dependencies and split only at a concrete domain/platform boundary
- [ ] Manager — `Interface/` + `Implementation/` (+ `Model/` if needed)
- [ ] UseCase — `[Feature]UseCaseProtocol` + `[Feature]UseCase`
- [ ] ViewModel — `[Feature]ViewModel` + `ViewState`
- [ ] View — `[Feature]View`
- [ ] DI wired in `App`; shared manager instances created once and reused

## Layer quick reference

| Layer | Responsibility |
|-------|----------------|
| Factory | Create individual Managers, UseCases, and ViewModels from supplied dependencies. `App` composes and shares the graph |
| Manager Common | Logging, navigation, networking, storage |
| Manager Feature | App-specific system/integration APIs |
| UseCase | Business logic; orchestrate manager protocols |
| ViewModel | UI state, user actions → use case |
| View | SwiftUI layout only |

## Testing guidance

- **UseCase tests**: mock Manager protocols; verify business rules and orchestration.
- **ViewModel tests**: mock UseCase protocol; verify `ViewState` transitions and user-intent handlers.
- **View tests**: prefer previews and snapshot-style checks; keep Views thin and deterministic.

## Migration guidance

If adopting this architecture in an existing codebase:

- **Existing MVVM**: introduce a UseCase when business logic grows beyond UI state shaping.
- **ViewModel calling managers directly**: wrap manager calls behind a UseCase protocol; ViewModel depends on the UseCase only.
- **Factory migration**: keep individual creation methods in appropriately scoped factories. Move graph assembly and shared-instance ownership into `App`; replace hidden feature builders one feature at a time. Do not consolidate focused factories merely to enforce a single factory.

## Anti-patterns (reject if suggested)

```swift
// BAD — ViewModel → Manager
final class ItemListViewModel {
    private let fileManager = FileSystemManager()
}

// GOOD
final class ItemListViewModel {
    private let useCase: ItemListUseCaseProtocol
}
```

```swift
// BAD — View → UseCase
Button("Load") { Task { await useCase.load() } }

// GOOD
Button("Load") { Task { await viewModel.load() } }
```

## Additional resources

Read only what you need:

- [Layer overview](references/layer-overview.md)
- [Folder structure](references/folder-structure.md)
- [Dependency flow](references/dependency-flow.md)
- [Naming conventions](references/naming-conventions.md)
- [New feature checklist](references/new-feature-checklist.md)
- [Anti-patterns](references/anti-patterns.md)
- [Concurrency](references/concurrency.md)
- [End-to-end example](references/examples.md)
