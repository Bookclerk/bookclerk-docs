**bookclerk_plugin_source_chirp > client**

# Module: client

## Contents

**Structs**

- [`Audiobook`](#audiobook) - Chirp audiobook metadata (+ optional tracks).
- [`AuthorCatalog`](#authorcatalog) - Author summary expansion result.
- [`CatalogAudiobook`](#catalogaudiobook) - Catalog-oriented audiobook (search / related / series).
- [`CatalogAuthor`](#catalogauthor) - Chirp author metadata.
- [`CatalogSeries`](#catalogseries) - Chirp series metadata.
- [`CatalogTag`](#catalogtag) - Chirp `Tag` used as a promoted genre chip on catalog audiobooks.
- [`ChirpClient`](#chirpclient) - Authenticated Chirp GraphQL helper.
- [`ChirpProductPricing`](#chirpproductpricing) - Live Chirp storefront pricing (`audiobook.currentProduct`).
- [`LibraryItem`](#libraryitem) - One `currentUserAudiobooks` row.
- [`RelatedCatalog`](#relatedcatalog) - Related-audiobook expansion result.
- [`SeriesAudiobookRef`](#seriesaudiobookref) - Series membership on a catalog audiobook.
- [`SeriesCatalog`](#seriescatalog) - Series page expansion result.
- [`SignInUser`](#signinuser) - User object from `signIn`.
- [`Track`](#track) - One downloadable / playable track.
- [`TypeaheadCatalog`](#typeaheadcatalog) - Typeahead hits.

**Functions**

- [`chirp_slug_candidates`](#chirp_slug_candidates) - Slug guesses for Chirp `series(slug:)` / `author(slug:)`.

**Constants**

- [`DEFAULT_GRAPHQL_URL`](#default_graphql_url) - Android Mockingjay GraphQL endpoint.

---

## bookclerk_plugin_source_chirp::client::Audiobook

*Struct*

Chirp audiobook metadata (+ optional tracks).

**Fields:**
- `id: String` - Identifier.
- `display_title: Option<String>` - Display title.
- `sub_title: Option<String>` - Sub title.
- `display_authors: Option<String>` - Display authors.
- `display_narrators: Option<String>` - Display narrators.
- `duration_ms: Option<u64>` - Duration ms.
- `abridged: Option<bool>` - Abridged.
- `publisher: Option<String>` - Publisher.
- `released_on: Option<String>` - Released on.
- `cover_url: Option<String>` - Cover URL.
- `description: Option<String>` - Description.
- `tracks: Vec<Track>` - Tracks.

**Trait Implementations:**

- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> Audiobook`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_plugin_source_chirp::client::AuthorCatalog

*Struct*

Author summary expansion result.

**Fields:**
- `author: CatalogAuthor` - Author.
- `audiobooks: Vec<CatalogAudiobook>` - Audiobooks.

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> AuthorCatalog`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_plugin_source_chirp::client::CatalogAudiobook

*Struct*

Catalog-oriented audiobook (search / related / series).

**Fields:**
- `id: String` - Identifier.
- `display_title: Option<String>` - Display title.
- `display_authors: Option<String>` - Display authors.
- `display_narrators: Option<String>` - Display narrators.
- `url: Option<String>` - URL.
- `cover_url: Option<String>` - Cover URL.
- `series_audiobook: Option<SeriesAudiobookRef>` - Series audiobook.
- `sub_title: Option<String>` - Sub title.
- `description: Option<String>` - Description.
- `publisher: Option<String>` - Publisher.
- `duration_ms: Option<u64>` - Duration ms.
- `released_on: Option<String>` - Released on.
- `language: Option<String>` - Chirp display language (`English`, `Spanish`, …).
- `abridged: Option<bool>` - Abridged.
- `promoted_tags: Vec<CatalogTag>` - Storefront genre chips (`Thrillers`, `Crime Fiction & Mysteries`, …).

**Methods:**

- `fn title(self: &Self) -> String` - Title.
- `fn series_name(self: &Self) -> Option<String>` - Series name.

**Trait Implementations:**

- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> CatalogAudiobook`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_plugin_source_chirp::client::CatalogAuthor

*Struct*

Chirp author metadata.

**Fields:**
- `id: String` - Identifier.
- `name: String` - Display or configuration name.
- `slug: String` - Slug.

**Trait Implementations:**

- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> CatalogAuthor`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_plugin_source_chirp::client::CatalogSeries

*Struct*

Chirp series metadata.

**Fields:**
- `id: String` - Identifier.
- `name: String` - Display or configuration name.
- `slug: String` - Slug.

**Trait Implementations:**

- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> CatalogSeries`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_plugin_source_chirp::client::CatalogTag

*Struct*

Chirp `Tag` used as a promoted genre chip on catalog audiobooks.

**Fields:**
- `display_name: Option<String>` - Storefront genre-chip label (wire `displayName`).



## bookclerk_plugin_source_chirp::client::ChirpClient

*Struct*

Authenticated Chirp GraphQL helper.

**Methods:**

- `fn new<impl Into<String>>(graphql_url: impl Trait) -> Self` - Constructs a new instance with default or provided parameters.
- `fn with_http(self: Self, http: HttpClient) -> Self` - With HTTP.
- `fn with_token<impl Into<String>>(self: Self, token: impl Trait) -> Self` - With token.
- `fn access_token(self: &Self) -> Option<&str>` - Access token.
- `fn graphql_url(self: &Self) -> &str` - Graphql URL.
- `fn login(self: & mut Self, email: &str, password: &str) -> Result<SignInUser>` - GraphQL `signIn` mutation.
- `fn library_page(self: &Self, page: u32, page_size: u32) -> Result<Vec<LibraryItem>>` - Paginated owned library (`currentUserAudiobooks`).
- `fn audiobook(self: &Self, id: &str) -> Result<Audiobook>` - Full audiobook with track `mediaUrl`s.
- `fn audiobook_pricing(self: &Self, id: &str) -> Result<Option<ChirpProductPricing>>` - Live deal / list pricing for a Chirp audiobook (public GraphQL).
- `fn search_catalog(self: &Self, query: &str, page: u32, page_size: u32) -> Result<Vec<CatalogAudiobook>>` - Catalog search (`audiobooks(query:)`) — no auth required.
- `fn related_audiobooks(self: &Self, id: &str) -> Result<RelatedCatalog>` - Related titles for a Chirp audiobook id — no auth required.
- `fn series_catalog(self: &Self, slug: &str) -> Result<Option<SeriesCatalog>>` - Series catalog by slug (`mistborn-audiobooks`, …) — no auth required.
- `fn author_summary(self: &Self, slug: &str) -> Result<Option<AuthorCatalog>>` - Author summary titles by slug — no auth required.
- `fn typeahead(self: &Self, search_term: &str) -> Result<TypeaheadCatalog>` - Typeahead for author slug resolution and quick title hits — no auth required.
- `fn resolve_author_slug(self: &Self, author_name: &str) -> Result<Option<String>>` - Resolve a Chirp author slug via typeahead (exact name match only).
- `fn resolve_series_catalog(self: &Self, series_name: &str) -> Result<Option<SeriesCatalog>>` - Try common Chirp series slug forms derived from a series title.
- `fn top_deals(self: &Self, count: u32) -> Result<Vec<CatalogAudiobook>>` - Current Chirp top deals — no auth required.
- `fn free_deals(self: &Self) -> Result<Vec<CatalogAudiobook>>` - Current Chirp free deals — no auth required.
- `fn download_bytes(self: &Self, url: &str) -> Result<bytes::Bytes>` - Download bytes from an absolute media URL.

**Trait Implementations:**

- **Default**
  - `fn default() -> Self`
- **Clone**
  - `fn clone(self: &Self) -> ChirpClient`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_plugin_source_chirp::client::ChirpProductPricing

*Struct*

Live Chirp storefront pricing (`audiobook.currentProduct`).

**Fields:**
- `id: Option<String>` - Identifier.
- `discount_price: String` - Discount price.
- `discounted_price_cents: Option<i64>` - Discounted price cents.
- `listing_price: Option<String>` - Listing price.
- `is_free_listing: bool` - Is free listing.
- `hot_deal: bool` - Hot deal.
- `purchase_url: Option<String>` - Purchase URL.
- `salable_in_current_country: Option<bool>` - Salable in current country.

**Trait Implementations:**

- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> ChirpProductPricing`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_plugin_source_chirp::client::DEFAULT_GRAPHQL_URL

*Constant*: `&str`

Android Mockingjay GraphQL endpoint.



## bookclerk_plugin_source_chirp::client::LibraryItem

*Struct*

One `currentUserAudiobooks` row.

**Fields:**
- `id: String` - Identifier.
- `archived: Option<bool>` - Archived.
- `playable: Option<bool>` - Playable.
- `audiobook: Option<Audiobook>` - Audiobook.

**Trait Implementations:**

- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> LibraryItem`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_plugin_source_chirp::client::RelatedCatalog

*Struct*

Related-audiobook expansion result.

**Fields:**
- `related: Vec<CatalogAudiobook>` - Related.
- `series: Option<CatalogSeries>` - Series.

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> RelatedCatalog`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_plugin_source_chirp::client::SeriesAudiobookRef

*Struct*

Series membership on a catalog audiobook.

**Fields:**
- `number: Option<i64>` - Numeric series position when Chirp reports one.
- `display_number: Option<String>` - Storefront-facing series position (wire `displayNumber`, may be non-integer).
- `series: CatalogSeries` - Series identity (id, name, slug) this title belongs to.



## bookclerk_plugin_source_chirp::client::SeriesCatalog

*Struct*

Series page expansion result.

**Fields:**
- `series: CatalogSeries` - Series.
- `audiobooks: Vec<CatalogAudiobook>` - Audiobooks.

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> SeriesCatalog`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_plugin_source_chirp::client::SignInUser

*Struct*

User object from `signIn`.

**Fields:**
- `id: String` - Chirp user id returned by `signIn`.
- `token: String` - API bearer token used for authenticated GraphQL.
- `web_token: Option<String>` - Optional web-session token (wire `webToken`); unused by the Android client.
- `email: String` - Account email from `signIn`.



## bookclerk_plugin_source_chirp::client::Track

*Struct*

One downloadable / playable track.

**Fields:**
- `id: String` - Identifier.
- `media_url: Option<String>` - Media URL.
- `chapter_number: Option<i64>` - Chapter number.
- `part_number: Option<i64>` - Part number.
- `duration_ms: Option<u64>` - Duration ms.
- `offset_from_book_start_ms: Option<u64>` - Offset from book start ms.
- `display_name: Option<String>` - Display name.

**Trait Implementations:**

- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> Track`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_plugin_source_chirp::client::TypeaheadCatalog

*Struct*

Typeahead hits.

**Fields:**
- `audiobooks: Vec<CatalogAudiobook>` - Audiobooks.
- `authors: Vec<CatalogAuthor>` - Authors.

**Trait Implementations:**

- **Default**
  - `fn default() -> TypeaheadCatalog`
- **Clone**
  - `fn clone(self: &Self) -> TypeaheadCatalog`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_plugin_source_chirp::client::chirp_slug_candidates

*Function*

Slug guesses for Chirp `series(slug:)` / `author(slug:)`.

```rust
fn chirp_slug_candidates(name: &str) -> Vec<String>
```



