# End-to-end example: Item catalog

The complete source is embedded below. It is a standalone macOS 14+ SwiftUI app with no packages, credentials, network access, or omitted implementations. Use a macOS App target with Swift 6 and nonisolated default actor isolation; replace its generated app entrypoint with this source rather than adding a second `@main`.

The sample loads a small catalog and lets the user mark items as favorites. Storage is deliberately in memory: changes last for the running app session only. This demonstrates architecture without adding persistence setup or implying that SwiftData models are domain values.

## Dependency graph

```text
ItemCatalogExampleApp — composition and lifetime ownership
  ├─ CatalogFactory.makeStorage(items:) → one shared storage actor
  ├─ CatalogFactory.makeLoadCatalog(storage:) → LoadCatalogUseCase
  ├─ CatalogFactory.makeSetFavorite(storage:) → SetFavoriteUseCase
  └─ CatalogFactory.makeViewModel(loadCatalog:setFavorite:) → CatalogViewModel

CatalogView → CatalogViewModel
                ├─ LoadCatalogUseCaseProtocol ─┐
                └─ SetFavoriteUseCaseProtocol ┴─ CatalogStorageProtocol

CatalogRow ← value + action closure from CatalogView
```

## What to review

| Source section | Architectural point |
|---|---|
| Domain value | `CatalogItem` is an identifiable, equatable, Sendable value; no persistence framework dependency |
| Manager | `InMemoryCatalogStorage` is an actor protecting mutable storage and exposing a narrow protocol |
| Focused UseCases | Loading and setting a favorite have separate protocols; both depend directly on the same storage instance, never on each other |
| ViewModel | One main-actor observable ViewModel receives both UseCase protocols and owns loading, error, and mutation state |
| Visual component | `CatalogRow` receives a value and one action closure, without ViewModel or service access |
| Feature View | `CatalogView` receives exactly one ViewModel and forwards user actions to it |
| Factory | Each method constructs one object from supplied dependencies; ViewModel creation is explicitly main-actor isolated |
| App | Composes the graph, deliberately shares storage, and presents one macOS window |

The UseCases are intentionally small so the whole dependency graph can be inspected. They are operation boundaries for this example, not a reason to add forwarding types to every trivial screen.

## State and lifetime decisions

- The single macOS `Window` deliberately uses one app-owned feature ViewModel. For multiple windows, choose scene-specific ViewModel ownership rather than copying this lifetime unchanged.
- Navigation stays in presentation: `App` supplies the `NavigationStack`. There are no routes or navigation commands in the ViewModel. See [navigation examples](navigation.md) for destination composition and result-based dismissal.
- Load requests and favorite changes are serialized by the ViewModel's state guards. A favorite change refreshes displayed data through the loading UseCase; it does not call that UseCase from another UseCase.
- Cancellation is not displayed as an error. Cancellation does not roll back a completed storage mutation; a later load reads authoritative state again.
- Favorite errors leave the current list visible. Empty data has an explicit presentation.

## Complete source

This is the authoritative example. Copy the entire Swift block into a macOS App target, replacing its generated app entrypoint. Sections are grouped for review; split types into the existing project's layer folders when adapting the example.

```swift
import Foundation
import Observation
import SwiftUI

// MARK: - Domain value

struct CatalogItem: Identifiable, Equatable, Sendable {
    let id: UUID
    let title: String
    var isFavorite: Bool
}

enum CatalogError: LocalizedError {
    case itemUnavailable

    var errorDescription: String? {
        "This item is no longer available."
    }
}

// MARK: - Manager

protocol CatalogStorageProtocol: Sendable {
    func fetchItems() async throws -> [CatalogItem]
    func setFavorite(_ isFavorite: Bool, for id: UUID) async throws
}

actor InMemoryCatalogStorage: CatalogStorageProtocol {
    private var items: [CatalogItem]

    init(items: [CatalogItem]) {
        self.items = items
    }

    func fetchItems() async throws -> [CatalogItem] {
        try Task.checkCancellation()
        return items
    }

    func setFavorite(_ isFavorite: Bool, for id: UUID) async throws {
        try Task.checkCancellation()
        guard let index = items.firstIndex(where: { $0.id == id }) else {
            throw CatalogError.itemUnavailable
        }
        items[index].isFavorite = isFavorite
    }
}

// MARK: - Focused UseCases

protocol LoadCatalogUseCaseProtocol: Sendable {
    func execute() async throws -> [CatalogItem]
}

struct LoadCatalogUseCase: LoadCatalogUseCaseProtocol {
    private let storage: any CatalogStorageProtocol

    init(storage: any CatalogStorageProtocol) {
        self.storage = storage
    }

    func execute() async throws -> [CatalogItem] {
        try await storage.fetchItems()
    }
}

protocol SetFavoriteUseCaseProtocol: Sendable {
    func execute(itemID: UUID, isFavorite: Bool) async throws
}

struct SetFavoriteUseCase: SetFavoriteUseCaseProtocol {
    private let storage: any CatalogStorageProtocol

    init(storage: any CatalogStorageProtocol) {
        self.storage = storage
    }

    func execute(itemID: UUID, isFavorite: Bool) async throws {
        try await storage.setFavorite(isFavorite, for: itemID)
    }
}

// MARK: - ViewModel

@MainActor
@Observable
final class CatalogViewModel {
    enum ViewState: Equatable {
        case idle
        case loading
        case loaded([CatalogItem])
        case failed(String)
    }

    private let loadCatalog: any LoadCatalogUseCaseProtocol
    private let setFavorite: any SetFavoriteUseCaseProtocol

    private(set) var state: ViewState = .idle
    private(set) var isUpdatingFavorite = false
    private(set) var actionError: String?

    init(
        loadCatalog: any LoadCatalogUseCaseProtocol,
        setFavorite: any SetFavoriteUseCaseProtocol
    ) {
        self.loadCatalog = loadCatalog
        self.setFavorite = setFavorite
    }

    func load() async {
        guard state != .loading, !isUpdatingFavorite else { return }
        let previousState = state
        state = .loading
        do {
            let items = try await loadCatalog.execute()
            try Task.checkCancellation()
            state = .loaded(items)
        } catch {
            if error is CancellationError || Task.isCancelled {
                state = previousState
            } else {
                state = .failed(error.localizedDescription)
            }
        }
    }

    func toggleFavorite(for item: CatalogItem) async {
        guard case .loaded = state, !isUpdatingFavorite else { return }
        isUpdatingFavorite = true
        actionError = nil
        defer { isUpdatingFavorite = false }

        do {
            try await setFavorite.execute(itemID: item.id, isFavorite: !item.isFavorite)
            // Re-read after a completed mutation, even if the UI task was cancelled.
            // Storage remains authoritative; a later load reconciles interrupted reads.
            let items = try await loadCatalog.execute()
            state = .loaded(items)
        } catch {
            if !(error is CancellationError) && !Task.isCancelled {
                actionError = error.localizedDescription
            }
        }
    }
}

// MARK: - Visual component

struct CatalogRow: View {
    let item: CatalogItem
    let onToggleFavorite: () -> Void

    var body: some View {
        HStack {
            Text(item.title)
            Spacer()
            Button(action: onToggleFavorite) {
                Image(systemName: item.isFavorite ? "star.fill" : "star")
            }
            .buttonStyle(.borderless)
            .accessibilityLabel(item.isFavorite ? "Remove favorite" : "Add favorite")
        }
    }
}

// MARK: - Feature View

@MainActor
struct CatalogView: View {
    let viewModel: CatalogViewModel

    var body: some View {
        VStack {
            switch viewModel.state {
            case .idle, .loading:
                ProgressView("Loading catalog")
            case .loaded(let items):
                if items.isEmpty {
                    ContentUnavailableView("No items", systemImage: "tray")
                } else {
                    List(items) { item in
                        CatalogRow(item: item) {
                            Task { await viewModel.toggleFavorite(for: item) }
                        }
                        .disabled(viewModel.isUpdatingFavorite)
                    }
                }
            case .failed(let message):
                Text(message)
                Button("Retry") {
                    Task { await viewModel.load() }
                }
            }
            if let message = viewModel.actionError {
                Text(message).foregroundStyle(.red)
            }
        }
        .navigationTitle("Catalog")
        .task { await viewModel.load() }
    }
}

// MARK: - Factory: one object per call

protocol CatalogFactoryProtocol {
    func makeStorage(items: [CatalogItem]) -> any CatalogStorageProtocol
    func makeLoadCatalog(storage: any CatalogStorageProtocol) -> any LoadCatalogUseCaseProtocol
    func makeSetFavorite(storage: any CatalogStorageProtocol) -> any SetFavoriteUseCaseProtocol
    @MainActor
    func makeViewModel(
        loadCatalog: any LoadCatalogUseCaseProtocol,
        setFavorite: any SetFavoriteUseCaseProtocol
    ) -> CatalogViewModel
}

struct CatalogFactory: CatalogFactoryProtocol {
    func makeStorage(items: [CatalogItem]) -> any CatalogStorageProtocol {
        InMemoryCatalogStorage(items: items)
    }

    func makeLoadCatalog(storage: any CatalogStorageProtocol) -> any LoadCatalogUseCaseProtocol {
        LoadCatalogUseCase(storage: storage)
    }

    func makeSetFavorite(storage: any CatalogStorageProtocol) -> any SetFavoriteUseCaseProtocol {
        SetFavoriteUseCase(storage: storage)
    }

    @MainActor
    func makeViewModel(
        loadCatalog: any LoadCatalogUseCaseProtocol,
        setFavorite: any SetFavoriteUseCaseProtocol
    ) -> CatalogViewModel {
        CatalogViewModel(loadCatalog: loadCatalog, setFavorite: setFavorite)
    }
}

// MARK: - App: composition and shared lifetime

@main
@MainActor
struct ItemCatalogExampleApp: App {
    private let viewModel: CatalogViewModel

    init() {
        let factory: any CatalogFactoryProtocol = CatalogFactory()
        let storage = factory.makeStorage(items: [
            CatalogItem(id: UUID(), title: "Notebook", isFavorite: false),
            CatalogItem(id: UUID(), title: "Pencil", isFavorite: true)
        ])
        let loadCatalog = factory.makeLoadCatalog(storage: storage)
        let setFavorite = factory.makeSetFavorite(storage: storage)
        viewModel = factory.makeViewModel(loadCatalog: loadCatalog, setFavorite: setFavorite)
    }

    var body: some Scene {
        // One macOS window: sharing this feature ViewModel is deliberate.
        Window("Catalog", id: "catalog") {
            NavigationStack {
                CatalogView(viewModel: viewModel)
            }
            .frame(minWidth: 320, minHeight: 240)
        }
    }
}
```

## Compile verification

Extract the Swift block into a temporary file, then compile it using an installed macOS SDK. Run these commands from the repository root:

```sh
python3 - <<'PYTHON'
from pathlib import Path
text = Path("skills/swiftui-architecture/references/examples.md").read_text()
source = text.split("```swift\n", 1)[1].split("\n```", 1)[0]
Path("/tmp/ItemCatalogExample.swift").write_text(source + "\n")
PYTHON

xcrun swiftc -swift-version 6 -strict-concurrency=complete -warnings-as-errors \
  -parse-as-library -target arm64-apple-macos14.0 \
  -module-cache-path /tmp/swiftui-architecture-module-cache \
  /tmp/ItemCatalogExample.swift -o /tmp/ItemCatalogExample
```

Use `x86_64-apple-macos14.0` for an Intel executable. Compilation verifies the embedded source; it does not verify interactions, accessibility, or a different project's concurrency settings. Keep temporary extracted files outside the repository to avoid maintaining duplicate examples.
