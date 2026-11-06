# bookclerk_resolve::model

Stored rows the resolver reads, the snapshot of them it is given, and
the plan of writes it returns.

| Item | Kind | Summary |
| --- | --- | --- |
| [`WorkRow`](#workrow) | struct | A stored work (`works`). |
| [`ExpressionRow`](#expressionrow) | struct | A stored expression (`expressions`). |
| [`EditionRow`](#editionrow) | struct | A stored edition (`editions`). |
| [`EditionIdentifierRow`](#editionidentifierrow) | struct | An identifier attached to an edition (`edition_identifiers`). |
| [`WorkIdentifierRow`](#workidentifierrow) | struct | An identifier attached to a work (`work_identifiers`). |
| [`ListingRow`](#listingrow) | struct | A stored store listing (`store_listings`). |
| [`HoldingRow`](#holdingrow) | struct | A stored holding (`holdings`). |
| [`Snapshot`](#snapshot) | struct | The stored rows that bear on one listing, read before resolving it. |
| [`CreditWrite`](#creditwrite) | struct | A credit to write on a new work, expression or edition. |
| [`WorkParent`](#workparent) | struct | A show link for a stored episode work that was created before its show was known. |
| [`WorkMatchKey`](#workmatchkey) | struct | A matcher key for a stored work created before its listing named an author, so the work had none. |
| [`SeriesWrite`](#serieswrite) | struct | A series entry to write on a new work. |
| [`Plan`](#plan) | struct | Every write one listing's link needs, applied as one atomic batch. |
| [`Override`](#override) | enum | An Administrator's decision the resolver obeys (`resolution_overrides`). |
| [`LinkReason`](#linkreason) | enum | Why a listing reached the edition it did. |
| [`WorkReason`](#workreason) | enum | How a new edition found its work. |
| [`Conflict`](#conflict) | enum | Something the resolver could not decide alone; recorded for an Administrator (`resolution_conflicts`). |
| [`Event`](#event) | enum | A `core.events` outbox event the link publishes. |

## Structs

### WorkRow

```rust
pub struct WorkRow {
    pub id: String,
    pub kind: WorkKind,
    pub title: String,
    pub match_key: String,
    pub parent_work_id: Option<String>,
}
```

A stored work (`works`).

**Fields**

- `id`: Work id.
- `kind`: Kind of work.
- `title`: Display title.
- `match_key`: The work matcher's key ([`crate::listing::work_match_key`](listing.md#work_match_key)); empty
  when the work is never matched by text.
- `parent_work_id`: The show a podcast episode belongs to.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### ExpressionRow

```rust
pub struct ExpressionRow {
    pub id: String,
    pub work_id: String,
    pub kind: ExpressionKind,
    pub language: Option<String>,
    pub abridgement: Abridgement,
    pub credit_key: String,
}
```

A stored expression (`expressions`).

**Fields**

- `id`: Expression id.
- `work_id`: The work it realises.
- `kind`: How it realises the work.
- `language`: Lower-cased language code; `None` when unknown.
- `abridgement`: Abridgement.
- `credit_key`: Sorted, cleaned names of its narrator, translator, host and cast
  credits ([`crate::listing::Listing::expression_credit_key`](listing.md#listingexpression_credit_key)).

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### EditionRow

```rust
pub struct EditionRow {
    pub id: String,
    pub expression_id: String,
    pub media_kind: MediaKind,
    pub title: String,
    pub subtitle: Option<String>,
    pub published_at: Option<String>,
    pub duration_minutes: Option<i64>,
}
```

A stored edition (`editions`).

**Fields**

- `id`: Edition id.
- `expression_id`: The expression it releases.
- `media_kind`: Its form.
- `title`: Display title.
- `subtitle`: Subtitle.
- `published_at`: Publication date (RFC 3339).
- `duration_minutes`: Runtime in whole minutes.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### EditionIdentifierRow

```rust
pub struct EditionIdentifierRow {
    pub edition_id: String,
    pub key: IdentifierKey,
}
```

An identifier attached to an edition (`edition_identifiers`).

**Fields**

- `edition_id`: The edition.
- `key`: The identifier.

Implements: `Clone`, `Debug`, `Eq`, `Ord`, `PartialEq`, `PartialOrd`

### WorkIdentifierRow

```rust
pub struct WorkIdentifierRow {
    pub work_id: String,
    pub key: IdentifierKey,
}
```

An identifier attached to a work (`work_identifiers`).

**Fields**

- `work_id`: The work.
- `key`: The identifier.

Implements: `Clone`, `Debug`, `Eq`, `Ord`, `PartialEq`, `PartialOrd`

### ListingRow

```rust
pub struct ListingRow {
    pub id: String,
    pub key: ListingKey,
    pub edition_id: String,
}
```

A stored store listing (`store_listings`).

**Fields**

- `id`: Listing id.
- `key`: Store, marketplace and product.
- `edition_id`: The edition it sells.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### HoldingRow

```rust
pub struct HoldingRow {
    pub listing_id: String,
    pub holder: Holder,
    pub entitlement: Entitlement,
    pub purchased_at: Option<String>,
}
```

A stored holding (`holdings`).

**Fields**

- `listing_id`: The listing held.
- `holder`: Who holds it.
- `entitlement`: How they hold it.
- `purchased_at`: When they bought or added it (RFC 3339).

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### Snapshot

```rust
pub struct Snapshot {
    pub listing: Option<ListingRow>,
    pub parent_listing: Option<ListingRow>,
    pub edition_identifiers: Vec<EditionIdentifierRow>,
    pub work_identifiers: Vec<WorkIdentifierRow>,
    pub editions: Vec<EditionRow>,
    pub expressions: Vec<ExpressionRow>,
    pub works: Vec<WorkRow>,
    pub overrides: Vec<Override>,
    pub holding: Option<HoldingRow>,
    pub credited: Vec<(Level, String)>,
    pub works_with_series: Vec<String>,
}
```

The stored rows that bear on one listing, read before resolving it.

The caller fills it with:

- the listing's own row and its parent show's;
- every edition identifier row sharing one of the listing's edition
  identifiers, and every identifier row of each edition included;
- every work identifier row sharing one of the listing's work identifiers;
- the editions those rows, the two listings and any pin reach, with
  their expressions and works;
- every work with the listing's match key and work kind;
- every expression of each included work;
- the pins on the listing and the isolations of included works;
- the holder's holding of the listing;
- which included works, expressions and editions already have credits,
  and which included works already have series entries.

[`Snapshot::is_closed`](#snapshotis_closed) checks the references are complete.

**Fields**

- `listing`: The listing's stored row, when it was seen before.
- `parent_listing`: The parent show's stored listing, for an episode.
- `edition_identifiers`: Stored rows sharing one of the listing's edition identifiers.
- `work_identifiers`: Stored rows sharing one of the listing's work identifiers.
- `editions`: Editions the rows above reach.
- `expressions`: Expressions of those editions and of every included work.
- `works`: Works of those expressions, identifier rows and match candidates.
- `overrides`: Overrides on the listing or on any included work.
- `holding`: The holder's stored holding of the listing.
- `credited`: Included works, expressions and editions whose credits are complete.
  
  A store that writes credits in several batches must commit each
  target's first-billed credit (position 0) last and count only
  targets that have it, so a partial write is finished on the next
  scan rather than taken as complete.
- `works_with_series`: Included works that have at least one series entry.

#### Snapshot::edition

```rust
pub fn edition(&self, id: &str) -> Option<&EditionRow>
```

The edition with `id`, when included.

#### Snapshot::expression

```rust
pub fn expression(&self, id: &str) -> Option<&ExpressionRow>
```

The expression with `id`, when included.

#### Snapshot::work

```rust
pub fn work(&self, id: &str) -> Option<&WorkRow>
```

The work with `id`, when included.

#### Snapshot::work_of_edition

```rust
pub fn work_of_edition(&self, edition_id: &str) -> Option<&WorkRow>
```

The work an included edition realises, through its expression.

#### Snapshot::is_closed

```rust
pub fn is_closed(&self) -> Option<String>
```

Checks that every edition and work the snapshot refers to is
included, with each edition's expression and work. A pin whose
edition is gone is ignored, as [`crate::resolve()`](README.md#resolve) ignores it.

##### Returns

The first missing reference, described, or `None` when closed.

Implements: `Clone`, `Debug`, `Default`, `PartialEq`

### CreditWrite

```rust
pub struct CreditWrite {
    pub level: Level,
    pub target_id: String,
    pub credit: Credit,
    pub position: u32,
}
```

A credit to write on a new work, expression or edition.

**Fields**

- `level`: The level of the target.
- `target_id`: The target's id.
- `credit`: The credit.
- `position`: Billing position, from 0.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### WorkParent

```rust
pub struct WorkParent {
    pub work_id: String,
    pub parent_work_id: String,
}
```

A show link for a stored episode work that was created before its
show was known.

**Fields**

- `work_id`: The episode work.
- `parent_work_id`: The show work it belongs to.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### WorkMatchKey

```rust
pub struct WorkMatchKey {
    pub work_id: String,
    pub kind: WorkKind,
    pub parent_work_id: Option<String>,
    pub match_key: String,
}
```

A matcher key for a stored work created before its listing named an
author, so the work had none.

**Fields**

- `work_id`: The work, whose stored key is empty.
- `kind`: Its kind; only works of the same kind compete for a key.
- `parent_work_id`: Its show, for an episode; episodes only compete within one show.
- `match_key`: The key it gains ([`crate::listing::work_match_key`](listing.md#work_match_key)).

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### SeriesWrite

```rust
pub struct SeriesWrite {
    pub work_id: String,
    pub entry: SeriesEntry,
}
```

A series entry to write on a new work.

**Fields**

- `work_id`: The work.
- `entry`: The entry.

Implements: `Clone`, `Debug`, `PartialEq`

### Plan

```rust
pub struct Plan {
    pub listing: ListingRow,
    pub listing_is_new: bool,
    pub link: LinkReason,
    pub work_reason: Option<WorkReason>,
    pub new_work: Option<WorkRow>,
    pub new_expression: Option<ExpressionRow>,
    pub new_edition: Option<EditionRow>,
    pub edition_update: Option<EditionRow>,
    pub expression_update: Option<ExpressionRow>,
    pub work_parent: Option<WorkParent>,
    pub work_match_key: Option<WorkMatchKey>,
    pub edition_identifiers: Vec<EditionIdentifierRow>,
    pub work_identifiers: Vec<WorkIdentifierRow>,
    pub credits: Vec<CreditWrite>,
    pub series: Vec<SeriesWrite>,
    pub holding: HoldingRow,
    pub holding_is_new: bool,
    pub holding_changed: bool,
    pub conflicts: Vec<Conflict>,
    pub events: Vec<Event>,
}
```

Every write one listing's link needs, applied as one atomic batch.

**Fields**

- `listing`: The listing row to insert or refresh.
- `listing_is_new`: True when the listing row is new.
- `link`: Why the listing reached its edition.
- `work_reason`: How a new edition found its work; `None` for an existing edition.
- `new_work`: A new work to insert.
- `new_expression`: A new expression to insert.
- `new_edition`: A new edition to insert.
- `edition_update`: New metadata for an existing edition a rescan changed.
- `expression_update`: The stored expression of an existing edition with the language,
  abridgement or performer key it lacked filled from the listing.
- `work_parent`: The show link a stored, show-less episode work gains.
- `work_match_key`: The matcher key a stored work without one gains.
- `edition_identifiers`: Edition identifiers to attach.
- `work_identifiers`: Work identifiers to attach.
- `credits`: Credits for the work, expression and edition the listing reached,
  for each of them that has none yet.
- `series`: Series entries for the work, when it has none yet.
- `holding`: The holding to insert or refresh.
- `holding_is_new`: True when the holding is new.
- `holding_changed`: True when a stored holding's entitlement or purchase date changes.
- `conflicts`: Decisions left to an Administrator.
- `events`: Events to publish.

#### Plan::is_noop

```rust
pub fn is_noop(&self) -> bool
```

True when applying the plan would change nothing but timestamps:
no new rows, no metadata or holding change, no conflict and no
event.

Implements: `Clone`, `Debug`, `PartialEq`

## Enums

### Override

```rust
pub enum Override {
    PinListing { listing: ListingKey, edition_id: String },
    IsolateWork { work_id: String },
}
```

An Administrator's decision the resolver obeys (`resolution_overrides`).

**Variants**

- `PinListing`: This listing sells that edition, whatever its identifiers say.
  - `listing`: The listing.
  - `edition_id`: The edition it sells.
- `IsolateWork`: The work matcher never joins a new listing to this work (a split).
  - `work_id`: The work.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### LinkReason

```rust
pub enum LinkReason {
    Pinned,
    Listing,
    StrongIdentifier,
    WeakIdentifier,
    New,
}
```

Why a listing reached the edition it did.

**Variants**

- `Pinned`: An Administrator pinned the listing to the edition.
- `Listing`: The listing was stored before and keeps its edition.
- `StrongIdentifier`: A strong identifier named the edition.
- `WeakIdentifier`: A weak identifier proposed the edition and its expression and
  media kind were compatible.
- `New`: No stored edition matched; a new one is created.

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `PartialEq`

### WorkReason

```rust
pub enum WorkReason {
    Identifier,
    Matcher,
    New,
}
```

How a new edition found its work.

**Variants**

- `Identifier`: A shared work identifier named the work.
- `Matcher`: The work matcher found exactly one work.
- `New`: No work matched; a new one is created.

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `PartialEq`

### Conflict

```rust
pub enum Conflict {
    StrongIdentifiers { edition_ids: Vec<String> },
    WorkIdentifiers { work_ids: Vec<String> },
    AmbiguousWork { work_ids: Vec<String> },
    WorkKindMismatch { work_id: String },
}
```

Something the resolver could not decide alone; recorded for an
Administrator (`resolution_conflicts`).

**Variants**

- `StrongIdentifiers`: The listing's strong identifiers name more than one edition, or an
  edition other than the one it reached. None of them were merged.
  - `edition_ids`: The editions the identifiers name, the reached one first.
- `WorkIdentifiers`: The listing's work identifiers name more than one work, or a work
  other than its edition's.
  - `work_ids`: The works the identifiers name, the chosen one first.
- `AmbiguousWork`: More than one stored work matches the listing's title and authors,
  so it joined none of them.
  - `work_ids`: The matching works.
- `WorkKindMismatch`: A work identifier names a work of a different kind; it was not joined.
  - `work_id`: The work the identifier names.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### Event

```rust
pub enum Event {
    EditionLinked { edition_id: String, listing_id: String },
    EditionUpdated { edition_id: String },
    HoldingAdded { listing_id: String, holder: Holder },
}
```

A `core.events` outbox event the link publishes.

**Variants**

- `EditionLinked`: A listing joined an edition, new or existing (`edition_linked`).
  - `edition_id`: The edition.
  - `listing_id`: The listing.
- `EditionUpdated`: A rescan changed an existing edition's metadata (`edition_updated`).
  - `edition_id`: The edition.
- `HoldingAdded`: A holder gained a listing (`holding_added`).
  - `listing_id`: The listing.
  - `holder`: The holder.

#### Event::event_type

```rust
pub fn event_type(&self) -> &'static str
```

The event type written to the outbox.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`
