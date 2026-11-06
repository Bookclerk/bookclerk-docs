# bookclerk_resolve::identifiers

External identifiers: normalisation and the key they are stored under.

| Item | Kind | Summary |
| --- | --- | --- |
| [`IdentifierKey`](#identifierkey) | struct | One normalised external identifier, as stored and compared. |
| [`normalize_identifier`](#normalize_identifier) | function | Trims and normalises an identifier value for its scheme. |

## Structs

### IdentifierKey

```rust
pub struct IdentifierKey {
    pub scheme: IdentifierScheme,
    pub value: String,
    pub marketplace: String,
}
```

One normalised external identifier, as stored and compared.

`marketplace` is the lower-cased marketplace for a scheme scoped to one
([`IdentifierScheme::marketplace_scoped`](kinds.md#identifierschememarketplace_scoped)) and empty for a global scheme,
so two spellings of one identifier always compare equal.

**Fields**

- `scheme`: Scheme the value belongs to.
- `value`: Normalised value (ISBN-13 without separators, upper-case ASIN, …).
- `marketplace`: Lower-cased marketplace for a scoped scheme; empty otherwise.

#### IdentifierKey::new

```rust
pub fn new(
    scheme: IdentifierScheme,
    raw: &str,
    marketplace: Option<&str>,
    fallback_marketplace: &str,
) -> Option<Self>
```

Normalises a raw identifier into its stored key.

##### Arguments

* `scheme` - The identifier's scheme.
* `raw` - The value as the source printed it.
* `marketplace` - The marketplace the source scoped it to, if any;
  used only for a scoped scheme.
* `fallback_marketplace` - The listing's own marketplace, used when a
  scoped identifier names none.

##### Returns

`None` when the value is blank after normalising.

#### IdentifierKey::level

```rust
pub fn level(&self) -> Level
```

The level whose identifier table this key belongs in.

#### IdentifierKey::strength

```rust
pub fn strength(&self) -> Strength
```

Whether this key names exactly one edition (or one work).

#### IdentifierKey::strong_key

```rust
pub fn strong_key(&self) -> Option<String>
```

The unique key a strong identifier is stored under, `None` for a weak one.

Strong edition identifiers are unique across the library. SQL keeps
that with a nullable `UNIQUE` column holding this text (`NULL` for
weak rows, which never conflict).

Implements: `Clone`, `Debug`, `Eq`, `Hash`, `Ord`, `PartialEq`, `PartialOrd`

## Functions

### normalize_identifier

```rust
pub fn normalize_identifier(scheme: IdentifierScheme, raw: &str) -> Option<String>
```

Trims and normalises an identifier value for its scheme.

ASINs are upper-cased. ISBNs lose hyphens, spaces and Unicode dashes,
are upper-cased (`x` check digit), and a valid ISBN-10 becomes its
ISBN-13. OpenLibrary ids are kept as given.

#### Arguments

* `scheme` - The identifier's scheme.
* `raw` - The value as the source printed it.

#### Returns

`None` when the value is blank.
