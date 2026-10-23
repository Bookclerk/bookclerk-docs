**bookclerk_integrations > tickets**

# Module: tickets

## Contents

**Structs**

- [`MintedClaimTicket`](#mintedclaimticket) - Freshly minted claim ticket (includes plaintext once).

**Functions**

- [`generate_token`](#generate_token) - Generate a URL-safe random token.
- [`identity_from_session`](#identity_from_session) - Resolve identity from a portal session cookie value.
- [`mint_claim_ticket`](#mint_claim_ticket) - Ensure identity exists and mint a claim ticket.
- [`redeem_ticket_to_session`](#redeem_ticket_to_session) - Redeem a claim ticket into a portal session cookie value (plaintext).
- [`redeem_ticket_to_session_with_client`](#redeem_ticket_to_session_with_client) - Redeem a claim ticket and mint a portal session with optional client metadata.
- [`session_for_identity`](#session_for_identity) - Create a portal session for an already-resolved identity (credential login).
- [`session_for_identity_with_client`](#session_for_identity_with_client) - Create a portal session for an identity with optional client metadata.
- [`ticket_portal_url`](#ticket_portal_url) - Build a shareable SPA claim URL when `public_origin` is configured.

---

## bookclerk_integrations::tickets::MintedClaimTicket

*Struct*

Freshly minted claim ticket (includes plaintext once).

**Fields:**
- `record: bookclerk_library::ClaimTicketRecord` - Persisted claim-ticket row (hash only; no plaintext token).
- `token: String` - Plaintext secret shown once at mint time; only a hash is persisted.
- `identity: bookclerk_library::PortalIdentity` - Portal identity bound to this ticket.
- `portal_url: Option<String>` - Shareable SPA claim URL when `public_origin` is configured.

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> MintedClaimTicket`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_integrations::tickets::generate_token

*Function*

Generate a URL-safe random token.

```rust
fn generate_token() -> String
```



## bookclerk_integrations::tickets::identity_from_session

*Function*

Resolve identity from a portal session cookie value.

# Arguments

* `library` - Open library store used for reads/writes.
* `raw_session` - Plaintext portal session cookie value.

# Returns

On success, the inner `Option<PortalIdentity>` value.

# Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

```rust
fn identity_from_session(library: &bookclerk_library::LibraryStore, raw_session: &str) -> crate::error::Result<Option<bookclerk_library::PortalIdentity>>
```



## bookclerk_integrations::tickets::mint_claim_ticket

*Function*

Ensure identity exists and mint a claim ticket.

# Arguments

* `library` - Open library store used for reads/writes.
* `integrations` - Integrations config (TTL, public origin, …).
* `user` - External user identity from an integration login/watcher.
* `created_by` - Actor string recorded on the claim ticket (`daemon`, CLI user, …).

# Returns

On success, the inner `MintedClaimTicket` value.

# Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

```rust
fn mint_claim_ticket(library: &bookclerk_library::LibraryStore, integrations: &bookclerk_config::IntegrationsConfig, user: &crate::types::ExternalUser, created_by: &str) -> crate::error::Result<MintedClaimTicket>
```



## bookclerk_integrations::tickets::redeem_ticket_to_session

*Function*

Redeem a claim ticket into a portal session cookie value (plaintext).

# Arguments

* `library` - Open library store used for reads/writes.
* `integrations` - Integrations config (TTL, public origin, …).
* `raw_ticket` - Plaintext claim ticket from the SPA / share URL.

# Returns

The successful result value for this operation.

# Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

```rust
fn redeem_ticket_to_session(library: &bookclerk_library::LibraryStore, integrations: &bookclerk_config::IntegrationsConfig, raw_ticket: &str, nonce: &str) -> crate::error::Result<(String, bookclerk_library::PortalIdentity)>
```



## bookclerk_integrations::tickets::redeem_ticket_to_session_with_client

*Function*

Redeem a claim ticket and mint a portal session with optional client metadata.

`password_hash` is applied in the same transaction as ticket consume and
session insert, and only while the ticket-bound local user has no password.
`password_fingerprint` is hashed into the atomic request so a retry with
a freshly salted Argon2id encoding still matches the receipt.

The raw session token is derived from the claim ticket, browser nonce, and
process DEK so a lost atomic reply plus a new HTTP request reuse the
same operation id. Possession of the used magic link without the nonce
cannot recover the session.

# Errors

Returns an error when the operation fails.

```rust
fn redeem_ticket_to_session_with_client(library: &bookclerk_library::LibraryStore, integrations: &bookclerk_config::IntegrationsConfig, raw_ticket: &str, nonce: &str, client: Option<&bookclerk_library::SessionClientInfo>, password_hash: Option<&str>, password_fingerprint: Option<&str>) -> crate::error::Result<(String, bookclerk_library::PortalIdentity)>
```



## bookclerk_integrations::tickets::session_for_identity

*Function*

Create a portal session for an already-resolved identity (credential login).

# Arguments

* `library` - Open library store used for reads/writes.
* `integrations` - Integrations config (TTL, public origin, …).
* `identity` - Already-resolved portal identity.

# Returns

On success, the inner `String` value.

# Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

```rust
fn session_for_identity(library: &bookclerk_library::LibraryStore, integrations: &bookclerk_config::IntegrationsConfig, identity: &bookclerk_library::PortalIdentity) -> crate::error::Result<String>
```



## bookclerk_integrations::tickets::session_for_identity_with_client

*Function*

Create a portal session for an identity with optional client metadata.

# Errors

Returns an error when the operation fails.

```rust
fn session_for_identity_with_client(library: &bookclerk_library::LibraryStore, integrations: &bookclerk_config::IntegrationsConfig, identity: &bookclerk_library::PortalIdentity, client: Option<&bookclerk_library::SessionClientInfo>) -> crate::error::Result<String>
```



## bookclerk_integrations::tickets::ticket_portal_url

*Function*

Build a shareable SPA claim URL when `public_origin` is configured.

Opens the Bookclerk GUI invite page (`/invite?ticket=`).

# Arguments

* `integrations` - Integrations config (TTL, public origin, …).
* `token` - Plaintext claim or session token.

# Returns

`Some(...)` when found / applicable; otherwise `None`.

```rust
fn ticket_portal_url(integrations: &bookclerk_config::IntegrationsConfig, token: &str) -> Option<String>
```



