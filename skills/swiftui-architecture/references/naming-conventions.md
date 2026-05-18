# Naming conventions

## Types

| Kind | Pattern | Example |
|------|---------|---------|
| Factory | `[Domain]Factory` | `AuthFactory` |
| Factory protocol | `[Domain]FactoryProtocol` | `AuthFactoryProtocol` |
| Manager | `[Domain]Manager` | `FileSystemManager` |
| Manager protocol | `[Domain]Protocol` or `[Domain]Operations` | `FileSystemOperations` |
| Use case protocol | `[Feature]UseCaseProtocol` | `ItemListUseCaseProtocol` |
| Use case | `[Feature]UseCase` | `ItemListUseCase` |
| ViewModel | `[Feature]ViewModel` | `ItemListViewModel` |
| View state | `ViewState` (nested in VM) | `ItemListViewModel.ViewState` |
| View | `[Feature]View` | `ItemListView` |
| Navigation route | `[Feature]Route` | `ItemListRoute` |
| Provider | `[Platform][Domain]Provider` | `iOSFileAccessProvider` |

## Files

| Kind | Pattern |
|------|---------|
| Use case protocol | `[Feature]UseCaseProtocol.swift` |
| Use case impl | `[Feature]UseCase.swift` |
| Utility extension | `Extension+[Type].swift` |

## Models

- **State/** — runtime enums (`LoadingState`)
- **Configuration/** — settings (`RetryPolicy`)
- **Data/** — DTOs / entities (`Item`)
- **Error/** — `LocalizedError` types (`ItemListError`)

## Constants

```swift
enum Constant {
    enum UserDefault {
        static let lastSelectedFilter = "itemList.lastSelectedFilter"
    }
}
```

Embed full key strings; avoid string concatenation for keys.
