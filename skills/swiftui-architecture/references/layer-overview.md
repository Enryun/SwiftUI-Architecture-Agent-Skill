# Layer overview

## Factory

**Purpose:** Create individual dependencies through factories. `App` is the composition root and owns sharing and lifetimes.

**Rules**

- Define factory protocols and implementations. Start with one factory; split when unrelated domain responsibilities accumulate or platform/target boundaries require separate creation code. Do not create a factory per feature without a current need.
- Each method creates one Manager, UseCase, or ViewModel and accepts its required dependencies as arguments.
- May select a platform implementation based on availability or configuration.
- No business logic, service lookup, or hidden construction of a feature/application graph.
- Do not store or recreate shared managers inside feature builders. `App` creates each shared manager once and passes that instance to its consumers.
- Do not add `makeEntireFeature()` or `makeApp()` methods. Multiple factories must follow the same individual-creation boundary.
- Mark ViewModel creation requirements and implementations `@MainActor` to match the objects they construct.

**Structure**

A small app can use `Factory` or `AppFactory`; these names are interchangeable conventions. For example:

```
Factory/
├── Interface/AppFactoryProtocol.swift
└── Implementation/AppFactory.swift
```

For a larger app, group focused factories under `Factory/[Domain]/Interface/` and `Implementation/`, such as `AccountFactory` or `CatalogFactory`. `App` supplies any shared dependencies across these factories.

**Example responsibilities**

```swift
protocol AppFactoryProtocol {
    func createSessionManager() -> any SessionManagerProtocol
    func createSignInUseCase(session: any SessionManagerProtocol) -> any SignInUseCaseProtocol
    @MainActor
    func makeSignInViewModel(useCase: any SignInUseCaseProtocol) -> SignInViewModel
}
```

`App` calls these methods in dependency order, supplies existing instances, and injects the resulting ViewModel into the View. See [Dependency flow](dependency-flow.md).

## Manager

**Purpose:** Low-level operations — filesystem, network, persistence, device APIs. Expose protocols for testability.

### Manager/Common

Reusable across apps: Logging, Navigation, Networking, Storage, Analytics.

### Manager/Feature

App-specific integrations: e.g. camera, document picker, on-device ML.

**Per-domain structure**

```
Manager/.../[Domain]/
├── Interface/[Domain]Protocol.swift
├── Implementation/[Domain]Manager.swift
├── Providers/                # optional platform variants
├── Helper/                   # optional
└── Model/
    ├── State/
    ├── Configuration/
    ├── Data/
    └── Error/
```

**Rules:** No business orchestration across features; no SwiftUI.

## UseCase

**Purpose:** Own one focused business responsibility; orchestrate manager protocols directly. A feature may require several UseCases.

**Rules**

- `[Feature]UseCaseProtocol` + `[Feature]UseCase`
- `async/await` for operations
- Inject the narrow manager **protocols** needed in `init`
- Never depend on or call another UseCase. Keep a business workflow in the UseCase responsible for that operation, using manager protocols directly
- Respect cancellation in long work
- May expose `AsyncStream` or publishers for progress

**Structure**

A small app can use `Factory` or `AppFactory`; these names are interchangeable conventions. For example:

```
UseCase/[Domain]/[Feature]/
├── Interface/[Feature]UseCaseProtocol.swift
└── Implementation/[Feature]UseCase.swift
```

## ViewModel

**Purpose:** UI state and user-intent handling.

**Rules**

- `@Observable` + `@MainActor`
- Nested `enum ViewState` with associated values
- Depends on one or more focused UseCase protocols; no required primary UseCase
- Never depends on managers or other ViewModels
- Do not merge unrelated business responsibilities into a single UseCase to reduce initializer parameters
- Filtering, selection, pagination — UI-facing only
- Organize with extensions (e.g. `// MARK: - Loading`)

## View

**Purpose:** SwiftUI presentation.

**Rules**

- Depends on ViewModel only (`@State` for `@Observable` VMs)
- `Task { await viewModel.action() }` for async
- Extract subviews as private computed properties or `Component/`

## Component

Reusable UI shared by 2+ features. Group by type: `Button/`, `Loading/`, etc.

## Constants

App-wide static values (`UserDefaults` keys, endpoints). Nested enums — no logic.

## Utility

Cross-cutting extensions (`Extension+URL.swift`). Foundation-level helpers only — not feature business rules.

## Pages (feature module)

Feature UI module:

```
Pages/[Feature]/
├── Model/
├── View/
└── ViewModel/
```

Feature-specific models live here unless shared at manager level.
