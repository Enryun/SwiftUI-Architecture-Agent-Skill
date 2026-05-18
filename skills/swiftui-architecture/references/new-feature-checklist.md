# New feature checklist

Use when adding `[Feature]` to an existing SwiftUI app.

## 1. Plan (before files)

- [ ] Feature name and domain (`Catalog` / `ItemDetail`)
- [ ] Which managers are needed (new vs existing)
- [ ] Use case public API (methods, async, streams)
- [ ] `ViewState` cases
- [ ] Navigation routes (if any)

## 2. Factory (if needed)

Add when you need implementation selection **or** feature assembly for `App`. Skip if wiring once in `App` is enough.

- [ ] `[Domain]FactoryProtocol`
- [ ] `[Domain]Factory` implementation
- [ ] Optional `Model/` for config/API enum
- [ ] `create…()` methods returning **manager protocols** (availability/config selection **only** here)
- [ ] Optional `make[Feature]ViewModel()` composing UseCase + ViewModel
- [ ] No business rules in factory methods

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

- [ ] Wire in `App` or factory `make[Feature]ViewModel()`
- [ ] Navigation: `NavigationManager<FeatureRoute>` if using stack routes

## 9. Optional

- [ ] Shared UI → `Component/`
- [ ] App constants → `Constants/`
- [ ] Tests with mock protocols

## 10. Review

- [ ] No upward dependencies
- [ ] No concrete manager types in ViewModel `init`
- [ ] No business logic in View
