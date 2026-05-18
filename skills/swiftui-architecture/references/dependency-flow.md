# Dependency flow

## Diagram

**Construction** (startup / feature entry):

```
App
  ↓
Factory (optional) — builds managers; may assemble UseCase + ViewModel
  ↓
wired instances: Manager, UseCase, ViewModel
```

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

## App composition root

```swift
@main
struct MyApp: App {
    private let itemListFactory: ItemListFactoryProtocol

    init() {
        itemListFactory = ItemListFactory()
    }

    var body: some Scene {
        WindowGroup {
            RootView(factory: itemListFactory)
        }
    }
}
```

Factory may expose **manager builders** and **feature assemblers**:

```swift
protocol CatalogFactoryProtocol {
    func createStorage() -> any StorageProtocol<Item>
    func makeItemListViewModel() -> ItemListViewModel
}
```

`makeItemListViewModel()` typically constructs the use case internally, then the view model — so `App` stays thin.

## Wiring a feature

```swift
// 1. Managers (via factory or direct in small apps)
let storage: StorageProtocol = SwiftDataStorageManager<Item>(context: context)
let network: URLDownloadManagerProtocol = URLDownloadManager()

// 2. Use case
let useCase: ItemListUseCaseProtocol = ItemListUseCase(
    storage: storage,
    download: network
)

// 3. View model
let viewModel = ItemListViewModel(useCase: useCase)

// 4. View
ItemListView(viewModel: viewModel)
```

## What each layer may import

| Layer | May depend on |
|-------|----------------|
| Factory | Manager protocols/implementations, UseCase, ViewModel (composition only) |
| Manager | Other manager protocols, Foundation, system frameworks |
| UseCase | Manager protocols, domain models |
| ViewModel | UseCase protocols, feature models |
| View | ViewModel, Component, SwiftUI |

## Testing

| Layer | Mock |
|-------|------|
| UseCase tests | Fake manager protocols |
| ViewModel tests | Fake use case protocol |
| View previews | ViewModel with preview/mock use case |
