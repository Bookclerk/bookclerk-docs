# bookclerk_scan_delta

Hash-based scan deltas for storefronts with no change feed.

A store such as Libro.fm, Chirp or GraphicAudio can only list a whole
library, so every pass fetches every store page. [`ScanDelta`](#scandelta) keeps a
fingerprint of each title the last pass listed in the plugin's own
database (the [`SCAN_STATE_BINDING`](#scan_state_binding) `[[databases]]` binding) and lets the
plugin send the host only the titles that are new or changed, plus the
product ids that disappeared. The store requests are the same; the host
receives, compares and writes far less.

## How a pass uses it

1. An empty `syncToken` is a full pass. Every title goes to the host, and
   the pass records one row per title.
2. The pass's last page marks the pass finished and returns a token
   naming it (`hd1.<generation>`). The host sends it on the account's
   next pass.
3. A pass with a token is a delta pass. Each page sends only titles whose
   product id and fingerprint the named pass does not hold. The last page
   lists the named pass's product ids that no page of this pass listed as
   `removed`, marks this pass finished and returns its token. A delta pass
   is never `complete`.

## State it keeps

Two tables, created by [`migrations`]:

- `scan_delta_titles`: one row per title a pass listed (account, pass
  generation, store page, product id, fingerprint).
- `scan_delta_passes`: one row per pass, finished or not, with the time it
  was last touched.

The account and store page columns hold hashes, never the account id or
the store's cursor. The generation of a pass is derived from the account
and the token the pass started with, so every call of one pass, resumed
or not, agrees on it without keeping state. The call that starts a pass
(empty cursor) deletes the account's rows other than the pass its token
names, and every pass of any account untouched for
[`RETENTION_SECONDS`](#retention_seconds). A finished pass is replaced only by a later pass,
so a host that lost the last page of a pass and sends the older token
again still finds the pass it needs.

Every write is split with the binding's `chunk()` and every read is
sized from the session's limits (one streamed read when the session
streams, key-ordered pages when it does not), so the same code runs on
the shipped adapters, on D1 and on the smallest advertisement a host
accepts. See `docs/plugins/database-limits.md`.

## When it falls back

- No binding (the operator has not approved it yet): every pass is a full
  pass with an empty token, as before.
- A token this crate did not write, or one whose pass is gone or never
  finished: `invalid_cursor`, so the host restarts once as a full pass.
- A pass whose row was cleared while it ran (another pass of the account
  started): its last page returns `invalid_cursor` instead of removals
  worked out from a partial record, and the host starts over.

| Item | Kind | Summary |
| --- | --- | --- |
| [`DeltaPage`](#deltapage) | struct | What one store page of a pass sends to the host. |
| [`ScanDelta`](#scandelta) | struct | Hash-based delta state for one call of one account's pass. |
| [`migrations`](#migrations) | function | The `database_migrations` registration for `SCAN_STATE_BINDING`. |
| [`migrations_for`](#migrations_for) | function | The `database_migrations` answer for `binding`. |
| [`SCAN_STATE_BINDING`](#scan_state_binding) | constant | `[[databases]]` binding first-party storefronts declare for this crate's state. |
| [`RETENTION_SECONDS`](#retention_seconds) | constant | Seconds a pass is kept after it was last touched. The host asks for a full pass at least every seven days, so an older pass is never the base of a delta; this only reclaims accounts that stopped scanning. |

## Structs

### DeltaPage

```rust
pub struct DeltaPage {
    pub books: Vec<ScanBook>,
    pub removed: Vec<String>,
    pub sync_token: String,
    pub complete: bool,
}
```

What one store page of a pass sends to the host.

**Fields**

- `books`: Titles to send: all of them on a full pass, the new or changed ones
  on a delta pass.
- `removed`: On the last store page of a delta pass: product ids the named pass
  held and no page of this pass listed, sorted.
- `sync_token`: On the last store page: the token for the account's next pass, empty
  when the next pass must be a full pass.
- `complete`: On the last store page: true for a full pass, which listed every
  title the account holds.

Implements: `Clone`, `Debug`, `Default`, `Eq`, `PartialEq`

### ScanDelta

```rust
pub struct ScanDelta<'a> {
    /* private fields */
}
```

Hash-based delta state for one call of one account's pass.

Build it with [`ScanDelta::open`](#scandeltaopen) at the top of `scan`, then pass every
store page through [`ScanDelta::page`](#scandeltapage).

#### ScanDelta::open

```rust
pub async fn open(
    db: Option<&'a DatabaseBinding>,
    params: &ScanParams,
) -> Result<Self, PluginError>
```

Reads the pass's token and, on the call that starts a pass, clears
the account's stale rows.

##### Arguments

* `db` - The plugin's scan-state database, when the operator granted
  it.
* `params` - The scan call's params (account, cursor, sync token).

##### Errors

Returns `invalid_cursor` for a token this crate did not write or
whose pass is gone or unfinished, so the host restarts as a full
pass, and the binding's error when a database call fails.

#### ScanDelta::page

```rust
pub async fn page(
    &self,
    store_cursor: &str,
    books: Vec<ScanBook>,
    last: bool,
) -> Result<DeltaPage, PluginError>
```

Records one store page and filters it; on the last one, ends the
pass.

The same store page may come through more than once (the SDK's
`scan_page` helper fetches a large page again for each slice); each
time gives the same answer.

##### Arguments

* `store_cursor` - The plugin's own cursor for this store page (empty
  for the first).
* `books` - Every title the store page lists, in store order.
* `last` - Whether this is the last store page of the pass.

##### Errors

Returns `invalid_cursor` on the last store page when another pass of
the account cleared this one while it ran, and the binding's error
when a database call fails.

Implements: `Debug`

## Functions

### migrations

```rust
pub fn migrations() -> Vec<PluginMigration>
```

The `database_migrations` registration for [`SCAN_STATE_BINDING`](#scan_state_binding).

Applied once, in order, before the binding is handed to the plugin.

### migrations_for

```rust
pub fn migrations_for(binding: &str) -> Vec<PluginMigration>
```

The `database_migrations` answer for `binding`.

#### Returns

[`migrations`](#migrations) for [`SCAN_STATE_BINDING`](#scan_state_binding), nothing for any other binding.

## Constants

### SCAN_STATE_BINDING

```rust
pub const SCAN_STATE_BINDING: &str = "SCAN_STATE";
```

`[[databases]]` binding first-party storefronts declare for this crate's
state.

### RETENTION_SECONDS

```rust
pub const RETENTION_SECONDS: i64 = _;
```

Seconds a pass is kept after it was last touched. The host asks for a
full pass at least every seven days, so an older pass is never the base
of a delta; this only reclaims accounts that stopped scanning.
