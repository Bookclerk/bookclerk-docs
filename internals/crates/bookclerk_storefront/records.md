# bookclerk_storefront::records

Builders and flat views for the structured bibliographic records that
`ScanBook` and `CatalogHit` carry on the plugin ABI.

## Audience

Storefront guests that turn store payloads into credits, identifiers and
series entries, and host code that still reads the flat `books` row view
until the library schema stores works, expressions and editions
(`docs/adr/library-data-model.md`).

| Item | Kind | Summary |
| --- | --- | --- |
| [`credit`](#credit) | function | One credit, or `None` when `name` is blank after trimming. |
| [`credits_from_names`](#credits_from_names) | function | Credits for a list of names that a store already returns one per entry, in billing order, skipping blank names. |
| [`credits_from_display`](#credits_from_display) | function | Credits for a store's display string that lists several names in one field (`"Jane Doe, John Roe & Ann Poe"`). |
| [`split_display_names`](#split_display_names) | function | Splits a display string of contributor names into one name per entry. |
| [`identifier`](#identifier) | function | One identifier, or `None` when `value` is blank after trimming. |
| [`series_entries`](#series_entries) | function | A single-entry series list, or empty when `name` is missing or blank. |
| [`joined_names`](#joined_names) | function | Names credited with `role`, joined for the flat `books` row view. |
| [`identifier_value`](#identifier_value) | function | First non-blank identifier value in `scheme`, normalised, for the flat `books` row view. |
| [`asin_for_marketplace`](#asin_for_marketplace) | function | The first ASIN scoped to `marketplace`, else the first unscoped ASIN, else the first ASIN of any marketplace, normalised. |
| [`normalize_identifier`](#normalize_identifier) | function | The canonical form of an identifier value as a source printed it. |
| [`content_kind`](#content_kind) | function | The flat `books.content_kind` value for a work kind. |
| [`work_kind_from_label`](#work_kind_from_label) | function | The work kind for a store's own item-kind label (`episode`, `podcast`). |
| [`audio_kinds`](#audio_kinds) | function | The expression and media kinds an audio storefront's item has, given its work kind: a podcast is original audio, a book a narration, and both come as downloadable audio. |
| [`fill_credits`](#fill_credits) | function | Adds the credits of `fill` whose role `base` has none of, so a merge of two hits for one edition never overwrites the credits already known. |
| [`fill_identifiers`](#fill_identifiers) | function | Adds the identifiers of `fill` whose scheme `base` has none of. |
| [`NAME_SEPARATOR`](#name_separator) | constant | Separator the flat `books` row view uses between contributor names. |

## Re-exports

- [`abi::Credit`](../bookclerk_plugin_abi/generated.md#credit)
- [`abi::CreditLevel`](../bookclerk_plugin_abi/generated.md#creditlevel)
- [`abi::CreditRole`](../bookclerk_plugin_abi/generated.md#creditrole)
- [`abi::ExpressionKind`](../bookclerk_plugin_abi/generated.md#expressionkind)
- [`abi::Identifier`](../bookclerk_plugin_abi/generated.md#identifier)
- [`abi::IdentifierScheme`](../bookclerk_plugin_abi/generated.md#identifierscheme)
- [`abi::MediaKind`](../bookclerk_plugin_abi/generated.md#mediakind)
- [`abi::SeriesEntry`](../bookclerk_plugin_abi/generated.md#seriesentry)
- [`abi::WorkKind`](../bookclerk_plugin_abi/generated.md#workkind)

## Functions

### credit

```rust
pub fn credit(name: &str, role: CreditRole) -> Option<Credit>
```

One credit, or `None` when `name` is blank after trimming.

#### Arguments

* `name` - contributor display name, one person or body.
* `role` - what the contributor did; the level is the role's usual one.

### credits_from_names

```rust
pub fn credits_from_names<I, S>(names: I, role: CreditRole) -> Vec<Credit>
where
    I: IntoIterator<Item = S>,
    S: AsRef<str>,
```

Credits for a list of names that a store already returns one per entry,
in billing order, skipping blank names.

### credits_from_display

```rust
pub fn credits_from_display(display: Option<&str>, role: CreditRole) -> Vec<Credit>
```

Credits for a store's display string that lists several names in one
field (`"Jane Doe, John Roe & Ann Poe"`).

Splits on `,`, `;`, `&` and the word `and`, and keeps a name suffix such
as `Jr.` with the name before it. Prefer [`credits_from_names`](#credits_from_names) when the
store returns a list.

### split_display_names

```rust
pub fn split_display_names(display: &str) -> Vec<String>
```

Splits a display string of contributor names into one name per entry.

#### Returns

The trimmed, non-empty names in the order they appear. HTML entities
are decoded first, so `O&#39;Brien` stays one name; splitting is
[`bookclerk_resolve::names::split_display_names`](../bookclerk_resolve/names.md#split_display_names).

### identifier

```rust
pub fn identifier(
    scheme: IdentifierScheme,
    value: &str,
    marketplace: Option<&str>,
) -> Option<Identifier>
```

One identifier, or `None` when `value` is blank after trimming.

#### Arguments

* `scheme` - scheme the value belongs to.
* `value` - the identifier as the store printed it.
* `marketplace` - marketplace an ASIN is scoped to; ignored when blank.

### series_entries

```rust
pub fn series_entries(name: Option<&str>, position: Option<&str>) -> Vec<SeriesEntry>
```

A single-entry series list, or empty when `name` is missing or blank.

### joined_names

```rust
pub fn joined_names(credits: &[Credit], role: CreditRole) -> Option<String>
```

Names credited with `role`, joined for the flat `books` row view.

#### Returns

`None` when no credit has the role.

### identifier_value

```rust
pub fn identifier_value(
    identifiers: &[Identifier],
    scheme: IdentifierScheme,
) -> Option<String>
```

First non-blank identifier value in `scheme`, normalised, for the flat
`books` row view.

Values are normalised with [`normalize_identifier`](#normalize_identifier), so a guest that
sends a padded, hyphenated ISBN-10 still fills the column with the
ISBN-13 the library keys on.

### asin_for_marketplace

```rust
pub fn asin_for_marketplace(
    identifiers: &[Identifier],
    marketplace: &str,
) -> Option<String>
```

The first ASIN scoped to `marketplace`, else the first unscoped ASIN,
else the first ASIN of any marketplace, normalised.

Marketplaces compare without regard to case or surrounding space.

### normalize_identifier

```rust
pub fn normalize_identifier(scheme: IdentifierScheme, raw: &str) -> Option<String>
```

The canonical form of an identifier value as a source printed it.

Every value is trimmed. ASINs are upper-cased. ISBNs lose spaces and
hyphens, and a valid ISBN-10 becomes its ISBN-13; an ISBN of any other
shape is kept as the stripped text, since rejecting it would lose data.

#### Returns

`None` when the value is blank.

### content_kind

```rust
pub fn content_kind(kind: WorkKind, has_parent: bool) -> Option<&'static str>
```

The flat `books.content_kind` value for a work kind.

A podcast that names its show (`parent_product_id`) is an `episode`; a
podcast without one is the show (`podcast`).

#### Arguments

* `kind` - The record's work kind.
* `has_parent` - Whether the record names a parent show.

#### Returns

`None` for `WorkKind::Unspecified`, which the host refuses.

### work_kind_from_label

```rust
pub fn work_kind_from_label(label: &str) -> WorkKind
```

The work kind for a store's own item-kind label (`episode`, `podcast`).

Labels a storefront uses for podcast episodes and shows (`episode`,
`podcast`, `parent`, `podcastparent`, `season`) read as
`WorkKind::Podcast`; anything else is a book. Whether a podcast is the
show or an episode is then carried by `parent_product_id`.

### audio_kinds

```rust
pub fn audio_kinds(work: WorkKind) -> (ExpressionKind, MediaKind)
```

The expression and media kinds an audio storefront's item has, given its
work kind: a podcast is original audio, a book a narration, and both
come as downloadable audio.

#### Returns

`ExpressionKind::Unspecified` and `MediaKind::Unspecified` for an
unspecified work kind, so the host refuses the record.

### fill_credits

```rust
pub fn fill_credits(base: &mut Vec<Credit>, fill: Vec<Credit>)
```

Adds the credits of `fill` whose role `base` has none of, so a merge of
two hits for one edition never overwrites the credits already known.

### fill_identifiers

```rust
pub fn fill_identifiers(base: &mut Vec<Identifier>, fill: Vec<Identifier>)
```

Adds the identifiers of `fill` whose scheme `base` has none of.

## Constants

### NAME_SEPARATOR

```rust
pub const NAME_SEPARATOR: &str = bookclerk_resolve::names::NAME_SEPARATOR;
```

Separator the flat `books` row view uses between contributor names.
