# Naming conventions

## Types

| Kind | Pattern | Example |
|------|---------|---------|
| Factory | `Factory`, `AppFactory`, or `[Domain]Factory` | `CatalogFactory` |
| Factory protocol | Matching factory name + `Protocol` | `CatalogFactoryProtocol` |
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

Choose the owner before choosing subfolders; see [model ownership](models.md). These optional groupings apply within the owning domain, integration, or UI feature. Do not create each grouping or duplicate each model across layers.

- **State/** — runtime enums (`LoadingState`)
- **Configuration/** — settings (`RetryPolicy`)
- **Data/** — domain values in domain code (`Item`), DTOs in integrations (`ItemResponse`)
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
