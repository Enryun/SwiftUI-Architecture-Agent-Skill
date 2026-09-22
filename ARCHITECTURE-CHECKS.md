# Architecture checks

Human-readable index of what the **swiftui-architecture** skill enforces. Details live in [`skills/swiftui-architecture/references/`](skills/swiftui-architecture/references/).

## Layer responsibilities

| Layer | May do | Must not do |
|-------|--------|-------------|
| **Factory** | Create one Manager, UseCase, or ViewModel per method using supplied dependencies; select implementations | Business logic; hidden feature/application graphs; ownership of shared services |
| **Manager (Common)** | Reusable infra: logging, navigation, networking, storage | Feature-specific business rules |
| **Manager (Feature)** | App-specific system APIs (camera, files, ML, etc.) | UI state; orchestration across features |
| **UseCase** | Focused business logic; async orchestration; call manager protocols | Touch SwiftUI; depend on ViewModels or other UseCases |
| **ViewModel** | UI state; `ViewState` enum; call focused UseCase protocols | Depend on or call Managers or other ViewModels |
| **Feature View** | Present one feature; receive one owning ViewModel plus values, bindings, closures | Receive unrelated feature ViewModels; business operations; service creation |
| **Composition View** | Assemble feature views using child ViewModels from `App`; coordinate presentation | Call UseCases; create services; duplicate feature business state |
| **Component** | Render values; use bindings and action closures; own local visual state | Depend on feature ViewModels, UseCases, or Managers |

## Dependency flow

| Check | Rule |
|-------|------|
| Construction | `App` composes and shares instances; factories create individual dependencies |
| Runtime direction | View → ViewModel → UseCase → Manager only (no upward deps) |
| Protocols | Depend on protocol types in initializers, not concrete types |
| Injection | Pass dependencies through `init`; composition root in `App` |
| Circles | No upward references between layers |

## ViewModel

**Exactly one owning ViewModel per feature View does not restrict that ViewModel to one UseCase.** Inject multiple focused UseCase protocols when the feature needs them.

| Check | Rule |
|-------|------|
| Isolation | `@MainActor` on ViewModels |
| Observation | `@Observable` (Swift Observation) |
| State | Nested `ViewState` enum with associated values, not many boolean flags |
| Dependencies | One or more focused UseCase protocols; no other ViewModels |

## View

| Check | Rule |
|-------|------|
| Feature dependencies | Exactly one owning feature ViewModel; values, bindings, closures allowed |
| Composition dependencies | Multiple child ViewModels allowed for independent feature views |
| Component inputs | Values, bindings, and action closures; no feature ViewModels |
| Async | `Task { await viewModel.method() }` from UI actions |
| Logic | No UseCase calls, business operations, or service creation in views |
| Closures | Small, purposeful UI event/content APIs; no hidden forbidden dependencies, duplicated ViewModel actions, or unnecessary forwarding chains |

## New feature checklist

- [ ] Reuse appropriately scoped factories; each creation method accepts dependencies and creates one object
- [ ] Manager protocol + implementation (if new infrastructure)
- [ ] Focused UseCase protocol/implementation pairs; no UseCase → UseCase dependencies
- [ ] `[Feature]ViewModel` + `ViewState`
- [ ] `[Feature]View` receives exactly one owning ViewModel
- [ ] Composition views assemble independent features without an aggregate ViewModel
- [ ] Models under feature `Model/` or manager `Model/`
- [ ] Wire in `App` (composition root); create shared managers once and reuse them
- [ ] Reusable UI → `Component/` only when shared

## Anti-patterns

| Anti-pattern | Fix |
|--------------|-----|
| ViewModel holds `FileSystemManager()` | Inject `ItemListUseCaseProtocol` |
| ViewModel depends on another ViewModel | Inject the relevant focused UseCase protocols |
| UseCase depends on another UseCase | Keep the business operation in its owning UseCase and inject manager protocols directly |
| View calls use case | Call `viewModel.load()` |
| `init(fileSystem: FileSystemManager)` | `init(fileSystem: FileSystemOperations)` |
| Business logic in `View.body` | Move to UseCase / ViewModel |
| `@MainActor` on everything to silence errors | Isolate UI vs actors intentionally |

## Concurrency (summary)

| Check | Rule |
|-------|------|
| ViewModels | `@MainActor` |
| Long work | UseCase or `actor`, not ViewModel body |
| Cancellation | `Task.checkCancellation()` in long UseCase work |
| Escape hatches | Avoid `@unchecked Sendable` unless documented |

Full rules: [`references/concurrency.md`](skills/swiftui-architecture/references/concurrency.md).
