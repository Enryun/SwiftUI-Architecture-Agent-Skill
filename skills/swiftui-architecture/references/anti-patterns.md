# Anti-patterns

## ViewModel → Manager directly

```swift
// BAD
@MainActor
final class ItemListViewModel {
    private let fileSystem = FileSystemManager()
}

// GOOD
@MainActor
final class ItemListViewModel {
    private let useCase: ItemListUseCaseProtocol
    init(useCase: ItemListUseCaseProtocol) { self.useCase = useCase }
}
```

## Business logic in View

```swift
// BAD
Button("Refresh") {
    Task {
        let items = try await useCase.fetchItems()
        self.items = items
    }
}

// GOOD
Button("Refresh") {
    Task { await viewModel.refresh() }
}
```

## Concrete dependency types

```swift
// BAD
init(storage: SwiftDataStorageManager<Item>) { }

// GOOD
init(storage: any StorageProtocol) { }
// or init(storage: StorageProtocol) { } when existential is appropriate
```

## Boolean flag soup in ViewModel

```swift
// BAD
var isLoading = false
var isEmpty = false
var error: Error?

// GOOD
enum ViewState {
    case idle
    case loading
    case loaded([Item])
    case empty
    case failed(Error)
}
private(set) var state: ViewState = .idle
```

## Business logic in Factory

```swift
// BAD — factory computes business rules
func createUseCase() -> ItemListUseCase {
    let discount = user.isPremium ? 0.2 : 0.0  // belongs in UseCase
    ...
}

// GOOD — App supplies dependencies; factory creates one object
func createUseCase(
    storage: any StorageProtocol<Item>,
    download: any URLDownloadManagerProtocol
) -> any ItemListUseCaseProtocol {
    ItemListUseCase(storage: storage, download: download)
}
```

## Hidden graph construction in Factory

Do not add methods that construct managers, UseCases, and ViewModels together. Use individual creation methods, whether there is one factory or several focused factories. `App` calls them in order and reuses shared manager instances.

## ViewModel → ViewModel

Do not inject `PurchaseViewModel` into `ProfileViewModel` to reuse purchase behavior. Inject `PurchaseUseCaseProtocol` alongside `ProfileUseCaseProtocol`; each ViewModel owns its own presentation state.

## UseCase → UseCase

Do not inject `PurchaseUseCaseProtocol` into `ProfileUseCase` merely to forward purchase actions. Let the ViewModel depend on both focused protocols. If a single business operation requires several infrastructure steps, its owning UseCase calls the required manager protocols directly; do not move that business workflow into the ViewModel.

## God UseCase

One UseCase doing unrelated business responsibilities → split into focused UseCases under `UseCase/[Domain]/`. Inject the relevant protocols into the ViewModel rather than creating a wrapper UseCase to hide them.

## Leaking SwiftUI into UseCase

UseCase must not `import SwiftUI`. Pass `URL`, `Date`, domain types — map to UI types in ViewModel.

## Circular dependencies

If Manager needs UseCase or ViewModel, extract shared protocol to a lower layer or use events/callbacks at infrastructure level only.
