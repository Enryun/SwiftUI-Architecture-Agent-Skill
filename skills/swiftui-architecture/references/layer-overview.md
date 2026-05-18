# Layer overview

## Factory

**Purpose:** **Dependency construction and composition** at the edge of the app — not a runtime layer on every call.

Factories handle two related jobs:

1. **Manager construction** — build and return manager **protocol** types; choose implementations using availability, configuration, or platform when needed.
2. **Feature composition** — optionally assemble UseCase → ViewModel (or a full feature graph) so `App` only holds factories and root views.

**Rules**

- Protocol-first: `[Domain]FactoryProtocol`
- No business logic — creation and wiring only (no pricing rules, validation policy, etc.)
- May detect API/OS availability before returning a manager
- Keep `App` small by delegating construction here
- Factories **do not replace** Manager/UseCase/ViewModel — they **instantiate** them

**When to add a Factory**

- Multiple implementations of a manager (API variants, OS version, user setting)
- Non-trivial wiring for a feature (several managers → use case → view model)
- You want `App` to call `factory.makeItemListViewModel()` instead of inline `init` chains

**When to skip Factory**

- One implementation per protocol; wire once in `App`
- Small feature with no implementation selection

**Structure**

```
Factory/[Domain]/
├── Interface/[Domain]FactoryProtocol.swift
├── Implementation/[Domain]Factory.swift
└── Model/                    # optional (e.g. API enum, config)
```

**Example responsibilities**

```swift
protocol AuthFactoryProtocol {
    func createSessionManager() async -> SessionManagerProtocol
    func makeSignInViewModel() -> SignInViewModel
}
```

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

**Purpose:** Business logic for one feature; orchestrate manager protocols.

**Rules**

- `[Feature]UseCaseProtocol` + `[Feature]UseCase`
- `async/await` for operations
- Inject manager **protocols** in `init`
- Respect cancellation in long work
- May expose `AsyncStream` or publishers for progress

**Structure**

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
- Depends on **one primary** use case protocol
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
