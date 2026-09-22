# New feature checklist

Use when adding `[Feature]` to an existing SwiftUI app.

## 1. Plan (before files)

- [ ] Feature name and domain (`Catalog` / `ItemDetail`)
- [ ] Which managers are needed (new vs existing)
- [ ] Use case public API (methods, async, streams)
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
- [ ] Inject manager protocols in `init`
- [ ] `async` APIs; cancellation in long operations

## 5. ViewModel

- [ ] `Pages/[Feature]/ViewModel/[Feature]ViewModel.swift`
- [ ] `@Observable` + `@MainActor`
- [ ] `enum ViewState { ... }`
- [ ] `private(set) var state`
- [ ] Extensions for logical groups (loading, filtering)

## 6. View

- [ ] `Pages/[Feature]/View/[Feature]View.swift`
- [ ] `@State private var viewModel` (or injected init)
- [ ] `switch viewModel.state` for UI modes
- [ ] No use case/manager references

## 7. Models

- [ ] Feature models in `Pages/[Feature]/Model/`
- [ ] Shared/persistence models in manager `Model/` if reused

## 8. Composition

- [ ] Wire individual objects in `App` through factories
- [ ] Create shared managers once in `App` and reuse those instances
- [ ] Navigation: `NavigationManager<FeatureRoute>` if using stack routes

## 9. Optional

- [ ] Shared UI → `Component/`
- [ ] App constants → `Constants/`
- [ ] Tests with mock protocols

## 10. Review

- [ ] No upward dependencies
- [ ] No concrete manager types in ViewModel `init`
- [ ] No business logic in View
