# bookclerk_storefront::listing

Plugin ABI records as resolver listing drafts.

The capnp enums and the resolver's stored enums are separate types: the
wire may grow a variant before the host stores it. Each mapping here is
an exhaustive `match`, so a new variant on either side fails to compile
until it is mapped.

| Item | Kind | Summary |
| --- | --- | --- |
| [`work_kind`](#work_kind) | function | The resolver's work kind for a wire work kind. |
| [`expression_kind`](#expression_kind) | function | The resolver's expression kind for a wire expression kind. |
| [`media_kind`](#media_kind) | function | The resolver's media kind for a wire media kind. |
| [`abridgement`](#abridgement) | function | The resolver's abridgement for a wire abridgement. |
| [`identifier_scheme`](#identifier_scheme) | function | The resolver's identifier scheme for a wire scheme. |
| [`credit_role`](#credit_role) | function | The resolver's credit role for a wire credit role. |
| [`credit_level`](#credit_level) | function | The level a wire credit names; `None` for the role's usual level. |
| [`scan_book_draft`](#scan_book_draft) | function | The listing draft a storefront's scan record describes. |
| [`catalog_hit_draft`](#catalog_hit_draft) | function | The listing draft a storefront's catalog hit describes, for the shared catalog pool. |

## Functions

### work_kind

```rust
pub fn work_kind(kind: WorkKind) -> Option<WorkKind>
```

The resolver's work kind for a wire work kind.

#### Returns

`None` for `unspecified`, so `bookclerk_resolve::Listing::from_draft`
refuses the record.

### expression_kind

```rust
pub fn expression_kind(kind: ExpressionKind) -> Option<ExpressionKind>
```

The resolver's expression kind for a wire expression kind.

#### Returns

`None` for `unspecified`, so the record is refused.

### media_kind

```rust
pub fn media_kind(kind: MediaKind) -> Option<MediaKind>
```

The resolver's media kind for a wire media kind.

#### Returns

`None` for `unspecified`, so the record is refused.

### abridgement

```rust
pub fn abridgement(value: Abridgement) -> Abridgement
```

The resolver's abridgement for a wire abridgement.

### identifier_scheme

```rust
pub fn identifier_scheme(scheme: IdentifierScheme) -> IdentifierScheme
```

The resolver's identifier scheme for a wire scheme.

### credit_role

```rust
pub fn credit_role(role: CreditRole) -> CreditRole
```

The resolver's credit role for a wire credit role.

### credit_level

```rust
pub fn credit_level(level: CreditLevel) -> Option<Level>
```

The level a wire credit names; `None` for the role's usual level.

### scan_book_draft

```rust
pub fn scan_book_draft(book: &ScanBook) -> ListingDraft
```

The listing draft a storefront's scan record describes.

The plugin key is left blank: the host sets it from the plugin it ran,
never from the record. A publisher named only in `publisher` becomes a
publisher credit. Human-readable text is HTML-entity decoded, as the
flat row is.

#### Arguments

* `book` - The scan record as the guest returned it.

#### Returns

The unchecked draft; `bookclerk_resolve::Listing::from_draft` validates it.

### catalog_hit_draft

```rust
pub fn catalog_hit_draft(hit: &CatalogHit, marketplace: &str) -> ListingDraft
```

The listing draft a storefront's catalog hit describes, for the shared
catalog pool.

Like [`scan_book_draft`](#scan_book_draft), the plugin key is left blank for the host to
set from the plugin it asked. The marketplace is the one the host asked
the store about (catalog hits carry none of their own). Abridgement is
unknown unless the store flagged it. Human-readable text is HTML-entity
decoded, as for scans, so match keys agree with a later scan of the same
title.

#### Arguments

* `hit` - The catalog hit as the guest returned it.
* `marketplace` - Marketplace the store was asked about (`us`).

#### Returns

The unchecked draft; `bookclerk_resolve::Listing::from_draft` validates it.
