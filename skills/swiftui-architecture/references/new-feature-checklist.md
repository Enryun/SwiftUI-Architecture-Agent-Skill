# New feature checklist

Use when adding `[Feature]` to an existing SwiftUI app.

## 1. Plan (before files)

- [ ] Feature name and domain (`Catalog` / `ItemDetail`)
- [ ] View roles: feature, composition, or reusable visual component
- [ ] Which managers are needed (new vs existing)
- [ ] Focused UseCases needed (reuse vs new) and each public API (methods, async, streams)
- [ ] `ViewState` cases
- [ ] Navigation routes (if any)

## 2. Factory

Reuse an existing factory; add a creation method only when a new dependency must be constructed. Split factories only when current domain/platform boundaries justify it.

- [ ] Appropriately scoped factory protocol + implementation (`Factory`, `AppFactory`, or `[Domain]Factory`)
- [ ] Each method creates one Manager, UseCase, or ViewModel
- [ ] Required dependencies supplied as arguments by `App`
- [ ] Implementation selection (availability/config) stays in the factory
- [ ] No business rules or hidden feature/application graph construction

## 3. Manager (if needed)

- [ ] `Manager/Common/...` vs `Manager/Feature/...`
- [ ] `[Domain]Protocol` in `Interface/`
- [ ] `[Domain]Manager` in `Implementation/`
- [ ] Models under `Model/{State,Configuration,Data,Error}/`

## 4. UseCase

- [ ] `UseCase/[Domain]/[Feature]/Interface/[Feature]UseCaseProtocol.swift`
- [ ] `UseCase/[Domain]/[Feature]/Implementation/[Feature]UseCase.swift`
- [ ] Each UseCase owns one focused business responsibility
- [ ] Inject manager protocols in `init`; no other UseCase dependencies
- [ ] `async` APIs; cancellation in long operations

## 5. ViewModel

- [ ] `Pages/[Feature]/ViewModel/[Feature]ViewModel.swift`
- [ ] `@Observable` + `@MainActor`
- [ ] Inject one or more focused UseCase protocols; no managers or other ViewModels
- [ ] `enum ViewState { ... }`
- [ ] `private(set) var state`
- [ ] Extensions for logical groups (loading, filtering)

## 6. View

- [ ] `Pages/[Feature]/View/[Feature]View.swift`
- [ ] Exactly one owning feature ViewModel received through injection
- [ ] `switch viewModel.state` for UI modes
- [ ] No UseCase calls, business operations, or service creation
- [ ] Independent features assembled by a composition view receiving existing child ViewModels
- [ ] No aggregate ViewModel that merely holds child ViewModels or copies their state
- [ ] Visual components receive values, bindings, and action closures
- [ ] Callbacks express focused UI events or content composition, with no hidden business dependencies or redundant forwarding chains

## 7. Models

- [ ] Feature models in `Pages/[Feature]/Model/`
- [ ] Shared/persistence models in manager `Model/` if reused

## 8. Composition

- [ ] Wire individual objects in `App` through factories
- [ ] Create shared managers once in `App` and reuse those instances
- [ ] Navigation: typed routes and one path per independent stack/window; local state or `NavigationManager<FeatureRoute>` as needed
- [ ] Route payloads contain identifiers/values, not ViewModels or services
- [ ] Navigation accessed only by presentation Views, never ViewModels/UseCases
- [ ] Destination ViewModel identity and lifetime are explicit

## 9. Optional

- [ ] Shared UI → `Component/`
- [ ] App constants → `Constants/`
- [ ] Tests with mock protocols

## 10. Review

- [ ] No upward dependencies
- [ ] No ViewModel → ViewModel or UseCase → UseCase dependencies
- [ ] No concrete manager types in ViewModel `init`
- [ ] No business logic in View
- [ ] Each feature View receives one owning ViewModel; multiple child ViewModels appear only in composition views
