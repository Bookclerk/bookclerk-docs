# bookclerk_resolve

Sans-IO resolver for the library data model.

Storefronts describe what they sell; the host decides identity. This
crate maps one store listing to an FRBR edition, expression and work
([library data model ADR](https://github.com/Bookclerk/bookclerk/blob/main/docs/adr/library-data-model.md)):

1. [`Listing::from_draft`](listing.md#listingfrom_draft) validates and normalises what a plugin
   reported. It is the untrusted-input boundary.
2. The caller reads a [`Snapshot`](model.md#snapshot) of the stored rows that bear on the
   listing (the store from SQL, [`memory::MemoryCatalog`](memory.md#memorycatalog) from memory).
3. [`resolve`](#resolve) returns a [`Plan`]: the rows to insert, the identifiers to
   attach, the conflicts to record for an Administrator and the events
   to publish. The caller applies it as one atomic batch.

Nothing here performs I/O, so the fuzz target (`library_resolve`), the
replay test and the store all drive the same code.

| Item | Kind | Summary |
| --- | --- | --- |
| [`IdSource`](#idsource) | trait | Mints ids for new rows; the store uses random UUIDs, tests use counters. |
| [`EntityKind`](#entitykind) | enum | The kind of row a new id is minted for. |
| [`resolve`](#resolve) | function | Resolves one listing against a snapshot of the stored rows. |

## Re-exports

- [`identifiers::normalize_identifier`](identifiers.md#normalize_identifier)
- [`identifiers::IdentifierKey`](identifiers.md#identifierkey)
- [`kinds::Abridgement`](kinds.md#abridgement)
- [`kinds::CreditRole`](kinds.md#creditrole)
- [`kinds::Entitlement`](kinds.md#entitlement)
- [`kinds::ExpressionKind`](kinds.md#expressionkind)
- [`kinds::IdentifierScheme`](kinds.md#identifierscheme)
- [`kinds::Level`](kinds.md#level)
- [`kinds::MediaKind`](kinds.md#mediakind)
- [`kinds::Strength`](kinds.md#strength)
- [`kinds::WorkKind`](kinds.md#workkind)
- [`listing::stored_marketplace`](listing.md#stored_marketplace)
- [`listing::work_match_key`](listing.md#work_match_key)
- [`listing::Credit`](listing.md#credit)
- [`listing::Holder`](listing.md#holder)
- [`listing::HoldingDraft`](listing.md#holdingdraft)
- [`listing::Listing`](listing.md#listing)
- [`listing::ListingDraft`](listing.md#listingdraft)
- [`listing::ListingError`](listing.md#listingerror)
- [`listing::ListingKey`](listing.md#listingkey)
- [`listing::SeriesEntry`](listing.md#seriesentry)
- [`model::Conflict`](model.md#conflict)
- [`model::CreditWrite`](model.md#creditwrite)
- [`model::EditionIdentifierRow`](model.md#editionidentifierrow)
- [`model::EditionRow`](model.md#editionrow)
- [`model::Event`](model.md#event)
- [`model::ExpressionRow`](model.md#expressionrow)
- [`model::HoldingRow`](model.md#holdingrow)
- [`model::LinkReason`](model.md#linkreason)
- [`model::ListingRow`](model.md#listingrow)
- [`model::Override`](model.md#override)
- [`model::Plan`](model.md#plan)
- [`model::SeriesWrite`](model.md#serieswrite)
- [`model::Snapshot`](model.md#snapshot)
- [`model::WorkIdentifierRow`](model.md#workidentifierrow)
- [`model::WorkMatchKey`](model.md#workmatchkey)
- [`model::WorkParent`](model.md#workparent)
- [`model::WorkReason`](model.md#workreason)
- [`model::WorkRow`](model.md#workrow)

## Modules

- [`fuzzing`](fuzzing.md): Fuzz input decoding and the oracle the `library_resolve` target checks.
- [`identifiers`](identifiers.md): External identifiers: normalisation and the key they are stored under.
- [`kinds`](kinds.md): Closed vocabularies of the library data model.
- [`listing`](listing.md): The validated listing a storefront reports, and who holds it.
- [`memory`](memory.md): An in-memory library that reads snapshots and applies plans.
- [`model`](model.md): Stored rows the resolver reads, the snapshot of them it is given, and the plan of writes it returns.
- [`names`](names.md): Splitting a store's display string of contributor names.
- [`normalize`](normalize.md): Title and author cleaning for compares.

## Traits

### IdSource

```rust
pub trait IdSource {
    fn next_id(&mut self, kind: EntityKind) -> String;
}
```

Mints ids for new rows; the store uses random UUIDs, tests use counters.

#### IdSource::next_id

```rust
fn next_id(&mut self, kind: EntityKind) -> String
```

A fresh id, never returned before, for a new row of `kind`.

Implemented by: `CounterIds`

## Enums

### EntityKind

```rust
pub enum EntityKind {
    Work,
    Expression,
    Edition,
    Listing,
}
```

The kind of row a new id is minted for.

**Variants**

- `Work`: A work.
- `Expression`: An expression.
- `Edition`: An edition.
- `Listing`: A store listing.

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `PartialEq`

## Functions

### resolve

```rust
pub fn resolve(
    listing: &Listing,
    holding: &HoldingDraft,
    snapshot: &Snapshot,
    ids: &mut dyn IdSource,
) -> Plan
```

Resolves one listing against a snapshot of the stored rows.

#### Arguments

* `listing` - The validated listing.
* `holding` - Who holds it and how.
* `snapshot` - The stored rows that bear on it ([`Snapshot`](model.md#snapshot) states
  what the caller must include).
* `ids` - Mints ids for the rows the plan creates.

#### Returns

The plan of writes. Resolving the same listing again after applying
the plan returns a plan for which [`Plan::is_noop`](model.md#planis_noop) holds.
