# Layer overview

## Factory

**Purpose:** Create and configure managers (availability, configuration, platform variants). Optional composition root for UseCase → ViewModel graphs.

**Rules**

- Protocol-first: `[Domain]FactoryProtocol`
- No business logic — creation and wiring only
- May detect API/OS availability before returning a manager
- Keep `App` small by delegating construction here

**Structure**

```
Factory/[Domain]/
├── Interface/[Domain]FactoryProtocol.swift
├── Implementation/[Domain]Factory.swift
└── Model/                    # optional (e.g. API enum)
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
