# Folder structure

Adapt root names to your Xcode target layout (`App/Sources/`, single app folder, etc.). Keep **layer names** consistent.

## Recommended app layout

```
App/
├── Pages/                    # Feature modules (SwiftUI)
│   └── [Feature]/
│       ├── Model/
│       ├── View/
│       └── ViewModel/
├── Factory/
│   ├── Interface/AppFactoryProtocol.swift
│   └── Implementation/AppFactory.swift
├── Manager/
│   ├── Common/               # Logging, Navigation, Networking, Storage, …
│   │   └── [Service]/
│   │       ├── Interface/
│       │       ├── Implementation/
│   │       └── Model/
│   └── Feature/              # App-specific managers
│       └── [Domain]/
│           ├── Interface/
│           ├── Implementation/
│           └── Model/
├── UseCase/
│   └── [Domain]/
│       └── [Feature]/
│           ├── Interface/
│           └── Implementation/
├── Component/                # Shared UI
├── Constants/
└── Utility/                  # Extension+Type.swift
```

The tree shows one `AppFactory` as a starting point, not a naming or count requirement. When complexity warrants it, use `Factory/[Domain]/Interface/[Domain]FactoryProtocol.swift` and `Factory/[Domain]/Implementation/[Domain]Factory.swift`. Keep graph composition and shared lifetimes in `App`.

## Navigation (Common manager)

```
Manager/Common/Navigation/
├── Interface/NavigationManagerProtocol.swift
├── Implementation/NavigationManager.swift
└── Model/NavigationRoute.swift    # protocol; per-stack routes in feature Model/
```

Per-stack route enums conform to `NavigationRoute` in the owning feature/composition `Model/` folder. Navigation is presentation state despite its Manager location; scope each instance to an independent stack/window. See [navigation and lifetimes](navigation.md).

## Networking (Common manager)

```
Manager/Common/Networking/
├── Download/
│   ├── Interface/
│   └── Implementation/
├── Shared/Model/NetworkError.swift
└── [OtherFeature]/             # API client, WebSocket, …
```

## Storage (Common manager)

```
Manager/Common/Storage/
├── Interface/StorageProtocol.swift
├── Implementation/
└── Model/                      # SwiftData @Model types
```

## SPM / multi-target

- Shared layers → shared framework target
- Factories that construct target-specific types → corresponding app target; platform-specific managers → platform target or `Providers/`
- Move target-specific ViewModels/Views out of shared code when iOS/macOS diverge

## File placement rules

| Item | Location |
|------|----------|
| Feature screen + VM | `Pages/[Feature]/` |
| Business logic | `UseCase/[Domain]/[Feature]/` |
| System API wrapper | `Manager/Common/` or `Manager/Feature/` |
| Individual dependency creation | `Factory/` or `Factory/[Domain]/` |
| Dependency composition & shared lifetimes | `App` |
| Shared button, row style | `Component/` |
| UserDefaults key strings | `Constants/` |
| `URL.isRemoteURL` helper | `Utility/` |
