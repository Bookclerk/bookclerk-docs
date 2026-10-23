**bookclerk_integrations > portal > routes**

# Module: portal::routes

## Contents

**Structs**

- [`PortalState`](#portalstate) - Shared state for portal handlers.

**Functions**

- [`mint_for_external_user`](#mint_for_external_user) - Mint a claim ticket for an external user (daemon watcher / CLI).
- [`portal_identity_from_headers`](#portal_identity_from_headers) - Resolve a portal identity from the request cookie, if present and valid.
- [`portal_spa_router`](#portal_spa_router) - SPA-facing portal API. Nest under `/api/portal`.
- [`redeem_lose_next_responses`](#redeem_lose_next_responses) - Drop the next N successful redeem HTTP responses after the DB commit.

---

## bookclerk_integrations::portal::routes::PortalState

*Struct*

Shared state for portal handlers.

**Fields:**
- `config: std::sync::Arc<tokio::sync::RwLock<bookclerk_config::Config>>` - Loaded Bookclerk configuration shared with portal handlers.
- `library: std::sync::Arc<tokio::sync::RwLock<bookclerk_library::LibraryStore>>` - Open library store used for identity / ticket / session rows.
- `integrations: std::sync::Arc<tokio::sync::RwLock<crate::registry::IntegrationRegistry>>` - Configured outbound integration registry.
- `sources: std::sync::Arc<tokio::sync::RwLock<bookclerk_source::SourceRegistry>>` - Content-source registry used for portal store connect / OAuth.

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> PortalState`



## bookclerk_integrations::portal::routes::mint_for_external_user

*Function*

Mint a claim ticket for an external user (daemon watcher / CLI).

# Arguments

* `library` - Open library store used for reads/writes.
* `config` - Loaded Bookclerk configuration.
* `user` - External user identity from an integration login/watcher.
* `created_by` - Actor string recorded on the claim ticket (`daemon`, CLI user, …).

# Returns

On success, `crate::Result<crate::tickets::MintedClaimTicket>`.

# Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

```rust
fn mint_for_external_user(library: &bookclerk_library::LibraryStore, config: &bookclerk_config::Config, user: &crate::types::ExternalUser, created_by: &str) -> crate::Result<crate::tickets::MintedClaimTicket>
```



## bookclerk_integrations::portal::routes::portal_identity_from_headers

*Function*

Resolve a portal identity from the request cookie, if present and valid.

# Arguments

* `library` - Open library store used for reads/writes.
* `headers` - Incoming HTTP headers (cookie lookup).

# Returns

`Some(...)` when found / applicable; otherwise `None`.

```rust
fn portal_identity_from_headers(library: &bookclerk_library::LibraryStore, headers: &axum::http::HeaderMap) -> Option<bookclerk_library::PortalIdentity>
```



## bookclerk_integrations::portal::routes::portal_spa_router

*Function*

SPA-facing portal API. Nest under `/api/portal`.

# Arguments

* `state` - Shared portal / handler state.

# Returns

`Router` result.

```rust
fn portal_spa_router(state: PortalState) -> axum::Router
```



## bookclerk_integrations::portal::routes::redeem_lose_next_responses

*Function*

Drop the next N successful redeem HTTP responses after the DB commit.

Production stays at 0. Tests inject a committed-but-lost first reply so the
retry can recover the receipt through the real handler.

```rust
fn redeem_lose_next_responses(n: i32)
```



