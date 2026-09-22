# Folder structure

Adapt root names to your Xcode target layout (`App/Sources/`, single app folder, etc.). Keep **layer names** consistent.

## Recommended app layout

```
App/
├── Pages/                    # Feature modules (SwiftUI)
│   └── [Feature]/
│       ├── Model/            # Presentation-only models
│       ├── View/
│       └── ViewModel/
├── Domain/                   # Plain business values, when needed
│   └── [Domain]/Model/
├── Factory/
│   ├── Interface/AppFactoryProtocol.swift
│   └── Implementation/AppFactory.swift
├── Manager/
│   ├── Common/               # Logging, Navigation, Networking, Storage, …
│   │   └── [Service]/
│   │       ├── Interface/
│   │       ├── Implementation/
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

The Domain folder is a suggested placement, not a required new layer or migration. Domain folder names are adaptable: use an existing equivalent if present. Create only needed model representations and folders. Domain values do not depend on Pages or persistence implementations; see [model ownership](models.md).

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
└── Model/                      # Storage-specific records, if needed
```

## SPM / multi-target

- Shared layers → shared framework target
- Factories that construct target-specific types → corresponding app target; platform-specific managers → platform target or `Providers/`
- Move target-specific ViewModels/Views out of shared code when iOS/macOS diverge

## File placement rules

| Item | Location |
|------|----------|
| Feature screen + VM | `Pages/[Feature]/` |
| Plain business/domain values | `Domain/[Domain]/Model/` or existing equivalent |
| UI-only models/drafts | `Pages/[Feature]/Model/` |
| Storage records / transport DTOs | Owning manager implementation or its `Model/` |
| Business logic | `UseCase/[Domain]/[Feature]/` |
| System API wrapper | `Manager/Common/` or `Manager/Feature/` |
| Individual dependency creation | `Factory/` or `Factory/[Domain]/` |
| Dependency composition & shared lifetimes | `App` |
| Shared button, row style | `Component/` |
| UserDefaults key strings | `Constants/` |
| `URL.isRemoteURL` helper | `Utility/` |
