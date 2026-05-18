---
name: swiftui-architecture
description: Scaffold and extend SwiftUI apps using Factory, Manager, UseCase, ViewModel, and View layers with protocol-first dependency injection. Use when starting a new SwiftUI project, adding a feature module, reviewing layer boundaries, folder structure, or clean architecture on iOS or macOS.
---

# SwiftUI Architecture

Use this skill for **new SwiftUI apps**, **new features**, and **architecture reviews**.

## When not to use

- Single-screen prototypes → prefer simple MVVM.
- Projects committed to TCA, VIPER, or other architectures unless explicitly migrating.

## Non-negotiable rules

1. **Unidirectional dependencies:** Runtime flow is View → ViewModel → UseCase → Manager. `App`/Factory wire dependencies at construction (Factory optional when wiring in `App` is enough).
2. **Protocol-first:** Initializers take protocol types, not concrete managers/use cases.
3. **ViewModels** are `@MainActor` + `@Observable`; use a nested `ViewState` enum.
4. **ViewModels never access managers** — only use case protocols.
5. **Views never access use cases or managers** — only the view model.
6. **Factories wire, they don't decide business outcomes** — construction and composition only; no business logic.
7. **Recommend-first:** Propose folder tree + types before creating files.

## Scaffold workflow

### Phase 1 — Propose (no file creation)

1. Confirm feature name and domain (e.g. `Catalog` / `ItemList`).
2. List files to add under: `Pages/`, `Factory/`, `Manager/`, `UseCase/`, `Component/`, `Constants/`, `Utility/` — adapt roots to the project's existing layout.
3. Output:
   - Folder tree
   - Protocol names per layer
   - Dependency graph (what `App` or factory wires)
   - `ViewState` cases for the ViewModel
4. **Stop** and ask for approval.

### Phase 2 — Implement (after approval)

5. Create protocols before implementations.
6. Wire composition root in `App` or domain factory.
7. ViewModel + View last.
8. Extract shared UI to `Component/` only when used by 2+ features.

## New feature checklist (short)

- [ ] Factory — if selecting manager implementations (availability/config) **or** assembling UseCase + ViewModel for `App` (skip if wiring once in `App` is enough)
- [ ] Manager — `Interface/` + `Implementation/` (+ `Model/` if needed)
- [ ] UseCase — `[Feature]UseCaseProtocol` + `[Feature]UseCase`
- [ ] ViewModel — `[Feature]ViewModel` + `ViewState`
- [ ] View — `[Feature]View`
- [ ] DI wired at App/factory

## Layer quick reference

| Layer | Responsibility |
|-------|----------------|
| Factory | Dependency construction & composition: build managers (incl. availability/config when needed); may expose `make[Feature]ViewModel()` so `App` stays thin |
| Manager Common | Logging, navigation, networking, storage |
| Manager Feature | App-specific system/integration APIs |
| UseCase | Business logic; orchestrate manager protocols |
| ViewModel | UI state, user actions → use case |
| View | SwiftUI layout only |

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
