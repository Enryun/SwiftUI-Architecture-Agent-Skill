# Agent instructions

This repository is an **Agent Skills** package for SwiftUI app architecture.

- Start with [`skills/swiftui-architecture/SKILL.md`](skills/swiftui-architecture/SKILL.md) for workflows and non-negotiable rules.
- Use [`skills/swiftui-architecture/references/`](skills/swiftui-architecture/references/) for layer details, folder structure, and checklists.
- Human index: [`ARCHITECTURE-CHECKS.md`](ARCHITECTURE-CHECKS.md).

**Construction:** App / Factory wire dependencies. **Runtime:** View → ViewModel → UseCase → Manager. Factory = construction & composition (managers + optional feature graphs), not a per-call runtime layer.
