# End-to-end example: Item list

Neutral feature: load items from storage, optional remote refresh.

## 1. Manager protocol

```swift
// Manager/Common/Storage/Interface/StorageProtocol.swift
protocol StorageProtocol<Item>: Sendable {
    associatedtype Item: PersistentModel
    func fetchAll() async throws -> [Item]
    func save(_ item: Item) async throws
}
```

## 2. Use case

```swift
// UseCase/Catalog/ItemList/Interface/ItemListUseCaseProtocol.swift
protocol ItemListUseCaseProtocol: Sendable {
    func loadItems() async throws -> [Item]
    func refreshFromNetwork() async throws -> [Item]
}

// UseCase/Catalog/ItemList/Implementation/ItemListUseCase.swift
final class ItemListUseCase: ItemListUseCaseProtocol {
    private let storage: any StorageProtocol<Item>
    private let download: URLDownloadManagerProtocol

    init(storage: any StorageProtocol<Item>, download: URLDownloadManagerProtocol) {
        self.storage = storage
        self.download = download
    }

    func loadItems() async throws -> [Item] {
        try await storage.fetchAll()
    }

    func refreshFromNetwork() async throws -> [Item] {
        try Task.checkCancellation()
        // orchestrate download + map + save
        try await storage.fetchAll()
    }
}
```

## 3. ViewModel

```swift
// Pages/ItemList/ViewModel/ItemListViewModel.swift
@Observable
@MainActor
final class ItemListViewModel {
    enum ViewState: Equatable {
        case idle
        case loading
        case loaded([Item])
        case failed(String)
    }

    private let useCase: ItemListUseCaseProtocol
    private(set) var state: ViewState = .idle

    init(useCase: ItemListUseCaseProtocol) {
        self.useCase = useCase
    }

    func load() async {
        state = .loading
        do {
            let items = try await useCase.loadItems()
            state = .loaded(items)
        } catch {
            state = .failed(error.localizedDescription)
        }
    }
}
```

## 4. View

```swift
// Pages/ItemList/View/ItemListView.swift
struct ItemListView: View {
    @State private var viewModel: ItemListViewModel

    init(viewModel: ItemListViewModel) {
        _viewModel = State(initialValue: viewModel)
    }

    var body: some View {
        Group {
            switch viewModel.state {
            case .idle, .loading:
                ProgressView()
            case .loaded(let items):
                List(items) { item in
                    Text(item.title)
                }
            case .failed(let message):
                ContentUnavailableView("Error", systemImage: "exclamationmark.triangle", description: Text(message))
            }
        }
        .task { await viewModel.load() }
    }
}
```

## 5. Factory (optional)

```swift
protocol CatalogFactoryProtocol {
    func makeItemListViewModel() -> ItemListViewModel
}

final class CatalogFactory: CatalogFactoryProtocol {
    private let storage: any StorageProtocol<Item>
    private let download: URLDownloadManagerProtocol

    init(storage: any StorageProtocol<Item>, download: URLDownloadManagerProtocol) {
        self.storage = storage
        self.download = download
    }

    func makeItemListViewModel() -> ItemListViewModel {
        let useCase = ItemListUseCase(storage: storage, download: download)
        return ItemListViewModel(useCase: useCase)
    }
}
```

## 6. App

```swift
@main
struct MyApp: App {
    private let catalogFactory: CatalogFactoryProtocol

    init() {
        // wire storage + download, then factory
        catalogFactory = CatalogFactory(storage: ..., download: ...)
    }

    var body: some Scene {
        WindowGroup {
            ItemListView(viewModel: catalogFactory.makeItemListViewModel())
        }
    }
}
```
