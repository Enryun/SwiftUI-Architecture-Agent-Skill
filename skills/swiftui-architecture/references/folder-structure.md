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
│   └── [Domain]/
│       ├── Interface/
│       ├── Implementation/
│       └── Model/
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

## Navigation (Common manager)

```
Manager/Common/Navigation/
├── Interface/NavigationManagerProtocol.swift
├── Implementation/NavigationManager.swift
└── Model/NavigationRoute.swift    # protocol; per-stack routes in feature Model/
```

Per-feature route enums conform to `NavigationRoute` in `Pages/[Feature]/Model/`.

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
- Platform-specific factories or managers → platform target or `Providers/`
- Move target-specific ViewModels/Views out of shared code when iOS/macOS diverge

## File placement rules

| Item | Location |
|------|----------|
| Feature screen + VM | `Pages/[Feature]/` |
| Business logic | `UseCase/[Domain]/[Feature]/` |
| System API wrapper | `Manager/Common/` or `Manager/Feature/` |
| Manager creation | `Factory/[Domain]/` |
| Shared button, row style | `Component/` |
| UserDefaults key strings | `Constants/` |
| `URL.isRemoteURL` helper | `Utility/` |
