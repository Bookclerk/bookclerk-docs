# bookclerk_resolve::memory

An in-memory library that reads snapshots and applies plans.

It keeps the same rows and the same constraints as the SQL tables
(unique strong identifiers, unique listings, rows that reference rows
which exist), so tests and the fuzz oracle can resolve sequences of
listings without a database. The store's SQL reads follow
[`MemoryCatalog::snapshot`](#memorycatalogsnapshot) and its batch writes follow
[`MemoryCatalog::apply`](#memorycatalogapply).

| Item | Kind | Summary |
| --- | --- | --- |
| [`CounterIds`](#counterids) | struct | Mints sequential ids (`work-1`, `edition-1`, …) for tests and fuzzing. |
| [`ApplyError`](#applyerror) | struct | A plan broke a constraint the SQL schema enforces. |
| [`LinkRecord`](#linkrecord) | struct | What linked one listing, kept for the oracle. |
| [`MemoryCatalog`](#memorycatalog) | struct | The in-memory library. |

## Structs

### CounterIds

```rust
pub struct CounterIds {
    /* private fields */
}
```

Mints sequential ids (`work-1`, `edition-1`, …) for tests and fuzzing.

Implements: `Clone`, `Debug`, `Default`, `IdSource`

### ApplyError

```rust
pub struct ApplyError(pub String);
```

A plan broke a constraint the SQL schema enforces.

Implements: `Clone`, `Debug`, `Display`, `Eq`, `Error`, `PartialEq`

### LinkRecord

```rust
pub struct LinkRecord {
    pub listing: Listing,
    pub holding: HoldingDraft,
    pub edition_id: String,
    pub link: LinkReason,
    pub work_reason: Option<WorkReason>,
    pub parent_work_id: Option<String>,
}
```

What linked one listing, kept for the oracle.

**Fields**

- `listing`: The listing as resolved.
- `holding`: Who held it.
- `edition_id`: The edition it reached.
- `link`: Why.
- `work_reason`: How a new edition found its work.
- `parent_work_id`: The parent show's work when the listing was linked, for an episode.

Implements: `Clone`, `Debug`, `PartialEq`

### MemoryCatalog

```rust
pub struct MemoryCatalog {
    pub works: BTreeMap<String, WorkRow>,
    pub expressions: BTreeMap<String, ExpressionRow>,
    pub editions: BTreeMap<String, EditionRow>,
    pub edition_identifiers: BTreeSet<EditionIdentifierRow>,
    pub work_identifiers: BTreeMap<IdentifierKey, String>,
    pub listings: BTreeMap<ListingKey, ListingRow>,
    pub holdings: BTreeMap<(String, Holder), HoldingRow>,
    pub credits: Vec<(Level, String, Credit, u32)>,
    pub series: Vec<(String, SeriesEntry)>,
    pub overrides: Vec<Override>,
    pub conflicts: Vec<Conflict>,
    pub links: Vec<LinkRecord>,
    pub ids: CounterIds,
}
```

The in-memory library.

**Fields**

- `works`: Works by id.
- `expressions`: Expressions by id.
- `editions`: Editions by id.
- `edition_identifiers`: Edition identifiers.
- `work_identifiers`: Work identifiers by key (unique).
- `listings`: Listings by key.
- `holdings`: Holdings by listing id and holder.
- `credits`: Credits written: level, target id, credit and billing position.
- `series`: Series entries: work id and entry.
- `overrides`: Overrides.
- `conflicts`: Conflicts recorded, de-duplicated.
- `links`: Every link made, in order.
- `ids`: Ids for new rows.

#### MemoryCatalog::snapshot

```rust
pub fn snapshot(&self, listing: &Listing, holder: &Holder) -> Snapshot
```

Reads the snapshot [`resolve`](README.md#resolve) needs for `listing` and `holder`.

##### Arguments

* `listing` - The listing to resolve.
* `holder` - Who holds it.

##### Returns

Every stored row that bears on the listing, per [`Snapshot`](model.md#snapshot).

#### MemoryCatalog::apply

```rust
pub fn apply(&mut self, plan: &Plan) -> Result<(), ApplyError>
```

Applies `plan`, checking the constraints the SQL schema enforces.

##### Arguments

* `plan` - A plan [`resolve`](README.md#resolve) returned for a snapshot of this catalog.

##### Errors

Returns [`ApplyError`](#applyerror) when a write would break a constraint; the
catalog may then be partly changed, as a failed batch never is.

#### MemoryCatalog::record

```rust
pub fn record(
    &mut self,
    listing: &Listing,
    holding: &HoldingDraft,
) -> Result<Plan, ApplyError>
```

Resolves `listing` for `holding` and applies the plan.

##### Arguments

* `listing` - The listing.
* `holding` - Who holds it.

##### Returns

The applied plan.

##### Errors

Returns [`ApplyError`](#applyerror) when the plan breaks a constraint.

#### MemoryCatalog::record_catalog

```rust
pub fn record_catalog(&mut self, listing: &Listing) -> Result<Plan, ApplyError>
```

Resolves a catalog `listing` (nobody holds it) and applies the plan.

Unlike [`Self::record`](#memorycatalogrecord), nothing is added to the link log the
fuzzing replay walks, since there is no holding to replay.

##### Returns

The applied plan.

##### Errors

Returns [`ApplyError`](#applyerror) when the plan breaks a constraint.

#### MemoryCatalog::work_of_edition

```rust
pub fn work_of_edition(&self, edition_id: &str) -> Option<&WorkRow>
```

The work an edition realises.

Implements: `Clone`, `Debug`, `Default`
