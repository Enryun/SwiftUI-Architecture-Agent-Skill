# Model ownership and persistence boundaries

**Reuse a plain value model across layers when it fits. Split representations only when storage, transport, presentation, or isolation requirements actually differ.** There is no requirement to create a database model, domain model, and UI model for every entity.

## Guidance, not a mandatory three-model structure

Domain, presentation, and persistence models describe responsibilities to consider, not three types every feature must implement. Start with the simplest representation that fits the feature and the existing project. Add a separate type only when a concrete difference in behavior, lifecycle, schema, formatting, or isolation makes it useful.

- A View may display a domain value directly; that does not make it a separate presentation model.
- Storage may encode/decode the same plain domain value when the schema fits.
- Domain models include both entities with enduring identity and value objects defined by their values. Do not force identity onto every model.
- Folder names and model suffixes are suggestions. Do not move files or introduce mapper abstractions merely to match this guide.
- During review, explain the specific coupling or ownership problem before recommending a split. Prefer an incremental change over a model-wide migration.

Preserve behavior and valid framework/isolation requirements. Flexibility in representation does not make an unsafe actor transfer safe.

## Choose ownership by meaning

| Model kind | Responsibility | Suggested home |
|---|---|---|
| Domain value | Data used by business operations, independent of UI and storage frameworks | `Domain/[Domain]/Model/` or the existing equivalent shared domain location |
| Persistence record | Database schema, relationships, framework-managed identity and lifecycle | Relevant storage implementation's `Model/` |
| Transport DTO | External API payload shape and serialization details | Relevant networking manager's `Model/` |
| Presentation model | Form drafts, selection, display-only formatting, screen modes | `Pages/[Feature]/Model/` or a private nested ViewModel type |

These are responsibilities, not mandatory folders or suffixes. Keep the project's established layout when its module dependencies already respect the boundary. A domain value used by one business feature still belongs outside the UI module if Managers or UseCases need it. Do not place all shared models under Manager merely because multiple consumers use them.

Domain values may be used directly by Views and ViewModels. Add a separate row/display model only when it captures a real presentation transformation. A value can also conform to `Codable` without becoming a separate DTO if the wire format and domain meaning already align.

## When to convert

- **Plain value throughout:** an in-memory manager returns `CatalogItem`, and the UseCase and ViewModel use that same type. No mapping layer is necessary; see the [complete example](examples.md).
- **Framework-managed persistence:** keep mutable database records and contexts inside the owning storage implementation/isolation domain. Return domain snapshots or identifiers through the business-facing manager protocol.
- **Different external representation:** map API-specific field names, optionality, or schema into useful values in the integration implementation. Business validation and policy stay in UseCases.
- **Different UI representation:** format display text or build presentation state in the ViewModel, while retaining identifiers for subsequent actions.

For example:

```text
StoredWatch (database record)
  → storage implementation maps fields
  → Watch (plain domain value)
  → UseCase → ViewModel → View
```

Only add `WatchRowState` when the UI needs its own representation. Do not create a public mapper protocol, mapper factory, or repository layer just to copy fields once; a private conversion helper is enough.

## Protocol and isolation boundaries

- Business-facing storage protocols expose domain values and operation parameters, not framework contexts or requirements such as `Item: PersistentModel`. A framework-specific generic helper may exist inside the storage implementation.
- Map stored records while inside their valid context/isolation domain, before returning results across that boundary.
- Values crossing actors/tasks must satisfy the applicable transfer/isolation rules. Prefer `Sendable` value types with safe stored properties where transfer is required; a struct containing mutable reference objects is not automatically safe.
- Do not add `@unchecked Sendable` to a persistence record to make it cross actors.
- A snapshot is not live database state. Updates go back through ViewModel → UseCase → Manager using identifiers and values; reload or observe domain-value updates when freshness is required.
- UI drafts may differ from saved state intentionally. Make save/cancel behavior explicit instead of mutating a shared persistence object directly from a binding.

## Review checklist

- [ ] Model responsibilities are understood; existing locations are retained unless they cause a concrete dependency problem.
- [ ] Plain values are reused where their meaning matches; duplicate representations are justified.
- [ ] Business-facing APIs do not require UI or persistence-framework types.
- [ ] Storage/transport conversion stays with the integration; business rules stay in UseCases.
- [ ] Cross-isolation values are safe to transfer; persistence contexts/records stay confined.
- [ ] Snapshot freshness and draft save/cancel behavior are deliberate.
