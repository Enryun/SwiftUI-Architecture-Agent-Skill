# Contributing to SwiftUI Architecture

Thanks for helping improve this Agent Skills package. Keep contributions focused on **SwiftUI layered architecture** (Factory, Manager, UseCase, ViewModel, View).

## Scope

**In scope**

- Layer rules, folder structure, naming, anti-patterns
- Scaffold / review workflows in `SKILL.md`
- Neutral Swift examples (no app-specific product names)

**Out of scope**

- Build optimization, CI, unrelated iOS topics
- Full sample Xcode projects (unless proposed as a separate optional folder)
- Tool-specific docs beyond brief install notes in README

## Skill quality

- Every `SKILL.md` must have valid YAML `name` and `description` frontmatter.
- Keep `SKILL.md` concise; put depth in `references/`.
- Preserve **recommend-first** behavior: propose structure before creating many files.
- Generalize examples — avoid names tied to a single commercial app.

## Pull requests

1. Branch from `main`.
2. Update `SKILL.md` and any affected `references/` together.
3. If you add checks, update `ARCHITECTURE-CHECKS.md`.
4. Describe what you changed and why.

## Resources

- [Agent Skills format](https://agentskills.io)
- Cursor skills: install under `~/.cursor/skills/` or `.cursor/skills/`
