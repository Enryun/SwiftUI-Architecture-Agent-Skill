# Dependency flow

## Diagram

**Construction** (startup / feature entry):

```
App — composition root; owns sharing and lifetimes
  ├─ AppFactory.createManager(...) → manager
  ├─ AppFactory.createUseCase(manager: manager) → useCase
  └─ AppFactory.makeViewModel(useCase: useCase) → viewModel
```

Each factory call creates one object. `App` supplies the dependencies and injects the resulting ViewModel into the View.

**Runtime** (per action):

```
View → ViewModel → UseCase → Manager
```

Factory is part of **how objects are created**, not a hop in the runtime call chain.

## Critical rules

1. **Downward only** — higher layers never import lower UI layers.
2. **No cycles** — if UseCase needs something from ViewModel, redesign.
3. **Protocols at boundaries** — `FileSystemOperations`, not `FileSystemManager`.
4. **Initializer injection** — no service locator inside ViewModels.
5. **Focused dependencies** — a ViewModel may use multiple UseCase protocols. ViewModel → ViewModel and UseCase → UseCase dependencies are forbidden by this architecture.

## Multiple focused UseCases

```text
ProfileView → ProfileViewModel
               ├─ ProfileUseCaseProtocol → profile storage manager protocol
               └─ PurchaseUseCaseProtocol → purchase manager protocol
```

The ViewModel owns presentation state and forwards user intents to the relevant UseCase. It does not borrow another ViewModel or chain business rules together itself. When an operation requires coordinated business steps, its focused UseCase calls the required manager protocols directly.

## App composition root

Use one factory initially, or focused domain/platform factories when complexity warrants them. `AppFactory` below is an example name, not a required singleton factory. Factory methods accept required dependencies as arguments; they do not assemble a feature graph internally.

Illustrative wiring inside `App` initialization (`context` is supplied by the app's persistence setup):

```swift
let factory: any AppFactoryProtocol = AppFactory()

// 1. Create each shared manager once in App.
let storage = factory.createStorage(context: context)
let network = factory.createDownloadManager()

// 2. App passes the existing managers to individual UseCase builders.
let useCase = factory.createItemListUseCase(storage: storage, download: network)

// 3. App passes the UseCase to the ViewModel builder.
let viewModel = factory.makeItemListViewModel(useCase: useCase)

// 4. App retains the graph and supplies the ViewModel to the View.
```

Pass the same manager instances to other consumers when their state must be shared. Do not add service lookup or `makeEntireFeature()` / `makeApp()` methods. The View receives its ViewModel, not the factory.

See [the example](examples.md#5-appfactory-individual-creation) for factory methods and `App` wiring.

## What each layer may import

| Layer | May depend on |
|-------|----------------|
| App | Factory protocols, dependency protocols, ViewModel, View (composition and sharing) |
| Factory | Manager protocols/implementations, UseCase, ViewModel (individual creation only) |
| Manager | Other manager protocols, Foundation, system frameworks |
| UseCase | Manager protocols, domain models (never other UseCases) |
| ViewModel | One or more focused UseCase protocols, feature models (never other ViewModels) |
| View | ViewModel, Component, SwiftUI |

## Testing

| Layer | Mock |
|-------|------|
| UseCase tests | Fake manager protocols |
| ViewModel tests | Fakes for the injected UseCase protocols |
| View previews | ViewModel with preview/mock use case |
