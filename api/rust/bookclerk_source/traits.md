**bookclerk_source > traits**

# Module: traits

## Contents

**Enums**

- [`PortalAuthMode`](#portalauthmode) - How the connect portal authenticates this source.

**Functions**

- [`revoke_credentials_default`](#revoke_credentials_default) - Shared revoke path for plugins that seal as `.plugin.auth` (and legacy names).
- [`source_scope`](#source_scope) - Opens the [`SourceScope`](../bookclerk_library/scope.md#bookclerk_libraryscopesourcescope) for `source`: rows are isolated by its

**Traits**

- [`ContentSource`](#contentsource) - Pluggable audiobook storefront (auth, scan, fetch, catalog).

---

## bookclerk_source::traits::ContentSource

*Trait*

Pluggable audiobook storefront (auth, scan, fetch, catalog).

Plugins own their id, brand, auth mode, and config parsing. Hosts register
concrete crates at startup and talk only through this trait.

All credential and library mutations go through [`SourceScope`](../bookclerk_library/scope.md#bookclerk_libraryscopesourcescope), which
forces `source` / `provider` to this plugin's id. First-party in-repo
adapters and third-party JSON-RPC plugins share the same scope rules.

**Methods:**

- `id`: Display / CLI alias (`audible`, `libro`, …). Globally unique within one
- `plugin_key`: Provenance-qualified PluginKey when this adapter is an external guest.
- `plugin_instance_id`: Plugin instance that owns this deployed source.
- `guest_pid`: Native guest pid when this process spawned the source.
- `display_name`: Human-facing store name for UI / logs.
- `aliases`: Alternate ids accepted by CLI / config (`libro.fm`, `ga`, …).
- `portal_auth_mode`: Connect-portal auth mode.
- `portal_brand`: Portal button brand (colors + favicon).
- `sort_key`: Stable sort key for registry listing (lower first).
- `login`: Authenticate and persist credentials via [`SourceScope`](../bookclerk_library/scope.md#bookclerk_libraryscopesourcescope).
- `login_with_oauth_progress`: Interactive OAuth login with progress (URL / waiting).
- `import_credentials`: Import credentials from a file (auth JSON, Libation export, …).
- `revoke_credentials`: Delete stored credentials for `account_id` (books / account rows kept).
- `list_accounts`: List accounts for this plugin (scope filters by source id).
- `scan`: Sync library rows using scoped credentials / upserts.
- `fetch_title`: Fetch everything needed to acquire one title (no storage writes).
- `config_options`: Source-native `[sources.<id>]` knobs for config / UI discovery.
- `inspect_title`: Optional diagnostic / license inspect for one title (opaque JSON).
- `search_catalog`: Public catalog typeahead / search (no account required).
- `catalog_detail`: Fetch rich bibliographic metadata for one catalog product (no account).
- `expand_candidates`: Expand related / series / author candidates from a taste seed.
- `purchase_hint`: Resolve a purchase / catalog URL (optionally with live price).
- `list_deals`: Current deals / promos for discovery shelves.



## bookclerk_source::traits::PortalAuthMode

*Enum*

How the connect portal authenticates this source.

**Variants:**
- `Oauth` - Browser / QR OAuth (Audible).
- `Password` - Username + password form.

**Methods:**

- `fn as_str(self: Self) -> &'static str` - Wire / portal id (`oauth` or `password`).

**Traits:** Eq, Copy

**Trait Implementations:**

- **PartialEq**
  - `fn eq(self: &Self, other: &PortalAuthMode) -> bool`
- **Clone**
  - `fn clone(self: &Self) -> PortalAuthMode`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_source::traits::revoke_credentials_default

*Function*

Shared revoke path for plugins that seal as `.plugin.auth` (and legacy names).

# Errors

Returns an error when the operation fails.

```rust
fn revoke_credentials_default(scope: &bookclerk_library::SourceScope, account_id: &str) -> crate::error::Result<()>
```



## bookclerk_source::traits::source_scope

*Function*

Opens the [`SourceScope`](../bookclerk_library/scope.md#bookclerk_libraryscopesourcescope) for `source`: rows are isolated by its
`ContentSource::plugin_key` and labelled with its `ContentSource::id`.

Every instance of one plugin shares this scope; accounts are not tied
to a plugin instance.

```rust
fn source_scope(library: &bookclerk_library::LibraryStore, source: &dyn ContentSource) -> bookclerk_library::SourceScope
```



