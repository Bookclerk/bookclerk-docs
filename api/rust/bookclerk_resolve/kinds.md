# bookclerk_resolve::kinds

Closed vocabularies of the library data model.

Each enum is matched exhaustively in Rust, carried as a capnp enum on the
plugin wire, and stored as text under a `CHECK` constraint in SQL. The
[`as_str`](#workkindas_str) spellings are those stored values; a new
variant is appended to all three at once.

| Item | Kind | Summary |
| --- | --- | --- |
| [`WorkKind`](#workkind) | enum | What kind of creative work a work is (`works.kind`). |
| [`ExpressionKind`](#expressionkind) | enum | How a work is realised (`expressions.kind`). |
| [`MediaKind`](#mediakind) | enum | Physical or digital form of an edition (`editions.media_kind`). |
| [`Abridgement`](#abridgement) | enum | Whether an expression is abridged (`expressions.abridgement`). |
| [`IdentifierScheme`](#identifierscheme) | enum | Scheme of an external identifier; it fixes level and strength. |
| [`Strength`](#strength) | enum | Whether an edition identifier names exactly one edition. |
| [`Level`](#level) | enum | The bibliographic level a credit or identifier attaches to. |
| [`CreditRole`](#creditrole) | enum | What a contributor did (`credits.role`). |
| [`Entitlement`](#entitlement) | enum | How a holder came to own a listing (`holdings.entitlement`). |
| [`sql_check_list`](#sql_check_list) | function | SQL `CHECK` list of an enum's stored values: `'a', 'b', 'c'`. |

## Enums

### WorkKind

```rust
pub enum WorkKind {
    Book,
    PodcastShow,
    PodcastEpisode,
    Drama,
}
```

What kind of creative work a work is (`works.kind`).

**Variants**

- `Book`: A book, fiction or non-fiction.
- `PodcastShow`: A podcast show; its episodes are child works.
- `PodcastEpisode`: One podcast episode, a child work of its show.
- `Drama`: An original audio drama that adapts no book.

#### WorkKind::ALL

```rust
pub const ALL: &'static [Self] = _
```

Every variant, in declaration order.

#### WorkKind::as_str

```rust
pub const fn as_str(self) -> &'static str
```

The text stored for this value in SQL.

#### WorkKind::parse

```rust
pub fn parse(text: &str) -> Option<Self>
```

Parses a stored value; `None` for text that names no variant.

Implements: `Clone`, `Copy`, `Debug`, `Display`, `Eq`, `Hash`, `Ord`, `PartialEq`, `PartialOrd`

### ExpressionKind

```rust
pub enum ExpressionKind {
    Narration,
    Text,
    Translation,
    Dramatisation,
    PodcastAudio,
}
```

How a work is realised (`expressions.kind`).

**Variants**

- `Narration`: A reading of the text by one or more narrators.
- `Text`: The text itself, as sold in print and ebook editions.
- `Translation`: A translation of the text into another language.
- `Dramatisation`: A full-cast dramatisation or adaptation.
- `PodcastAudio`: The recorded audio of a podcast episode.

#### ExpressionKind::ALL

```rust
pub const ALL: &'static [Self] = _
```

Every variant, in declaration order.

#### ExpressionKind::as_str

```rust
pub const fn as_str(self) -> &'static str
```

The text stored for this value in SQL.

#### ExpressionKind::parse

```rust
pub fn parse(text: &str) -> Option<Self>
```

Parses a stored value; `None` for text that names no variant.

Implements: `Clone`, `Copy`, `Debug`, `Display`, `Eq`, `Hash`, `Ord`, `PartialEq`, `PartialOrd`

### MediaKind

```rust
pub enum MediaKind {
    Audiobook,
    Ebook,
    Print,
    PodcastAudio,
}
```

Physical or digital form of an edition (`editions.media_kind`).

**Variants**

- `Audiobook`: A downloadable audiobook.
- `Ebook`: An ebook; never acquired.
- `Print`: A printed book; never acquired.
- `PodcastAudio`: Podcast episode audio.

#### MediaKind::ALL

```rust
pub const ALL: &'static [Self] = _
```

Every variant, in declaration order.

#### MediaKind::as_str

```rust
pub const fn as_str(self) -> &'static str
```

The text stored for this value in SQL.

#### MediaKind::parse

```rust
pub fn parse(text: &str) -> Option<Self>
```

Parses a stored value; `None` for text that names no variant.

#### MediaKind::acquirable

```rust
pub const fn acquirable(self) -> bool
```

Whether editions of this form are ever acquired.

Implements: `Clone`, `Copy`, `Debug`, `Display`, `Eq`, `Hash`, `Ord`, `PartialEq`, `PartialOrd`

### Abridgement

```rust
pub enum Abridgement {
    Unknown,
    Unabridged,
    Abridged,
}
```

Whether an expression is abridged (`expressions.abridgement`).

**Variants**

- `Unknown`: The source did not say. Never matches any value, itself included.
- `Unabridged`: The full text.
- `Abridged`: A shortened text.

#### Abridgement::ALL

```rust
pub const ALL: &'static [Self] = _
```

Every variant, in declaration order.

#### Abridgement::as_str

```rust
pub const fn as_str(self) -> &'static str
```

The text stored for this value in SQL.

#### Abridgement::parse

```rust
pub fn parse(text: &str) -> Option<Self>
```

Parses a stored value; `None` for text that names no variant.

Implements: `Clone`, `Copy`, `Debug`, `Display`, `Eq`, `Hash`, `Ord`, `PartialEq`, `PartialOrd`

### IdentifierScheme

```rust
pub enum IdentifierScheme {
    Isbn,
    Asin,
    OpenLibraryEdition,
    OpenLibraryWork,
}
```

Scheme of an external identifier; it fixes level and strength.

**Variants**

- `Isbn`: ISBN, normalised to ISBN-13; a weak edition identifier.
- `Asin`: Amazon ASIN in one marketplace; a strong edition identifier.
- `OpenLibraryEdition`: OpenLibrary edition id (`OL…M`); a strong edition identifier.
- `OpenLibraryWork`: OpenLibrary work id (`OL…W`); names a work.

#### IdentifierScheme::ALL

```rust
pub const ALL: &'static [Self] = _
```

Every variant, in declaration order.

#### IdentifierScheme::as_str

```rust
pub const fn as_str(self) -> &'static str
```

The text stored for this value in SQL.

#### IdentifierScheme::parse

```rust
pub fn parse(text: &str) -> Option<Self>
```

Parses a stored value; `None` for text that names no variant.

#### IdentifierScheme::level

```rust
pub const fn level(self) -> Level
```

The level whose identifier table this scheme lands in.

#### IdentifierScheme::strength

```rust
pub const fn strength(self) -> Strength
```

Whether an edition identifier of this scheme is strong or weak.

Work identifiers report [`Strength::Strong`]: one names one work.

#### IdentifierScheme::marketplace_scoped

```rust
pub const fn marketplace_scoped(self) -> bool
```

Whether a value of this scheme is scoped to a marketplace.

Implements: `Clone`, `Copy`, `Debug`, `Display`, `Eq`, `Hash`, `Ord`, `PartialEq`, `PartialOrd`

### Strength

```rust
pub enum Strength {
    Strong,
    Weak,
}
```

Whether an edition identifier names exactly one edition.

**Variants**

- `Strong`: Names exactly one edition; unique across the library.
- `Weak`: Shared in practice; only proposes a candidate edition.

#### Strength::ALL

```rust
pub const ALL: &'static [Self] = _
```

Every variant, in declaration order.

#### Strength::as_str

```rust
pub const fn as_str(self) -> &'static str
```

The text stored for this value in SQL.

#### Strength::parse

```rust
pub fn parse(text: &str) -> Option<Self>
```

Parses a stored value; `None` for text that names no variant.

Implements: `Clone`, `Copy`, `Debug`, `Display`, `Eq`, `Hash`, `Ord`, `PartialEq`, `PartialOrd`

### Level

```rust
pub enum Level {
    Work,
    Expression,
    Edition,
}
```

The bibliographic level a credit or identifier attaches to.

**Variants**

- `Work`: The work.
- `Expression`: The expression.
- `Edition`: The edition.

#### Level::ALL

```rust
pub const ALL: &'static [Self] = _
```

Every variant, in declaration order.

#### Level::as_str

```rust
pub const fn as_str(self) -> &'static str
```

The text stored for this value in SQL.

#### Level::parse

```rust
pub fn parse(text: &str) -> Option<Self>
```

Parses a stored value; `None` for text that names no variant.

Implements: `Clone`, `Copy`, `Debug`, `Display`, `Eq`, `Hash`, `Ord`, `PartialEq`, `PartialOrd`

### CreditRole

```rust
pub enum CreditRole {
    Author,
    Narrator,
    Translator,
    Editor,
    Illustrator,
    Publisher,
    Host,
    Cast,
    Adaptor,
}
```

What a contributor did (`credits.role`).

**Variants**

- `Author`: Wrote the work.
- `Narrator`: Reads the text in a narration.
- `Translator`: Translated the text.
- `Editor`: Edited the work or edition.
- `Illustrator`: Illustrated the work or edition.
- `Publisher`: Published the edition.
- `Host`: Hosts a podcast.
- `Cast`: Performs a part in a dramatisation.
- `Adaptor`: Adapted the work into a dramatisation.

#### CreditRole::ALL

```rust
pub const ALL: &'static [Self] = _
```

Every variant, in declaration order.

#### CreditRole::as_str

```rust
pub const fn as_str(self) -> &'static str
```

The text stored for this value in SQL.

#### CreditRole::parse

```rust
pub fn parse(text: &str) -> Option<Self>
```

Parses a stored value; `None` for text that names no variant.

#### CreditRole::default_level

```rust
pub const fn default_level(self) -> Level
```

The level this role attaches to unless a source says otherwise.

Implements: `Clone`, `Copy`, `Debug`, `Display`, `Eq`, `Hash`, `Ord`, `PartialEq`, `PartialOrd`

### Entitlement

```rust
pub enum Entitlement {
    Purchase,
    Plus,
    Loan,
    Free,
}
```

How a holder came to own a listing (`holdings.entitlement`).

**Variants**

- `Purchase`: Bought outright.
- `Plus`: Included with a store membership while it lasts.
- `Loan`: Borrowed for a time.
- `Free`: Given away by the store.

#### Entitlement::ALL

```rust
pub const ALL: &'static [Self] = _
```

Every variant, in declaration order.

#### Entitlement::as_str

```rust
pub const fn as_str(self) -> &'static str
```

The text stored for this value in SQL.

#### Entitlement::parse

```rust
pub fn parse(text: &str) -> Option<Self>
```

Parses a stored value; `None` for text that names no variant.

Implements: `Clone`, `Copy`, `Debug`, `Display`, `Eq`, `Hash`, `Ord`, `PartialEq`, `PartialOrd`

## Functions

### sql_check_list

```rust
pub fn sql_check_list(values: impl IntoIterator<Item = &'static str>) -> String
```

SQL `CHECK` list of an enum's stored values: `'a', 'b', 'c'`.

#### Arguments

* `values` - The stored spellings, usually `Enum::ALL` mapped through `as_str`.

#### Returns

The comma-separated, single-quoted list for an `IN (…)` clause.
