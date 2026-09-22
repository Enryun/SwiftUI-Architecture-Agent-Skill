# Agent instructions

This repository is an **Agent Skills** package for SwiftUI app architecture.

- Start with [`skills/swiftui-architecture/SKILL.md`](skills/swiftui-architecture/SKILL.md) for workflows and non-negotiable rules.
- Use [`skills/swiftui-architecture/references/`](skills/swiftui-architecture/references/) for layer details, folder structure, and checklists.
- Human index: [`ARCHITECTURE-CHECKS.md`](ARCHITECTURE-CHECKS.md).

**Construction:** `App` composes dependencies and owns shared lifetimes. Factories create one Manager, UseCase, or ViewModel per method using supplied dependencies. Start with one factory; split by domain or platform when current complexity warrants it. **Runtime:** View → ViewModel → UseCase → Manager. Factories never hide feature/application graphs or perform business logic.

ViewModels may depend on multiple focused UseCase protocols. ViewModel → ViewModel and UseCase → UseCase dependencies are forbidden; UseCases call manager protocols directly.
