# Navigation and dependency lifetimes

**One feature View → one owning ViewModel. Navigation is separate presentation state, scoped to the stack/window that uses it.** A navigation object is not a second feature ViewModel and is not a business-service dependency.

**Check deployment targets first.** The typed `NavigationStack(path:)`, value-based `NavigationLink`, and `navigationDestination` examples below require iOS 16/macOS 13 or later. For an app supporting earlier versions, retain its compatible `NavigationView`/destination approach or use an availability-gated navigation container with equivalent routes and lifetimes. Keep each stack/window scoped the same way on both paths; do not raise the deployment target to adopt an example. See Apple's [navigation migration guidance](https://developer.apple.com/documentation/swiftui/migrating-to-new-navigation-types).

## Choose the smallest navigation mechanism

| Need | Use |
|---|---|
| Simple link with no programmatic path control | A compatible `NavigationLink` pattern for the deployment target |
| Programmatic navigation owned by one container | Local `@State` with a typed `[Route]` path where NavigationStack is available; a compatible older-OS flow otherwise |
| Several descendants need to manipulate the same stack | A scoped observable `NavigationManager<Route>` in environment |
| Reusable button/row reports navigation intent | A focused action closure; parent handles navigation |
| Independent tab, sheet flow, or window | Its own path/instance, even if it uses the same route type |

Do not introduce a navigation manager, coordinator, or forwarding protocol solely because the architecture has a Navigation folder. Reuse an existing abstraction when it protects a real presentation boundary.

## Presentation boundary

Navigation is presentation state. `NavigationManager<Route>` may remain under `Manager/Common/Navigation/` for project consistency, but it is an explicit exception to the infrastructure-manager rules:

- Feature and composition Views may read the correctly scoped navigation object from SwiftUI environment and perform presentation-only path changes. This does not count as a second feature ViewModel.
- Views and composition Views own routes, paths, and navigation decisions. ViewModels and UseCases do not depend on NavigationManager, expose navigation commands, accept navigation callbacks, or mutate paths.
- For a simple presentation action, a View can use a compatible `NavigationLink`, change a supported path, or invoke a focused presentation closure directly. Do not route every navigation tap through a UseCase.
- When navigation depends on a business result, the ViewModel returns an operation outcome or exposes UI state. The View decides whether that outcome means dismissing, pushing a destination, or staying in place. Do not add route-specific intents to the ViewModel.
- Reusable visual components receive values, bindings, and action closures, not the navigation object.

Navigation may import SwiftUI for path bindings. Do not describe a protocol exposing `Binding` as framework-independent. This exception does not permit Views to access persistence, networking, purchases, or other infrastructure managers.

## Ownership and lifetime

`App` defines dependency composition and sharing, including scene-scoped composition when multiple windows are supported. Do not equate composition ownership with one process-wide instance of every dependency.

| Dependency | Typical scope |
|---|---|
| Shared infrastructure service | App/session, when consumers need the same state |
| Window presentation state | One scene/window |
| Navigation path | One independent stack within that window |
| Modal flow path | That sheet/full-screen flow |
| Feature ViewModel | Its feature/session or destination identity; sharing must be intentional |
| UseCase | Its consumers' needs; stateful workflows must not be shared accidentally |

A scene/root composition View may own local navigation state using `@State` (or `@StateObject` for an existing ObservableObject implementation). This is presentation-state ownership, not permission to construct business services in Views. App-created navigation instances are also valid when their lifetime matches the intended stack.

- Separate tabs/stacks keep independent paths. Sharing the same route type does not require sharing the same instance.
- Inject navigation at the narrowest common ancestor of its stack; ensure destinations and previews receive that instance.
- Do not inject one app-level navigation instance into every window unless synchronized navigation is a deliberate product requirement.
- Keep path state stable across body evaluation. Destination ViewModels need explicit identity and ownership; do not accidentally recreate them on redraw or reuse one mutable editor across independent destinations.
- A simple local flow can use `@State` with a typed route array directly. Add a navigation object when shared path access or reusable navigation operations justify it.

## Stack-scoped composition example

This iOS 16/macOS 13+ illustrative container receives existing feature ViewModels from app/scene composition and owns only its local presentation state. `ProfileViewModel`, `SettingsViewModel`, and their Views represent existing features. Its exact navigation APIs need a compatible branch for older deployment targets.

```swift
import SwiftUI

enum AccountRoute: Hashable {
    case settings
}

@MainActor
struct AccountCompositionView: View {
    let profileViewModel: ProfileViewModel
    let settingsViewModel: SettingsViewModel
    @State private var path: [AccountRoute] = []

    var body: some View {
        NavigationStack(path: $path) {
            ProfileView(viewModel: profileViewModel)
                .toolbar {
                    NavigationLink("Settings", value: AccountRoute.settings)
                }
                .navigationDestination(for: AccountRoute.self) { route in
                    switch route {
                    case .settings:
                        SettingsView(viewModel: settingsViewModel)
                    }
                }
        }
    }
}
```

Each feature receives one ViewModel. The container neither constructs services nor requires a wrapper ViewModel. This example deliberately uses a local path because descendants do not need shared path mutation. If they later need it, retain a scoped navigation object here and inject it into this stack's hierarchy, preserving the same lifetime.

## Routes and destination registration

- Where NavigationStack is available, prefer one `Hashable` route enum per stack with a `[Route]` path. Use `NavigationPath` when a heterogeneous path is actually needed. On older systems, retain the same route ownership with supported navigation APIs.
- Route payloads are stable identifiers or small value state. Do not embed ViewModels, services, closures, or mutable persistence objects.
- Resolve destination data through the destination's owning ViewModel and UseCases. Handle deleted or unavailable identifiers with a deliberate unavailable state.
- Register destinations within the owning stack's hierarchy, outside lazy row builders. Switch exhaustively over that stack's route enum.
- Avoid a universal route enum that forces unrelated stacks to return `EmptyView()` for unsupported cases.
- Keep tab/sidebar selection, stack paths, and modal presentation distinct. A sheet may have its own stack; it does not automatically belong in the presenting stack's path.
- Add deep-link parsing and Codable restoration only when required. Translate external input into supported route values, validate identifiers, and target the correct window/stack.

## Business results and destination state

Use the direct path for presentation-only navigation:

```text
User taps Settings → View changes navigation
```

Use the feature's business boundary when success determines the destination:

```text
User taps Save → ViewModel → UseCase → Manager
ViewModel returns an operation result
Owning View interprets the result → navigation changes
```

For example, inside an editor View with its own ViewModel and a SwiftUI `dismiss` environment action:

```swift
Button("Save") {
    Task {
        let didSave = await viewModel.save()
        guard didSave, !Task.isCancelled else { return }
        dismiss()
    }
}
```

Here `save()` reports success and keeps failure details in the ViewModel's UI state. It does not know that this particular View dismisses after saving. Use a richer result type when the operation has multiple meaningful outcomes.

- Prefer handling the returned result at the View's action site instead of adding a navigation-event property to the ViewModel.
- If a View observes durable UI state, it owns any protection against repeated navigation. A persistent `saved == true` flag must not push a destination on every reappearance.
- Never move business validation into the View to make this separation work; the View interprets an already-decided outcome only for presentation.
- Decide what happens if an async operation finishes after its presentation flow is dismissed; do not let a stale result navigate a different flow.
- A route identifier selects data; it does not own a ViewModel. App/scene composition supplies destination dependencies and an explicit owner retains destination state for that presentation identity.
- Reuse a feature ViewModel across destinations only when they intentionally share the same feature state. Independent editors need independent drafts, even when they edit the same record.
- Do not add a global cache of destination ViewModels or a factory closure to every View as a shortcut for lifetime ownership.

## Navigation API behavior

Keep the API as small as the app needs. A typed path plus pop/reset operations is often sufficient; do not require a coordinator layer merely to forward calls.

- Isolate observable navigation state and its protocol requirements to `@MainActor`.
- Empty-stack pop is a safe no-op.
- Specify whether pop-to chooses the first or most recent matching route when values repeat. If individual occurrences matter, give entries their own identity.
- For pop-before, remove the suffix by the selected occurrence's index. Looking up its predecessor again by value can choose the wrong duplicate.
- Specify missing-target behavior (for example, no-op or reset) rather than silently mixing policies across helpers.
- A callback invoked immediately after path mutation does not signal completion of a SwiftUI navigation animation. Prefer no callback unless a concrete caller needs a clearly documented event.
- When verifying changes, inspect empty paths, repeated routes, missing targets, back gestures, and independent stacks/windows. Follow the project's build/test permissions.

## Review checklist

- [ ] Every independent stack/window has an explicit path owner.
- [ ] Navigation environment scope matches the stack; previews provide required environment objects.
- [ ] Views handle presentation; ViewModels/UseCases never receive navigation objects.
- [ ] Routes contain stable values and have supported destinations; no `EmptyView()` fallback for unrelated routes.
- [ ] Back gestures and dismissals update the same state used by programmatic navigation.
- [ ] Modal dismissal has a deliberate reset/preserve policy for its nested path.
- [ ] Destination identity, draft ownership, and repeated handling of operation results are handled deliberately.
- [ ] Optional pop helpers define repeated-route and missing-target behavior.

## Sources

- [Apple: Understanding the navigation stack](https://developer.apple.com/documentation/swiftui/understanding-the-navigation-stack)
- [Apple: WindowGroup](https://developer.apple.com/documentation/swiftui/windowgroup)
