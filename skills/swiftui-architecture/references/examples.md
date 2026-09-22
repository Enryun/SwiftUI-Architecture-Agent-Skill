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

## 5. AppFactory (individual creation)

`AppFactory` is the example name for this small graph; larger applications may split creation methods into focused domain/platform factories. Each method creates one object using dependencies supplied by `App`. The storage and networking implementations below are illustrative; their definitions are omitted.

```swift
protocol AppFactoryProtocol {
    func createStorage(context: ModelContext) -> any StorageProtocol<Item>
    func createDownloadManager() -> any URLDownloadManagerProtocol
    func createItemListUseCase(
        storage: any StorageProtocol<Item>,
        download: any URLDownloadManagerProtocol
    ) -> any ItemListUseCaseProtocol
    @MainActor
    func makeItemListViewModel(useCase: any ItemListUseCaseProtocol) -> ItemListViewModel
}

final class AppFactory: AppFactoryProtocol {
    func createStorage(context: ModelContext) -> any StorageProtocol<Item> {
        SwiftDataStorageManager<Item>(context: context)
    }

    func createDownloadManager() -> any URLDownloadManagerProtocol {
        URLDownloadManager()
    }

    func createItemListUseCase(
        storage: any StorageProtocol<Item>,
        download: any URLDownloadManagerProtocol
    ) -> any ItemListUseCaseProtocol {
        ItemListUseCase(storage: storage, download: download)
    }

    @MainActor
    func makeItemListViewModel(useCase: any ItemListUseCaseProtocol) -> ItemListViewModel {
        ItemListViewModel(useCase: useCase)
    }
}
```

## 6. App (composition and sharing)

`App` owns the graph. Container setup remains a placeholder in this abbreviated example.

```swift
@main
@MainActor
struct MyApp: App {
    private let modelContainer: ModelContainer
    private let itemListViewModel: ItemListViewModel

    init() {
        let factory: any AppFactoryProtocol = AppFactory()
        // Configure the container and startup error handling for your app.
        let container = ... // ModelContainer configured for Item
        modelContainer = container

        // App creates shared dependencies once and passes them to consumers.
        let storage = factory.createStorage(context: container.mainContext)
        let download = factory.createDownloadManager()
        let useCase = factory.createItemListUseCase(storage: storage, download: download)
        itemListViewModel = factory.makeItemListViewModel(useCase: useCase)
    }

    var body: some Scene {
        WindowGroup {
            ItemListView(viewModel: itemListViewModel)
        }
    }
}
```
