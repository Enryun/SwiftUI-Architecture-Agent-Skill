# Concurrency

Apply across all layers. For deep Swift 6 migration, use a dedicated concurrency skill or Apple docs.

## Isolation

| Layer | Default |
|-------|---------|
| ViewModel | `@MainActor` |
| View | Main actor (SwiftUI) |
| UseCase | Nonisolated or custom actor — owns async workflow |
| Manager | Prefer value types / actors for shared mutable state |
| Factory | Individual creation; ViewModel builders and their protocol requirements are `@MainActor`; other methods follow the isolation of their dependencies |

Do **not** add `@MainActor` only to silence compiler errors.

## ViewModel + View

```swift
// View
Button("Load") {
    Task { await viewModel.load() }
}
```

- Long work runs in UseCase (or manager `actor`), not on `@MainActor`.
- Update `state` on MainActor after async work completes.

## UseCase

- Owns structured concurrency for the feature.
- `async let` for fixed parallelism; `withTaskGroup` for dynamic.
- `try Task.checkCancellation()` in loops and long operations.

## Manager

- Prefer immutable structs and `Sendable` models.
- Shared mutable caches → `actor`, not raw locks in ViewModels.

## Avoid unless documented

- `Task.detached`
- `@unchecked Sendable`
- `nonisolated(unsafe)`

## Settings

Check project concurrency mode before “fixing” warnings:

- Xcode: `SWIFT_STRICT_CONCURRENCY`, `SWIFT_DEFAULT_ACTOR_ISOLATION`
- SPM: `StrictConcurrency` experimental features

## Verification before merge

- [ ] ViewModels UI-bound only
- [ ] No long CPU/IO on `@MainActor`
- [ ] Shared state behind actors or single isolation domain
- [ ] Build clean under project's Swift version settings
