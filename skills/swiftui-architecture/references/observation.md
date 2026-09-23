# ViewModel observation and deployment targets

Keep the architectural rule separate from the observation mechanism: each feature View receives one owning ViewModel, and that ViewModel owns UI state on the main actor. Choose how the View observes it from the app's supported OS versions, Swift toolchain, and existing conventions.

| App support | ViewModel and View |
|-------------|--------------------|
| iOS 17 / macOS 14 or later, with a toolchain that supports Observation | Prefer `@MainActor @Observable` and the corresponding Observation data flow. |
| Earlier supported OS versions | Use `@MainActor` with `ObservableObject` and publish state that the View must observe. An injected ViewModel is usually read with `@ObservedObject`; use `@StateObject` where supported when the App, Scene, or View intentionally creates and owns its lifetime. |

The choice is per target and existing codebase. Do not raise the deployment target, mix observation systems into a ViewModel, or migrate working `ObservableObject` features merely to match an example. `@MainActor` remains an isolation decision in either approach. A ViewModel injected by the composition root retains its intended lifetime regardless of which observation API the View uses.

For example, an app supporting iOS 15 can keep an App-created ViewModel and inject it into its feature View:

```swift
import SwiftUI

@MainActor
final class DetailViewModel: ObservableObject {
    @Published private(set) var title = "Detail"
}

struct DetailView: View {
    @ObservedObject var viewModel: DetailViewModel

    var body: some View {
        Text(viewModel.title)
    }
}
```

The App or scene composition supplies this ViewModel; the View observes it without taking over construction or ownership. Publish only state that must update the View.

For targets below the availability of a property wrapper, follow the project's compatible ownership approach. In particular, do not mechanically replace `@ObservedObject` with `@StateObject` when the View does not own construction.

The [end-to-end example](examples.md) intentionally targets macOS 14+ and uses `@Observable`. It illustrates the layer relationships, not a minimum OS requirement for this skill. When adapting it to an older target, choose compatible observation in both the ViewModel and View, then build for that target.

Check the app's actual deployment targets and Swift language/toolchain settings before scaffolding; build the affected target and verify state updates on a supported older runtime when available.

Apple guidance: [Managing model data with Observation](https://developer.apple.com/documentation/swiftui/managing-model-data-in-your-app), [Monitoring model data changes on earlier systems](https://developer.apple.com/documentation/swiftui/monitoring-model-data-changes-in-your-app).
