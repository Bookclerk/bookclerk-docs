**bookclerk_library > scope**

# Module: scope

## Contents

**Structs**

- [`SourceScope`](#sourcescope) - Host-enforced view of [`LibraryStore`](store.md#bookclerk_librarystorelibrarystore) for one content-source plugin.

---

## bookclerk_library::scope::SourceScope

*Struct*

Host-enforced view of [`LibraryStore`](store.md#bookclerk_librarystorelibrarystore) for one content-source plugin.

Both in-repo first-party adapters and external plugins must use this type
(via a `bookclerk_source::ContentSource` adapter or the external host
adapter). Secrets and books for other plugin keys are invisible. A
third-party package that reuses a first-party alias has a different key
and therefore sees nothing.

**Methods:**

- `fn new<impl Into<String>, impl Into<String>>(store: LibraryStore, plugin_key: impl Trait, source_id: impl Trait) -> Self` - Build a scope for `plugin_key` over `store`, labelling rows `source_id`.
- `fn plugin_key(self: &Self) -> &str` - Provenance-qualified PluginKey this scope isolates on.
- `fn source_id(self: &Self) -> &str` - Display alias (`audible`, `libro`, external `plugin.toml` id, …).
- `fn upsert_account(self: &Self, account_id: &str, marketplace: &str, label: Option<&str>, scan_enabled: bool) -> Result<AccountRecord>` - Upsert an account row; `source` is forced to this plugin id.
- `fn ensure_account(self: &Self, account_id: &str, marketplace: &str, label: Option<&str>) -> Result<AccountRecord>` - Ensure an account exists without flipping `scan_enabled` on conflict.
- `fn list_accounts(self: &Self) -> Result<Vec<AccountRecord>>` - Lists all store accounts in the library database.
- `fn get_account(self: &Self, account_id: &str) -> Result<Option<AccountRecord>>` - Fetch one account if it belongs to this plugin.
- `fn upsert_book(self: &Self, book: &NewBook) -> Result<BookRecord>` - Upsert a book; `plugin_key` and `source` are forced to this plugin.
- `fn get_book(self: &Self, title_id: &str, account_id: &str) -> Result<Option<BookRecord>>` - Look up one of this plugin's books by uuid, product id, ASIN, or ISBN.
- `fn save_source_auth(self: &Self, account_id: &str, name: &str, plaintext: &[u8]) -> Result<()>` - Seal and store a `source_auth` credential for `account_id`.
- `fn save_secret(self: &Self, kind: &str, account_id: &str, name: &str, plaintext: &[u8]) -> Result<()>` - Seal and store an arbitrary secret kind under this plugin's provider.
- `fn load_source_auth(self: &Self, account_id: &str, name: &str) -> Result<Option<Vec<u8>>>` - Load and unseal a `source_auth` credential for this plugin only.
- `fn load_secret(self: &Self, kind: &str, account_id: &str, name: &str) -> Result<Option<Vec<u8>>>` - Load and unseal a secret for this plugin only.
- `fn get_source_auth_record(self: &Self, account_id: &str, name: &str) -> Result<Option<EncryptedSecretRecord>>` - Fetch the sealed record (for formats that need custom unseal, e.g. Audible).
- `fn get_secret_record(self: &Self, kind: &str, account_id: &str, name: &str) -> Result<Option<EncryptedSecretRecord>>` - Fetch a sealed record of any kind for this plugin (custom unseal / legacy formats).
- `fn list_source_auth(self: &Self) -> Result<Vec<EncryptedSecretRecord>>` - List `source_auth` secrets for this plugin only (never other providers).
- `fn delete_source_auth(self: &Self, account_id: &str, name: &str) -> Result<()>` - Delete a `source_auth` secret for this plugin.
- `fn delete_secret(self: &Self, kind: &str, account_id: &str, name: &str) -> Result<()>` - Delete a secret of any kind for this plugin.
- `fn save_credentials_json(self: &Self, account_id: &str, credentials: &Value) -> Result<()>` - Upsert opaque JSON credentials (external plugin login blob).
- `fn load_credentials_json(self: &Self, account_id: &str) -> Result<Option<Value>>` - Load opaque JSON credentials for an external (or first-party) account.

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut std::fmt::Formatter) -> std::fmt::Result`
- **Clone**
  - `fn clone(self: &Self) -> SourceScope`



