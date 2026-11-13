# bookclerk_library::scope

Plugin-scoped library access.

First-party and third-party content sources share the same boundary:
a [`SourceScope`](#sourcescope) can only read/write rows whose `plugin_key` / `provider`
matches the provenance-qualified PluginKey it was opened for. Rows are not
tied to a plugin instance: every instance of that plugin, on any host,
sees the same accounts, credentials, and books. Host code that needs the
full store keeps its own [`LibraryStore`](README.md#librarystore) handle — this type does not
expose one.

| Item | Kind | Summary |
| --- | --- | --- |
| [`SourceScope`](#sourcescope) | struct | Host-enforced view of `LibraryStore` for one content-source plugin. |
| [`RefreshKeeping`](#refreshkeeping) | enum | How a storefront's sign-ins may be held, as its manifest declares it, for `SourceScope::replace_refreshed_credentials`. |

## Structs

### SourceScope

```rust
pub struct SourceScope {
    /* private fields */
}
```

Host-enforced view of [`LibraryStore`](README.md#librarystore) for one content-source plugin.

Both in-repo first-party adapters and external plugins must use this type
(via a `bookclerk_source::ContentSource` adapter or the external host
adapter). Secrets and books for other plugin keys are invisible. A
third-party package that reuses a first-party alias has a different key
and therefore sees nothing.

#### SourceScope::new

```rust
pub fn new(
    store: LibraryStore,
    plugin_key: impl Into<String>,
    source_id: impl Into<String>,
) -> Self
```

Build a scope for `plugin_key` over `store`, labelling rows `source_id`.

#### SourceScope::plugin_key

```rust
pub fn plugin_key(&self) -> &str
```

Provenance-qualified PluginKey this scope isolates on.

#### SourceScope::source_id

```rust
pub fn source_id(&self) -> &str
```

Display alias (`audible`, `libro`, external `plugin.toml` id, …).

#### SourceScope::upsert_account

```rust
pub async fn upsert_account(
    &self,
    account_id: &str,
    marketplace: &str,
    label: Option<&str>,
    scan_enabled: bool,
) -> Result<AccountRecord>
```

Upsert an account row; `source` is forced to this plugin id.

##### Errors

Returns an error when the operation fails.

#### SourceScope::ensure_account

```rust
pub async fn ensure_account(
    &self,
    account_id: &str,
    marketplace: &str,
    label: Option<&str>,
) -> Result<AccountRecord>
```

Ensure an account exists without flipping `scan_enabled` on conflict.

##### Errors

Returns an error when the operation fails.

#### SourceScope::list_accounts

```rust
pub async fn list_accounts(&self) -> Result<Vec<AccountRecord>>
```

Lists all store accounts in the library database.


##### Returns

`Result<Vec<AccountRecord>>` — `Ok` on success.

##### Errors

Returns a crate error when the database operation fails or inputs are invalid.

#### SourceScope::get_account

```rust
pub async fn get_account(&self, account_id: &str) -> Result<Option<AccountRecord>>
```

Fetch one account if it belongs to this plugin.

##### Errors

Returns an error when the operation fails.

#### SourceScope::upsert_book

```rust
pub async fn upsert_book(&self, book: &NewBook) -> Result<BookRecord>
```

Upsert a book; `plugin_key` and `source` are forced to this plugin.

##### Errors

Returns an error when the operation fails.

#### SourceScope::upsert_scanned_title

```rust
pub async fn upsert_scanned_title(&self, title: &ScannedTitle) -> Result<BookRecord>
```

Upsert a scanned title as this plugin's, and resolve its listing.

##### Errors

Returns an error when a read or write fails.

#### SourceScope::upsert_scanned_titles

```rust
pub async fn upsert_scanned_titles(
    &self,
    titles: &[ScannedTitle],
) -> Result<Vec<BookRecord>>
```

Upsert a page of scanned titles as this plugin's, and resolve their
listings as one page (see [`LibraryStore::upsert_scanned_titles`](README.md#librarystoreupsert_scanned_titles)).

##### Errors

Returns an error when a flat row cannot be read or written.

#### SourceScope::scan_state

```rust
pub async fn scan_state(&self, account_id: &str) -> Result<Option<ScanState>>
```

Where one of this plugin's accounts' scan stands.

##### Errors

Returns an error when the read fails.

#### SourceScope::save_scan_checkpoint

```rust
pub async fn save_scan_checkpoint(
    &self,
    account_id: &str,
    cursor: &str,
    pass_token: &str,
    pass_id: &str,
) -> Result<()>
```

Checkpoints a pass of one of this plugin's accounts after a page.

##### Errors

Returns an error when the write fails.

#### SourceScope::mark_holdings_seen

```rust
pub async fn mark_holdings_seen(
    &self,
    account_id: &str,
    pass_id: &str,
    product_ids: &[String],
) -> Result<()>
```

Stamps scan pass `pass_id` on one of this plugin's accounts' holdings
of `product_ids` (see [`LibraryStore::mark_holdings_seen`](README.md#librarystoremark_holdings_seen)).

##### Errors

Returns an error when a write fails.

#### SourceScope::remove_account_holdings

```rust
pub async fn remove_account_holdings(
    &self,
    account_id: &str,
    selection: HoldingSelection<'_>,
) -> Result<usize>
```

Removes one of this plugin's accounts' holdings that `selection`
names, publishing `holding_removed` for each (see
[`LibraryStore::remove_account_holdings`](README.md#librarystoreremove_account_holdings)).

##### Returns

How many holdings were selected for removal.

##### Errors

Returns an error when a read or write fails.

#### SourceScope::finish_scan_pass

```rust
pub async fn finish_scan_pass(
    &self,
    account_id: &str,
    sync_token: &str,
    complete: bool,
) -> Result<()>
```

Records the end of a pass of one of this plugin's accounts.

##### Errors

Returns an error when the write fails.

#### SourceScope::reset_scan_state

```rust
pub async fn reset_scan_state(&self, account_id: &str) -> Result<()>
```

Forgets the sync token and resume cursor of one of this plugin's
accounts.

##### Errors

Returns an error when the write fails.

#### SourceScope::get_book

```rust
pub async fn get_book(
    &self,
    title_id: &str,
    account_id: &str,
) -> Result<Option<BookRecord>>
```

Look up one of this plugin's books by uuid, product id, ASIN, or ISBN.

##### Errors

Returns an error when the operation fails.

#### SourceScope::save_source_auth

```rust
pub async fn save_source_auth(
    &self,
    account_id: &str,
    name: &str,
    plaintext: &[u8],
) -> Result<()>
```

Seal and store a `source_auth` credential for `account_id`.

##### Errors

Returns an error when the operation fails.

#### SourceScope::save_secret

```rust
pub async fn save_secret(
    &self,
    kind: &str,
    account_id: &str,
    name: &str,
    plaintext: &[u8],
) -> Result<()>
```

Seal and store an arbitrary secret kind under this plugin's provider.

Used for Widevine CDMs (`secret_kind::WIDEVINE`) and similar. Provider is
always this plugin key.

##### Errors

Returns an error when the operation fails.

#### SourceScope::load_source_auth

```rust
pub async fn load_source_auth(
    &self,
    account_id: &str,
    name: &str,
) -> Result<Option<Vec<u8>>>
```

Load and unseal a `source_auth` credential for this plugin only.

##### Errors

Returns an error when the operation fails.

#### SourceScope::load_secret

```rust
pub async fn load_secret(
    &self,
    kind: &str,
    account_id: &str,
    name: &str,
) -> Result<Option<Vec<u8>>>
```

Load and unseal a secret for this plugin only.

##### Errors

Returns an error when the operation fails.

#### SourceScope::get_source_auth_record

```rust
pub async fn get_source_auth_record(
    &self,
    account_id: &str,
    name: &str,
) -> Result<Option<EncryptedSecretRecord>>
```

Fetch the sealed record (for formats that need custom unseal, e.g. Audible).

##### Errors

Returns an error when the operation fails.

#### SourceScope::get_secret_record

```rust
pub async fn get_secret_record(
    &self,
    kind: &str,
    account_id: &str,
    name: &str,
) -> Result<Option<EncryptedSecretRecord>>
```

Fetch a sealed record of any kind for this plugin (custom unseal / legacy formats).

##### Errors

Returns an error when the operation fails.

#### SourceScope::list_source_auth

```rust
pub async fn list_source_auth(&self) -> Result<Vec<EncryptedSecretRecord>>
```

List `source_auth` secrets for this plugin only (never other providers).

##### Errors

Returns an error when the operation fails.

#### SourceScope::delete_source_auth

```rust
pub async fn delete_source_auth(&self, account_id: &str, name: &str) -> Result<()>
```

Delete a `source_auth` secret for this plugin.

##### Errors

Returns an error when the operation fails.

#### SourceScope::delete_secret

```rust
pub async fn delete_secret(&self, kind: &str, account_id: &str, name: &str) -> Result<()>
```

Delete a secret of any kind for this plugin.

##### Errors

Returns an error when the operation fails.

#### SourceScope::save_credentials_json

```rust
pub async fn save_credentials_json(
    &self,
    account_id: &str,
    credentials: &Value,
) -> Result<()>
```

Upsert opaque JSON credentials (external plugin login blob).

##### Errors

Returns an error when the operation fails.

#### SourceScope::keeping_branch

```rust
pub async fn keeping_branch(&self, account_id: &str) -> Result<Option<String>>
```

The branch that keeps this account's login fresh for this library
(the branch it was shared with, once that branch has pushed a
sign-in here; see [`crate::control_plane::confirmed_keeping_branch`](control_plane.md#confirmed_keeping_branch)),
or `None` when this install keeps it. For a single-keeper storefront the host then saves
none of its own refreshes of the account: the branch pushes them.

##### Errors

Returns the read error.

#### SourceScope::replace_refreshed_credentials

```rust
pub async fn replace_refreshed_credentials(
    &self,
    account_id: &str,
    sent: &Value,
    refreshed: &[u8],
    keeping: RefreshKeeping,
) -> Result<bool>
```

Stores the credentials a guest handed back after refreshing them
(`ScanPage.refreshedCredentials`, `PlainFetch.refreshedCredentials`)
in place of `sent`, the credentials the host gave it.

Written only while the stored credentials are still `sent`: when the
account was signed in again (or refreshed by another call) after
`sent` was read, the newer credentials stay and nothing is written.
The check and the write are one host batch guarded on the sealed row
read, so a sign-in that lands in between wins. Nothing is written
when the account has no stored credentials (it was disconnected),
nor, for a [`RefreshKeeping::SingleKeeper`](#refreshkeeping) storefront, once a branch
has taken the account's sign-in over for this library (it has applied
a push from one, a `node:` row in `account_credential_revisions`),
checked in the same batch so a first push that lands in between wins
too. A shared storefront ignores those rows: no branch takes its
sign-ins over, so a row left from when it kept a single sign-in
never blocks its saves.
The new value is registered with log redaction before it is stored.

##### Arguments

* `account_id` - The account the guest refreshed.
* `sent` - The credentials the host sent the guest.
* `refreshed` - The credentials the guest handed back, already
  checked as one JSON object within the ABI bound.
* `keeping` - How the storefront's manifest says its sign-ins may be
  held, which decides whether a keeping branch fences the write.

##### Returns

True when the refreshed credentials were stored.

##### Errors

Returns an error when `refreshed` is not JSON, or a read, the seal or
the write fails.

#### SourceScope::load_credentials_json

```rust
pub async fn load_credentials_json(&self, account_id: &str) -> Result<Option<Value>>
```

Load opaque JSON credentials for an external (or first-party) account.

##### Errors

Returns an error when the operation fails.

Implements: `Clone`, `Debug`

## Enums

### RefreshKeeping

```rust
pub enum RefreshKeeping {
    Shared,
    SingleKeeper,
}
```

How a storefront's sign-ins may be held, as its manifest declares it,
for [`SourceScope::replace_refreshed_credentials`](#sourcescopereplace_refreshed_credentials).

**Variants**

- `Shared`: Every holder refreshes on its own; no branch takes a sign-in over,
  so the library always saves its own refreshes.
- `SingleKeeper`: One install refreshes each sign-in; once a branch keeps an account
  for this library, the library's own refreshes of it are not saved.

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `PartialEq`
