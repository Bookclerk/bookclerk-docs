# bookclerk_resolve::listing

The validated listing a storefront reports, and who holds it.

[`ListingDraft`](#listingdraft) is what a plugin said, unchecked. [`Listing::from_draft`](#listingfrom_draft)
is the boundary: it trims and normalises every field, drops blank
identifiers, credits and series, removes duplicates, and refuses records
that are empty or too large. The resolver only ever sees a [`Listing`](#listing).

| Item | Kind | Summary |
| --- | --- | --- |
| [`ListingKey`](#listingkey) | struct | What a store sells, keyed the way `store_listings` is. |
| [`Credit`](#credit) | struct | One credited contributor on a listing, with its resolved level. |
| [`SeriesEntry`](#seriesentry) | struct | A work's place in one series, as a listing reports it. |
| [`ListingDraft`](#listingdraft) | struct | An unchecked listing, field for field as a plugin reported it. |
| [`Listing`](#listing) | struct | A validated, normalised listing: the resolver's only input record. |
| [`HoldingDraft`](#holdingdraft) | struct | Ownership details a scan reports with a listing. |
| [`Holder`](#holder) | enum | Who owns a listing (`holdings.holder`). |
| [`ListingError`](#listingerror) | enum | Why a draft listing was refused. |
| [`stored_marketplace`](#stored_marketplace) | function | The stored spelling of a marketplace a source reported: trimmed, lower-cased, and `DEFAULT_MARKETPLACE` when blank. |
| [`work_match_key`](#work_match_key) | function | The work matcher's key for a title and its authors. |
| [`DEFAULT_MARKETPLACE`](#default_marketplace) | constant | The marketplace a listing that names none is sold in. |
| [`MAX_TEXT_BYTES`](#max_text_bytes) | constant | Longest text field accepted, in bytes. |
| [`MAX_IDENTIFIERS`](#max_identifiers) | constant | Most identifiers one listing may carry; keeps one listing's link batch within every database engine's statement cap. |
| [`MAX_CREDITS`](#max_credits) | constant | Most credits one listing may carry. |
| [`MAX_SERIES`](#max_series) | constant | Most series entries one listing may carry; small enough that a work's series and series entries are written in one credit batch, so they are never left part-written. |

## Structs

### ListingKey

```rust
pub struct ListingKey {
    pub plugin_key: String,
    pub marketplace: String,
    pub product_id: String,
}
```

What a store sells, keyed the way `store_listings` is.

**Fields**

- `plugin_key`: Provenance-qualified PluginKey of the storefront.
- `marketplace`: Lower-cased marketplace (`us`, `uk`, …).
- `product_id`: The store's own product id.

Implements: `Clone`, `Debug`, `Eq`, `Hash`, `Ord`, `PartialEq`, `PartialOrd`

### Credit

```rust
pub struct Credit {
    pub name: String,
    pub role: CreditRole,
    pub level: Level,
}
```

One credited contributor on a listing, with its resolved level.

**Fields**

- `name`: Display name, whitespace collapsed.
- `role`: What the contributor did.
- `level`: The level the credit attaches to.

#### Credit::name_key

```rust
pub fn name_key(&self) -> String
```

The cleaned name contributors are matched by.

Implements: `Clone`, `Debug`, `Eq`, `Hash`, `Ord`, `PartialEq`, `PartialOrd`

### SeriesEntry

```rust
pub struct SeriesEntry {
    pub name: String,
    pub position: Option<String>,
}
```

A work's place in one series, as a listing reports it.

**Fields**

- `name`: Series name, whitespace collapsed.
- `position`: Position label (`1`, `2.5`, `1-3`) when known.

#### SeriesEntry::name_key

```rust
pub fn name_key(&self) -> String
```

The cleaned name series are matched by.

#### SeriesEntry::sort_position

```rust
pub fn sort_position(&self) -> Option<f64>
```

The position as a sortable number: the leading decimal of the label.

`1-3` sorts as 1 and `Book 2` as 2; a label without digits has none.

Implements: `Clone`, `Debug`, `Eq`, `Hash`, `Ord`, `PartialEq`, `PartialOrd`

### ListingDraft

```rust
pub struct ListingDraft {
    pub plugin_key: String,
    pub marketplace: Option<String>,
    pub product_id: String,
    pub work_kind: Option<WorkKind>,
    pub expression_kind: Option<ExpressionKind>,
    pub media_kind: Option<MediaKind>,
    pub language: Option<String>,
    pub abridgement: Option<Abridgement>,
    pub title: String,
    pub subtitle: Option<String>,
    pub published_at: Option<String>,
    pub duration_minutes: Option<i64>,
    pub categories: Option<String>,
    pub identifiers: Vec<(IdentifierScheme, String, Option<String>)>,
    pub credits: Vec<(String, CreditRole, Option<Level>)>,
    pub series: Vec<(String, Option<String>)>,
    pub parent_product_id: Option<String>,
}
```

An unchecked listing, field for field as a plugin reported it.

**Fields**

- `plugin_key`: PluginKey of the storefront; the host sets it, never the guest.
- `marketplace`: Marketplace when known; blank reads as `us`.
- `product_id`: The store's own product id.
- `work_kind`: Kind of work.
- `expression_kind`: How the work is realised.
- `media_kind`: Form of the edition.
- `language`: Content language code when known.
- `abridgement`: Abridgement as reported.
- `title`: Title.
- `subtitle`: Subtitle when distinct from the title.
- `published_at`: Publication date (RFC 3339) when known.
- `duration_minutes`: Runtime in whole minutes when known.
- `categories`: Category labels as one string when known.
- `identifiers`: Identifiers: scheme, raw value, and the marketplace the source gave.
- `credits`: Credits in billing order: name, role, and a level when the source
  departs from the role's usual one.
- `series`: Series entries: name and position label.
- `parent_product_id`: Parent show's product id, for a podcast episode.

Implements: `Clone`, `Debug`, `Default`, `PartialEq`

### Listing

```rust
pub struct Listing {
    pub key: ListingKey,
    pub work_kind: WorkKind,
    pub expression_kind: ExpressionKind,
    pub media_kind: MediaKind,
    pub language: Option<String>,
    pub abridgement: Abridgement,
    pub title: String,
    pub subtitle: Option<String>,
    pub published_at: Option<String>,
    pub duration_minutes: Option<i64>,
    pub categories: Option<String>,
    pub identifiers: Vec<IdentifierKey>,
    pub credits: Vec<Credit>,
    pub series: Vec<SeriesEntry>,
    pub parent_product_id: Option<String>,
}
```

A validated, normalised listing: the resolver's only input record.

**Fields**

- `key`: The store, marketplace and product it is.
- `work_kind`: Kind of work.
- `expression_kind`: How the work is realised.
- `media_kind`: Form of the edition.
- `language`: Lower-cased language code; `None` when unknown.
- `abridgement`: Abridgement; [`Abridgement::Unknown`](kinds.md#abridgement) when the source did not say.
- `title`: Title, whitespace collapsed.
- `subtitle`: Subtitle, whitespace collapsed.
- `published_at`: Publication date (RFC 3339) when known.
- `duration_minutes`: Runtime in whole minutes when known.
- `categories`: Category labels when known.
- `identifiers`: Normalised identifiers, duplicates removed, in reported order.
- `credits`: Credits in billing order, duplicates removed.
- `series`: Series entries, duplicates removed.
- `parent_product_id`: The parent show's product id, for a podcast episode only.

#### Listing::from_draft

```rust
pub fn from_draft(draft: &ListingDraft) -> Result<Self, ListingError>
```

Validates and normalises a draft.

##### Arguments

* `draft` - The listing as the plugin reported it.

##### Returns

The normalised listing. Absent kinds take their common value
(`book`, `narration`, `audiobook`); an episode keeps its parent id,
and any other kind drops it.

##### Errors

Returns [`ListingError`](#listingerror) when the plugin key, product id or title is
blank, a text field is too long, or a list is over its limit.

#### Listing::parent_key

```rust
pub fn parent_key(&self) -> Option<ListingKey>
```

The listing of this one's parent show, for an episode with a parent id.

#### Listing::credits_at

```rust
pub fn credits_at(&self, level: Level) -> impl Iterator<Item = &Credit>
```

Credits attached at `level`, in billing order.

#### Listing::edition_identifiers

```rust
pub fn edition_identifiers(&self) -> impl Iterator<Item = &IdentifierKey>
```

Identifiers whose scheme names an edition, in reported order.

#### Listing::work_identifiers

```rust
pub fn work_identifiers(&self) -> impl Iterator<Item = &IdentifierKey>
```

Identifiers whose scheme names a work, in reported order.

#### Listing::work_match_key

```rust
pub fn work_match_key(&self) -> String
```

The work matcher's key: cleaned title and sorted author names.

Empty when the cleaned title is empty or no author is credited at
the work level, so such a listing is never matched by text alone.

#### Listing::expression_credit_key

```rust
pub fn expression_credit_key(&self) -> String
```

The expression's credit key: its credits as sorted, cleaned
`role:name` pairs, so two realisations whose people swap roles
(narrator and translator, say) never compare equal.

Implements: `Clone`, `Debug`, `PartialEq`

### HoldingDraft

```rust
pub struct HoldingDraft {
    pub holder: Holder,
    pub entitlement: Entitlement,
    pub purchased_at: Option<String>,
}
```

Ownership details a scan reports with a listing.

**Fields**

- `holder`: Who holds the listing.
- `entitlement`: How they hold it.
- `purchased_at`: When they bought or added it (RFC 3339), when known.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

## Enums

### Holder

```rust
pub enum Holder {
    Account(String),
    Node(String),
}
```

Who owns a listing (`holdings.holder`).

**Variants**

- `Account`: A store account of the listing's plugin, by account id.
- `Node`: A branch library, by node id (`node:<node_id>` provenance).

#### Holder::stored

```rust
pub fn stored(&self) -> String
```

The stored text: `account:<id>` or `node:<id>`.

#### Holder::parse

```rust
pub fn parse(text: &str) -> Option<Self>
```

Parses stored holder text; `None` for any other shape.

Implements: `Clone`, `Debug`, `Eq`, `Hash`, `Ord`, `PartialEq`, `PartialOrd`

### ListingError

```rust
pub enum ListingError {
    Blank(&'static str),
    TooLong(&'static str),
    TooMany(&'static str),
    NegativeDuration,
}
```

Why a draft listing was refused.

**Variants**

- `Blank`: A required field (plugin key, product id or title) is blank.
- `TooLong`: A text field is longer than [`MAX_TEXT_BYTES`](#max_text_bytes).
- `TooMany`: A list holds more entries than its limit.
- `NegativeDuration`: The runtime is negative.

Implements: `Clone`, `Debug`, `Display`, `Eq`, `Error`, `PartialEq`

## Functions

### stored_marketplace

```rust
pub fn stored_marketplace(raw: &str) -> String
```

The stored spelling of a marketplace a source reported: trimmed,
lower-cased, and [`DEFAULT_MARKETPLACE`](#default_marketplace) when blank.

### work_match_key

```rust
pub fn work_match_key<'a>(title: &str, authors: impl Iterator<Item = &'a str>) -> String
```

The work matcher's key for a title and its authors.

#### Arguments

* `title` - Work title (decoded, any case).
* `authors` - Author display names.

#### Returns

`clean_title` and the sorted, de-duplicated `clean_author` names joined
with unit separators; empty when either side is empty.

## Constants

### DEFAULT_MARKETPLACE

```rust
pub const DEFAULT_MARKETPLACE: &str = "us";
```

The marketplace a listing that names none is sold in.

### MAX_TEXT_BYTES

```rust
pub const MAX_TEXT_BYTES: usize = 4096;
```

Longest text field accepted, in bytes.

### MAX_IDENTIFIERS

```rust
pub const MAX_IDENTIFIERS: usize = 12;
```

Most identifiers one listing may carry; keeps one listing's link
batch within every database engine's statement cap.

### MAX_CREDITS

```rust
pub const MAX_CREDITS: usize = 128;
```

Most credits one listing may carry.

### MAX_SERIES

```rust
pub const MAX_SERIES: usize = 12;
```

Most series entries one listing may carry; small enough that a work's
series and series entries are written in one credit batch, so they are
never left part-written.
