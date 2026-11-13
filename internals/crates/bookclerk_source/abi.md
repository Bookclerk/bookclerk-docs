# bookclerk_source::abi

Host-side conversions between storefront types and the typed plugin ABI.

Everything a guest also needs lives in [`bookclerk_storefront::abi`](../bookclerk_storefront/abi.md) and is
re-exported here. This module adds the conversions that touch host-only
types: [`DownloadOptions`](README.md#downloadoptions) (built from the host configuration) and the
library rows a scan record becomes ([`NewBook`](../bookclerk_library/README.md#newbook), [`ScannedTitle`](../bookclerk_library/README.md#scannedtitle)).

| Item | Kind | Summary |
| --- | --- | --- |
| [`scanned_title`](#scanned_title) | function | The title a scan record describes: its flat row and its listing. |
| [`scan_book_to_new`](#scan_book_to_new) | function | Maps a guest scan row onto the flat `NewBook` row the library stores until it keeps works, expressions and editions, forcing `source` to the plugin id so a guest can never claim another storefront's rows. The owning `plugin_key` is stamped by the `bookclerk_library::SourceScope` on write. |

## Re-exports

- [`bookclerk_storefront::abi::*`](../bookclerk_storefront/abi.md)

## Functions

### scanned_title

```rust
pub fn scanned_title(plugin_id: &str, book: ScanBook) -> Option<ScannedTitle>
```

The title a scan record describes: its flat row and its listing.

The listing is `None` when the record fails listing validation (the
flat row is still stored and its own listing derived from it); see
[`scan_book_to_new`](#scan_book_to_new) for when no title is returned at all.

#### Arguments

* `plugin_id` - The plugin that produced the record; it owns the listing.
* `book` - The scan record as the guest returned it.

### scan_book_to_new

```rust
pub fn scan_book_to_new(plugin_id: &str, book: ScanBook) -> Option<NewBook>
```

Maps a guest scan row onto the flat [`NewBook`](../bookclerk_library/README.md#newbook) row the library stores
until it keeps works, expressions and editions, forcing `source` to the
plugin id so a guest can never claim another storefront's rows. The
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
another blank row, for a record that leaves a kind unspecified (the
host refuses those), and for an ebook or print edition: the flat table has
no way to keep those out of acquire, so they are not stored until the
library keeps editions.
