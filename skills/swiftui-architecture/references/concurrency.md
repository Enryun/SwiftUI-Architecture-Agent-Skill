# Concurrency boundaries

This skill defines **where concurrency responsibilities belong in the architecture**. It does not replace a dedicated Swift Concurrency guide. For compiler diagnostics, Swift 6 migration, actors, `Sendable`, task isolation, async streams, or performance, use [Swift Concurrency Agent Skill](https://github.com/AvdLee/Swift-Concurrency-Agent-Skill) when available, together with the project's actual compiler settings.

Do not infer execution behavior from a layer name alone. An `async` UseCase does not automatically mean that all work runs away from the main actor; isolation, task entry context, compiler version, and project settings determine that behavior.

## Isolation

| Layer | Default |
|-------|---------|
| ViewModel | `@MainActor`; owns UI state and user-intent task lifetime |
| View | SwiftUI presentation; starts or cancels UI tasks and owns navigation |
| UseCase | Owns the feature's business workflow; choose isolation explicitly rather than assuming a background executor |
| Manager | Owns infrastructure state; use value types, existing framework isolation, or an actor when shared mutable state requires it |
| Factory | Follows the isolation of the objects it creates; ViewModel construction is usually `@MainActor` |

Do **not** add `@MainActor` only to silence compiler errors.

## ViewModel + View

```swift
// View
Button("Load") {
    Task { await viewModel.load() }
}
```

- Long work belongs to a UseCase or manager actor, but verify its isolation. Moving a function into a UseCase is not by itself proof that it runs off `@MainActor`.
- ViewModel state updates remain on `@MainActor`.
- A `Task {}` created by a View commonly inherits the caller's actor and lifetime context. Choose the task entry isolation intentionally; do not use `Task.detached` as a generic background-work fix.
- The ViewModel or View owns the task lifetime. Decide whether leaving the screen cancels the work and what cancellation should do to visible state.

## Request policy

Every async ViewModel action should have an explicit policy for overlapping work:

- **Serialize or ignore duplicates** when only one request should be active.
- **Cancel and replace** when the newest user intent supersedes the previous request.
- **Allow concurrency** only when results can safely merge or are independently identified.

Do not let an older response overwrite newer state accidentally. Treat cancellation as a control-flow outcome, not automatically as a user-visible failure. Check cancellation in long-running UseCase or manager work where appropriate.

## UseCase

- Owns structured concurrency for the feature and documents the isolation boundary when it matters.
- Use synchronous functions for synchronous work; do not add `async` merely because a function belongs to a UseCase.
- `async let` for fixed parallelism; `withTaskGroup` for dynamic.
- `try Task.checkCancellation()` in loops and long operations.

## Manager

- Prefer immutable structs and `Sendable` models.
- Shared mutable caches → `actor`, not raw locks in ViewModels.

## Avoid unless documented

- `Task.detached`
- `@unchecked Sendable`
- `nonisolated(unsafe)`

## Settings and escalation

Check project concurrency mode before “fixing” warnings:

- Xcode: `SWIFT_STRICT_CONCURRENCY`, `SWIFT_DEFAULT_ACTOR_ISOLATION`
- SPM: `StrictConcurrency` experimental features

Before interpreting an isolation or `Sendable` diagnostic, inspect the target's Swift language mode, strict-concurrency level, default actor isolation, and upcoming features. If the issue involves those settings or crosses module/actor boundaries, follow the dedicated concurrency skill rather than inventing an architecture-specific workaround.

## Verification before merge

- [ ] ViewModels UI-bound only
- [ ] Long CPU/IO has a deliberate isolation/execution strategy; no assumption that `async` alone moves it off `@MainActor`
- [ ] Shared mutable state has one explicit owner: actor, main actor, or another documented isolation domain
- [ ] Cancellation and overlapping-request policy are explicit
- [ ] Values crossing isolation boundaries satisfy the project's transfer rules
- [ ] Project settings were checked before applying a concurrency fix
- [ ] Build clean under project's Swift version settings when build verification is authorized

## Companion skill

Use the [Swift Concurrency Agent Skill](https://github.com/AvdLee/Swift-Concurrency-Agent-Skill) for detailed Swift 6 concurrency work. Keep this architecture skill responsible for layer ownership, task lifetime, request policy, and dependency boundaries; keep compiler-specific remedies in the companion skill so the two guides do not drift.
