# bookclerk_source::abi

Host-side conversions between storefront types and the typed plugin ABI.

Everything a guest also needs lives in [`bookclerk_storefront::abi`](../bookclerk_storefront/abi.md) and is
re-exported here. This module adds the conversions that touch host-only
types: [`DownloadOptions`](README.md#downloadoptions) (built from the host configuration) and the
library rows a scan record becomes ([`NewBook`](../bookclerk_library/README.md#newbook), [`ScannedTitle`](../bookclerk_library/README.md#scannedtitle)).

| Item | Kind | Summary |
| --- | --- | --- |
| [`scanned_title`](#scanned_title) | function | The title a scan record describes: its scan fields and its listing. |
| [`scan_book_to_new`](#scan_book_to_new) | function | Maps a guest scan row onto the `NewBook` fields the library records for a title (its account, store text and, when the structured listing is refused, the listing they describe), forcing `source` to the plugin id so a guest can never claim another storefront's titles. The owning `plugin_key` is stamped by the `bookclerk_library::SourceScope` on write. |
| [`offered_stream`](#offered_stream) | function | The stream a store's quality offer names, as the upgrade rule compares it (`bookclerk_library::quality::is_upgrade`). |

## Re-exports

- [`bookclerk_storefront::abi::*`](../bookclerk_storefront/abi.md)

## Functions

### scanned_title

```rust
pub fn scanned_title(plugin_id: &str, book: ScanBook) -> Option<ScannedTitle>
```

The title a scan record describes: its scan fields and its listing.

The listing is `None` when the record fails listing validation (the
library then derives the title's listing from its scan fields); see
[`scan_book_to_new`](#scan_book_to_new) for when no title is returned at all.

#### Arguments

* `plugin_id` - The plugin that produced the record; it owns the listing.
* `book` - The scan record as the guest returned it.

### scan_book_to_new

```rust
pub fn scan_book_to_new(plugin_id: &str, book: ScanBook) -> Option<NewBook>
```

Maps a guest scan row onto the [`NewBook`](../bookclerk_library/README.md#newbook) fields the library records
for a title (its account, store text and, when the structured listing
is refused, the listing they describe), forcing `source` to the plugin
id so a guest can never claim another storefront's titles. The
owning `plugin_key` is stamped by the [`bookclerk_library::SourceScope`](../bookclerk_library/scope.md#sourcescope)
on write.

Credits join by role, the first ISBN and ASIN fill the identifier
columns (normalised), the first series entry fills the series columns,
and the work kind (with whether the record names a parent show) becomes
`content_kind`. An episode with no store
series id names its show through `parent_product_id`, which fills
`series_asin` so the episode is filed under the show.

The ASIN columns take an ASIN scoped to the book's marketplace, or an
unscoped one, before an ASIN from another marketplace.

#### Returns

`None` for a record with a blank product id, which would overwrite
another blank title, for a record that leaves a kind unspecified (the
host refuses those), and for an ebook or print edition, which the
library does not store yet.

### offered_stream

```rust
pub fn offered_stream(answer: &TitleQualityOffer) -> Option<StreamQuality>
```

The stream a store's quality offer names, as the upgrade rule compares
it ([`bookclerk_library::quality::is_upgrade`](../bookclerk_library/quality.md#is_upgrade)).

#### Returns

`None` for an answer that is not `found` or names no codec, which the
host treats as "the store did not say".
