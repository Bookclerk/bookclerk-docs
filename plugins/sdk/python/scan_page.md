# bookclerk_plugin_sdk.scan_page

Paging helper for a storefront's `scan`.

A store page can hold more titles than one reply may carry
(`MAX_SCAN_PAGE_BOOKS`). [`scan_page`](#scan_page) fetches the store page the cursor
names, returns the next slice of it, and writes a cursor that names both the
store page and the slice, so the plugin pages once and stays stateless
between calls. Cursors match the TypeScript SDK's `scanPage`. Inside a
workerd isolate, import it as ``from bookclerk_plugin_sdk.scan_page import
scan_page``.

| Name | Kind | Summary |
| --- | --- | --- |
| [`StorePage`](#storepage) | class | One page of a storefront's own library listing, as the plugin fetched it. |
| [`scan_page`](#scan_page) | function | Answers one `scan` call from the storefront's own pages. |
| [`FetchStorePage`](#fetchstorepage) | constant | Fetches the store page a storefront cursor names (empty for the first). |

## Classes

### StorePage

```python
class StorePage(_StorePageRequired, total=False)
```

One page of a storefront's own library listing, as the plugin fetched it.

**Attributes**

- `removed`: Product ids gone from the account since `ScanParams.syncToken`, on a delta pass only; any number.
- `next`: The storefront's own cursor for the store page after this one; empty or absent when this is the last store page.
- `syncToken`: On the last store page, the token the host sends as `ScanParams.syncToken` on the account's next pass.
- `complete`: On the last store page, whether the pass listed every title the account holds (a full pass).
- `storePages`: Storefront requests this fetch took; 1 when absent.

## Functions

### scan_page

```python
async def scan_page(params: ScanParams, fetch_store_page: FetchStorePage) -> ScanPage
```

Answers one `scan` call from the storefront's own pages.

Fetches the store page `params["cursor"]` names and returns at most
`MAX_SCAN_PAGE_BOOKS` of its titles and removed ids. A larger store page
is returned over several calls, each fetching it again. The pass ends on
the last slice of the store page whose `next` is empty, which carries
that page's `syncToken` and `complete`.

**Args**

- `params`: The scan call's params.
- `fetch_store_page`: Fetches one store page from its storefront cursor.

**Returns**

The reply for this call.

**Raises**

- `PluginError`: `invalid_cursor` when the cursor was not written by this helper or no longer fits the store page (the store changed mid-pass), so the host restarts the pass; `internal` when a store cursor is too long for a scan cursor.

## Constants

### FetchStorePage

```python
FetchStorePage = Callable[[str, 'ScanParams'], Awaitable[StorePage]]
```

Fetches the store page a storefront cursor names (empty for the first).
