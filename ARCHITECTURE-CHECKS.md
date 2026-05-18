# Architecture checks

Human-readable index of what the **swiftui-architecture** skill enforces. Details live in [`skills/swiftui-architecture/references/`](skills/swiftui-architecture/references/).

## Layer responsibilities

| Layer | May do | Must not do |
|-------|--------|-------------|
| **Factory** | Build manager implementations; availability/config when choosing impls; compose UseCase → ViewModel (or full feature graph) for `App` | Business logic; runtime feature behavior |
| **Manager (Common)** | Reusable infra: logging, navigation, networking, storage | Feature-specific business rules |
| **Manager (Feature)** | App-specific system APIs (camera, files, ML, etc.) | UI state; orchestration across features |
| **UseCase** | Business logic; async orchestration; call manager protocols | Touch SwiftUI; depend on ViewModels |
| **ViewModel** | UI state; `ViewState` enum; call use case protocols | Access managers directly |
| **View** | Layout; forward actions to ViewModel | Business logic; UseCase/Manager access |

## Dependency flow

| Check | Rule |
|-------|------|
| Construction | `App` / Factory wire Manager, UseCase, ViewModel (Factory optional for simple features) |
| Runtime direction | View → ViewModel → UseCase → Manager only (no upward deps) |
| Protocols | Depend on protocol types in initializers, not concrete types |
| Injection | Pass dependencies through `init`; composition root in `App` / factories |
| Circles | No upward references between layers |

## ViewModel

| Check | Rule |
|-------|------|
| Isolation | `@MainActor` on ViewModels |
| Observation | `@Observable` (Swift Observation) |
| State | Nested `ViewState` enum with associated values, not many boolean flags |
| Dependencies | Primary dependency is one use case protocol |

## View

| Check | Rule |
|-------|------|
| Dependencies | ViewModel only |
| Async | `Task { await viewModel.method() }` from UI actions |
| Logic | No use case or manager calls in `View` |

## New feature checklist

- [ ] Factory (if impl selection / availability **or** feature assembly for `App`; else wire in `App`)
- [ ] Manager protocol + implementation (if new infrastructure)
- [ ] `[Feature]UseCaseProtocol` + `[Feature]UseCase`
- [ ] `[Feature]ViewModel` + `ViewState`
- [ ] `[Feature]View`
- [ ] Models under feature `Model/` or manager `Model/`
- [ ] Wire in `App` or factory (composition root)
- [ ] Reusable UI → `Component/` only when shared

## Anti-patterns

| Anti-pattern | Fix |
|--------------|-----|
| ViewModel holds `FileSystemManager()` | Inject `ItemListUseCaseProtocol` |
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
