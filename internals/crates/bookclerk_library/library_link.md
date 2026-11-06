# bookclerk_library::library_link

Named-atomic parameters for writing a resolved listing.

[`bookclerk_resolve::resolve`](../bookclerk_resolve/README.md#resolve) decides what a listing links to; these
types carry its plan to the database as host-compiled SQL batches
([`crate::DbAtomicParams::LinkListing`](README.md#dbatomicparams) and
[`crate::DbAtomicParams::WriteCredits`](README.md#dbatomicparams)). The link is one atomic batch,
never a read-modify-write across statements, because D1 has no
interactive transactions: every insert is guarded in its own `WHERE`
clause, and the batch answers `retry` when another writer got there
first, so the caller resolves again against the new rows.

| Item | Kind | Summary |
| --- | --- | --- |
| [`LinkWorkRow`](#linkworkrow) | struct | A new `works` row. |
| [`LinkExpressionRow`](#linkexpressionrow) | struct | A new `expressions` row. |
| [`LinkEditionRow`](#linkeditionrow) | struct | A new `editions` row. |
| [`LinkWorkParent`](#linkworkparent) | struct | The show a stored, show-less episode work gains. |
| [`LinkWorkMatchKey`](#linkworkmatchkey) | struct | The matcher key a stored work without one gains. |
| [`LinkEditionFill`](#linkeditionfill) | struct | Fields a listing fills on a stored edition that lacks them. |
| [`LinkExpressionFill`](#linkexpressionfill) | struct | Fields a listing fills on a stored expression that lacks them. |
| [`LinkIdentifierRow`](#linkidentifierrow) | struct | An `edition_identifiers` row to attach. |
| [`LinkWorkIdentifierRow`](#linkworkidentifierrow) | struct | A `work_identifiers` row to attach. |
| [`LinkConflictRow`](#linkconflictrow) | struct | A `resolution_conflicts` row to record. |
| [`LinkEvent`](#linkevent) | struct | A `core.events` row the link publishes when it commits. |
| [`ListingLinkParams`](#listinglinkparams) | struct | Everything one listing's link writes, as one atomic batch. |
| [`ContributorRow`](#contributorrow) | struct | A `contributors` row (id derived from the cleaned name). |
| [`CreditRow`](#creditrow) | struct | A `credits` row. |
| [`SeriesRow`](#seriesrow) | struct | A `series` row (id derived from the cleaned name). |
| [`SeriesEntryRow`](#seriesentryrow) | struct | A `series_entries` row. |
| [`CreditBatchParams`](#creditbatchparams) | struct | One batch of idempotent credit and series inserts. |

## Structs

### LinkWorkRow

```rust
pub struct LinkWorkRow {
    pub id: String,
    pub kind: String,
    pub title: String,
    pub match_key: String,
    pub parent_work_id: Option<String>,
}
```

A new `works` row.

**Fields**

- `id`: Work id.
- `kind`: Stored work kind (`book`, …).
- `title`: Display title.
- `match_key`: The work matcher's key; empty when never matched by text.
- `parent_work_id`: Parent show, for a podcast episode.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### LinkExpressionRow

```rust
pub struct LinkExpressionRow {
    pub id: String,
    pub work_id: String,
    pub kind: String,
    pub language: Option<String>,
    pub abridgement: String,
    pub credit_key: String,
}
```

A new `expressions` row.

**Fields**

- `id`: Expression id.
- `work_id`: The work it realises.
- `kind`: Stored expression kind.
- `language`: Lower-cased language code when known.
- `abridgement`: Stored abridgement.
- `credit_key`: Sorted, cleaned performer names.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### LinkEditionRow

```rust
pub struct LinkEditionRow {
    pub id: String,
    pub expression_id: String,
    pub media_kind: String,
    pub title: String,
    pub subtitle: Option<String>,
    pub published_at: Option<String>,
    pub duration_minutes: Option<i64>,
}
```

A new `editions` row.

**Fields**

- `id`: Edition id.
- `expression_id`: The expression it releases.
- `media_kind`: Stored media kind.
- `title`: Display title.
- `subtitle`: Subtitle.
- `published_at`: Publication date (RFC 3339).
- `duration_minutes`: Runtime in whole minutes.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### LinkWorkParent

```rust
pub struct LinkWorkParent {
    pub work_id: String,
    pub parent_work_id: String,
}
```

The show a stored, show-less episode work gains.

**Fields**

- `work_id`: The episode work.
- `parent_work_id`: The show work.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### LinkWorkMatchKey

```rust
pub struct LinkWorkMatchKey {
    pub work_id: String,
    pub kind: String,
    pub parent_work_id: Option<String>,
    pub match_key: String,
}
```

The matcher key a stored work without one gains.

**Fields**

- `work_id`: The work, whose stored key is empty.
- `kind`: Its stored kind; only works of this kind compete for the key.
- `parent_work_id`: Its show, for an episode; episodes only compete within one show.
- `match_key`: The key it gains.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### LinkEditionFill

```rust
pub struct LinkEditionFill {
    pub subtitle: Option<String>,
    pub published_at: Option<String>,
    pub duration_minutes: Option<i64>,
}
```

Fields a listing fills on a stored edition that lacks them.

**Fields**

- `subtitle`: Subtitle.
- `published_at`: Publication date (RFC 3339).
- `duration_minutes`: Runtime in whole minutes.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### LinkExpressionFill

```rust
pub struct LinkExpressionFill {
    pub expression_id: String,
    pub kind: String,
    pub language: Option<String>,
    pub abridgement: String,
    pub credit_key: String,
}
```

Fields a listing fills on a stored expression that lacks them.

Each value is written only where the stored one is unknown: a NULL
language, an `unknown` abridgement or an empty performer key.

**Fields**

- `expression_id`: The stored expression to fill.
- `kind`: Stored expression kind; a row of another kind is left alone.
- `language`: BCP 47 language tag.
- `abridgement`: Stored abridgement (`unknown` fills nothing).
- `credit_key`: Normalised performer credit key (empty fills nothing).

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### LinkIdentifierRow

```rust
pub struct LinkIdentifierRow {
    pub scheme: String,
    pub value: String,
    pub marketplace: String,
    pub strength: String,
    pub strong_key: Option<String>,
}
```

An `edition_identifiers` row to attach.

**Fields**

- `scheme`: Stored scheme.
- `value`: Normalised value.
- `marketplace`: Lower-cased marketplace for a scoped scheme; empty otherwise.
- `strength`: `strong` or `weak`.
- `strong_key`: Unique key of a strong identifier.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### LinkWorkIdentifierRow

```rust
pub struct LinkWorkIdentifierRow {
    pub scheme: String,
    pub value: String,
}
```

A `work_identifiers` row to attach.

**Fields**

- `scheme`: Stored scheme.
- `value`: Normalised value.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### LinkConflictRow

```rust
pub struct LinkConflictRow {
    pub kind: String,
    pub subject_ids: String,
}
```

A `resolution_conflicts` row to record.

**Fields**

- `kind`: Stored conflict kind (`strong_identifiers`, …).
- `subject_ids`: JSON array of the edition or work ids involved.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### LinkEvent

```rust
pub struct LinkEvent {
    pub id: String,
    pub event_type: String,
    pub account_id: String,
    pub source: String,
    pub dedup_key: String,
    pub payload: String,
    pub ordering_key: String,
}
```

A `core.events` row the link publishes when it commits.

**Fields**

- `id`: Event id (UUID).
- `event_type`: Event type (`edition_linked`, `edition_updated`, `holding_added`).
- `account_id`: Tenant account id; empty for library-wide events.
- `source`: PluginKey of the storefront that reported the listing.
- `dedup_key`: Unique with the event type; a replay never publishes twice.
- `payload`: Bounded JSON payload.
- `ordering_key`: FIFO key copied onto deliveries.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### ListingLinkParams

```rust
pub struct ListingLinkParams {
    pub listing_id: String,
    pub listing_is_new: bool,
    pub plugin_key: String,
    pub marketplace: String,
    pub product_id: String,
    pub title: String,
    pub categories: Option<String>,
    pub edition_id: String,
    pub stored_edition_id: Option<String>,
    pub gate_strong_keys: Vec<String>,
    pub new_work: Option<LinkWorkRow>,
    pub new_expression: Option<LinkExpressionRow>,
    pub new_edition: Option<LinkEditionRow>,
    pub edition_fill: Option<LinkEditionFill>,
    pub expression_fill: Option<LinkExpressionFill>,
    pub work_parent: Option<LinkWorkParent>,
    pub known_rivals: Vec<String>,
    pub work_match_key: Option<LinkWorkMatchKey>,
    pub edition_identifiers: Vec<LinkIdentifierRow>,
    pub work_id: String,
    pub work_identifiers: Vec<LinkWorkIdentifierRow>,
    pub holder: String,
    pub entitlement: String,
    pub purchased_at: Option<String>,
    pub conflicts: Vec<LinkConflictRow>,
    pub events: Vec<LinkEvent>,
}
```

Everything one listing's link writes, as one atomic batch.

**Fields**

- `listing_id`: Listing id (new or stored).
- `listing_is_new`: True when the batch inserts the listing.
- `plugin_key`: Storefront PluginKey.
- `marketplace`: Lower-cased marketplace.
- `product_id`: The store's product id.
- `title`: Title as the store lists it.
- `categories`: Store category labels.
- `edition_id`: The edition the listing reaches.
- `stored_edition_id`: The edition a stored listing sold when the plan was made; the
  refresh only applies while it still does, so a concurrent relink
  makes the batch answer `retry` instead of being overwritten.
- `gate_strong_keys`: Strong keys that must still be unowned for a new edition to be
  created; another writer taking one makes the batch answer `retry`.
- `new_work`: A new work.
- `new_expression`: A new expression.
- `new_edition`: A new edition.
- `edition_fill`: Fields to fill on a stored edition.
- `expression_fill`: Fields to fill on the stored edition's expression.
- `work_parent`: A show link for a stored episode work that has none.
- `known_rivals`: Works with the new work's matcher key that the snapshot already
  held; any other same-key work stored since holds the new work back.
- `work_match_key`: A matcher key for a stored work that has none.
- `edition_identifiers`: Edition identifiers to attach.
- `work_id`: The work work identifiers attach to.
- `work_identifiers`: Work identifiers to attach.
- `holder`: Stored holder (`account:<id>` or `node:<id>`).
- `entitlement`: Stored entitlement.
- `purchased_at`: When the holder bought or added it (RFC 3339).
- `conflicts`: Conflicts to record.
- `events`: Events to publish.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### ContributorRow

```rust
pub struct ContributorRow {
    pub id: String,
    pub name: String,
    pub sort_name: String,
    pub name_key: String,
}
```

A `contributors` row (id derived from the cleaned name).

**Fields**

- `id`: Contributor id.
- `name`: Display name.
- `sort_name`: Name for sorting.
- `name_key`: Cleaned name; unique.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### CreditRow

```rust
pub struct CreditRow {
    pub target_kind: String,
    pub target_id: String,
    pub contributor_id: String,
    pub role: String,
    pub position: i64,
}
```

A `credits` row.

**Fields**

- `target_kind`: `work`, `expression` or `edition`.
- `target_id`: Target id.
- `contributor_id`: Contributor id.
- `role`: Stored role.
- `position`: Billing position.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### SeriesRow

```rust
pub struct SeriesRow {
    pub id: String,
    pub name: String,
    pub name_key: String,
}
```

A `series` row (id derived from the cleaned name).

**Fields**

- `id`: Series id.
- `name`: Display name.
- `name_key`: Cleaned name; unique.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### SeriesEntryRow

```rust
pub struct SeriesEntryRow {
    pub series_id: String,
    pub work_id: String,
    pub position: Option<String>,
    pub sort_position: Option<f64>,
}
```

A `series_entries` row.

**Fields**

- `series_id`: Series id.
- `work_id`: Work id.
- `position`: Position label.
- `sort_position`: Sortable position.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `PartialEq`, `Serialize`

### CreditBatchParams

```rust
pub struct CreditBatchParams {
    pub contributors: Vec<ContributorRow>,
    pub credits: Vec<CreditRow>,
    pub series: Vec<SeriesRow>,
    pub series_entries: Vec<SeriesEntryRow>,
}
```

One batch of idempotent credit and series inserts.

Contributors and series precede the rows that reference them, in this
batch or an earlier one.

**Fields**

- `contributors`: Contributors to insert when missing.
- `credits`: Credits to insert when missing.
- `series`: Series to insert when missing.
- `series_entries`: Series entries to insert when missing.

#### CreditBatchParams::statement_count

```rust
pub fn statement_count(&self) -> usize
```

The number of insert statements the batch compiles to.

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `PartialEq`, `Serialize`
