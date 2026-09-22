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

Reusable across apps: Logging, Networking, Storage, Analytics. Navigation may share this folder as a presentation-state exception described below.

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

**Rules:** No business orchestration across features; no SwiftUI in infrastructure managers. `NavigationManager` is a presentation-state exception despite its folder/name: it may use SwiftUI bindings and be accessed by Views, never ViewModels or UseCases. See [navigation and lifetimes](navigation.md).

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

**One owning ViewModel per feature View; one or more focused UseCase protocols per ViewModel.** A feature spanning multiple business domains still has one owning ViewModel. Inject the focused UseCases it needs; do not force a single umbrella UseCase or introduce sibling ViewModels for that same feature.

**Rules**

- `@Observable` + `@MainActor`
- Nested `enum ViewState` with associated values
- Depends on one or more focused UseCase protocols; no required primary UseCase
- Never depends on managers or other ViewModels
- Do not merge unrelated business responsibilities into a single UseCase to reduce initializer parameters
- Filtering, selection, pagination — UI-facing only
- Organize with extensions (e.g. `// MARK: - Loading`)

## View

Choose the role by responsibility, not the type's name or how many screens it displays.

### Feature View

- Receives exactly one owning feature ViewModel, including destination screens and embedded feature sections.
- May also receive value models, bindings, and action closures.
- Presents that feature's state and forwards user actions to its ViewModel.
- Does not receive another feature's ViewModel, call UseCases, create business services, or perform business operations.
- Extract visual subviews using values, bindings, and closures; a visual subview does not need its own ViewModel merely because it is a separate type.

Feature Views may also access their scoped navigation environment for presentation only; this is not another feature ViewModel or a business-service dependency.

### Composition View

- Assembles independent feature views using child ViewModels supplied by `App`; may receive multiple child ViewModels.
- Owns layout, navigation destinations, and cross-feature presentation such as which feature sheet is displayed. May own local stack navigation state; shared business services remain composed by `App`.
- Keeps each feature's loading, actions, and business state in that feature's existing ViewModel.
- Does not construct business services, call UseCases, or perform business operations.
- Do not create an aggregate ViewModel solely to hold child ViewModels or copy their state.
- Use content closures for intermediate layout-only views; avoid forwarding containers without a concrete responsibility.

When one feature needs business operations from multiple domains, keep one feature ViewModel and inject the focused UseCases it needs. Multiple domains alone do not make a screen a composition view.

See [composition example](dependency-flow.md#composing-independent-features).

## Component

Reusable visual UI receives plain values, bindings, and action closures rather than feature ViewModels, UseCases, or Managers. It may own local visual state; feature loading and business state remain in the owning ViewModel.

Keep feature-only visual helpers beside the feature View. Move UI shared by 2+ features into `Component/`, grouped by type (`Button/`, `Loading/`, etc.).

### Closure boundaries

Use closures where they express a clear UI responsibility:

- **Component action:** `onSave` or `onSelect(id)` forwards an event to the owning feature View, which calls its ViewModel.
- **Presentation event:** `onClose` or `onShowDetails(id)` lets the parent coordinate presentation.
- **Content composition:** a `@ViewBuilder` closure supplies content to a layout container without passing feature ViewModels through it.

Avoid overuse:

- Keep callbacks small in number and coherent in purpose. There is no fixed numeric limit; if a View's initializer accumulates unrelated callbacks, reassess its responsibility and feature boundary.
- Do not replace a feature's owning ViewModel with a bundle of business-operation closures or add callbacks that duplicate its existing action methods.
- Do not pass callbacks through several containers that only forward them. Prefer direct composition or content closures where intermediate views only arrange layout; do not introduce a global event bus or service locator to avoid forwarding.
- A closure does not erase a dependency: wrapping a Manager, UseCase, or another ViewModel in a callback does not make an otherwise forbidden dependency acceptable.
- Keep business decisions and multi-step business workflows in UseCases. UI callbacks forward intent; they do not implement those workflows.
- Use descriptive event names and the smallest useful payload, such as an identifier or value. Do not pass service objects through callback arguments.

For example, inside `ProfileView`, `SaveButton(onSave: { viewModel.save() })` forwards a component event to the owning ViewModel. Supplying `ProfileView` with a separate `onSave` closure that calls a persistence Manager bypasses that ownership boundary.

Review capture ownership for stored callbacks; use weak captures when needed to break an actual retain cycle, not automatically for every closure.

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

UI-only models and drafts live here. Plain domain values used by business layers belong in a domain location independent of Pages and persistence implementations. Reuse values rather than adding a representation per layer; see [model ownership](models.md).
